### Pass 1: assign addresses and build the symbol table

Every instruction is 3 bytes, so the location counter (LC) only needs special handling for the directives.

Our assembly code:

| Line | LC   | Statement         | Size (bytes)      |
| ---- | ---- | ----------------- | ----------------- |
| 5    | 1000 | COPY START 1000   | sets LC = 1000    |
| 10   | 1000 | FIRST STL RETADR  | 3                 |
| 15   | 1003 | CLOOP JSUB RDREC  | 3                 |
| 20   | 1006 | LDA LENGTH        | 3                 |
| 25   | 1009 | COMP ZERO         | 3                 |
| 30   | 100C | JEQ ENDFIL        | 3                 |
| 35   | 100F | JSUB WRREC        | 3                 |
| 40   | 1012 | J CLOOP           | 3                 |
| 45   | 1015 | ENDFIL LDA EOF    | 3                 |
| 50   | 1018 | STA BUFFER        | 3                 |
| 55   | 101B | LDA THREE         | 3                 |
| 60   | 101E | STA LENGTH        | 3                 |
| 65   | 1021 | JSUB WRREC        | 3                 |
| 70   | 1024 | LDL RETADR        | 3                 |
| 75   | 1027 | RSUB              | 3                 |
| 80   | 102A | EOF BYTE C’EOF’   | 3                 |
| 85   | 102D | THREE WORD 3      | 3                 |
| 90   | 1030 | ZERO WORD 0       | 3                 |
| 95   | 1033 | RETADR RESW 1     | 3                 |
| 100  | 1036 | LENGTH RESW 1     | 3                 |
| 105  | 1039 | BUFFER RESB 4096  | 4096 (= 1000 hex) |
| 125  | 2039 | RDREC LDX ZERO    | 3                 |
| 130  | 203C | LDA ZERO          | 3                 |
| 135  | 203F | RLOOP TD INPUT    | 3                 |
| 140  | 2042 | JEQ RLOOP         | 3                 |
| 145  | 2045 | RD INPUT          | 3                 |
| 150  | 2048 | COMP ZERO         | 3                 |
| 155  | 204B | JEQ EXIT          | 3                 |
| 160  | 204E | STCH BUFFER,X     | 3                 |
| 165  | 2051 | TIX MAXLEN        | 3                 |
| 170  | 2054 | JLT RLOOP         | 3                 |
| 175  | 2057 | EXIT STX LENGTH   | 3                 |
| 180  | 205A | RSUB              | 3                 |
| 185  | 205D | INPUT BYTE X’F1'  | 1                 |
| 190  | 205E | MAXLEN WORD 4096  | 3                 |
| 210  | 2061 | WRREC LDX ZERO    | 3                 |
| 215  | 2064 | WLOOP TD OUTPUT   | 3                 |
| 220  | 2067 | JEQ WLOOP         | 3                 |
| 225  | 206A | LDCH BUFFER,X     | 3                 |
| 230  | 206D | WD OUTPUT         | 3                 |
| 235  | 2070 | TIX LENGTH        | 3                 |
| 240  | 2073 | JLT WLOOP         | 3                 |
| 245  | 2076 | RSUB              | 3                 |
| 250  | 2079 | OUTPUT BYTE X’05' | 1                 |
| 255  | 207A | END FIRST         |                   |

Points worth checking by hand:

- **BUFFER** sits at 1039, and `RESB 4096` adds 1000 hex, so the next line lands at 2039. This is why RDREC starts at 2039.
- **BYTE** **sizes come from the constant**. `C'EOF'` is 3 characters, so 3 bytes. `X'F1'` is two hex digits, so 1 byte. This is why the LC after `INPUT` moves by 1 (205D to 205E), not 3.
- **Program length** = final LC − start = 207A − 1000 = **107A**. The header will need this.

**Symbol table after Pass 1:**

|Symbol|Addr|Symbol|Addr|Symbol|Addr|
|---|---|---|---|---|---|
|FIRST|1000|BUFFER|1039|EXIT|2057|
|CLOOP|1003|RDREC|2039|INPUT|205D|
|ENDFIL|1015|RLOOP|203F|MAXLEN|205E|
|EOF|102A|WRREC|2061|OUTPUT|2079|
|THREE|102D|WLOOP|2064|RETADR|1033|
|ZERO|1030|LENGTH|1036|||

The program name `COPY` is not stored here. It only goes in the header.

### Pass 2: generate the object code

Each instruction has three fields: **opcode** (8 bits) | **x** (1 bit) | **address** (15 bits). For each line, the assembler takes the opcode from the MOT and the address from the symbol table, then joins them.

The cases below cover every kind of line.

**Case 1: ordinary instruction (`STL RETADR`, line 10)**

- MOT: `STL` = 14
- Symbol table: `RETADR` = 1033
- x = 0, so the address field is 1033
- Object code: `14` `1033` = **141033**

Line 10 is a forward reference, since `RETADR` is defined at line 95. Pass 2 can do this lookup only because Pass 1 finished the whole table first.

**Case 2: jump to another subroutine (`JSUB RDREC`, line 15)**

- `JSUB` = 48, `RDREC` = 2039
- Object code: **482039**

This is a forward reference far ahead, into a different part of the file. The assembler doesn’t distinguish these cases. It looks the symbol up the same way.

**Case 3: indexed addressing (`STCH BUFFER,X`, line 160)**

The `,X` means the effective address is `BUFFER + (X register)`, so the x bit must be 1.

- `STCH` = 54, `BUFFER` = 1039
- In binary, 1039 = `0001 0000 0011 1001`. Setting the top bit (the x bit) gives `1001 0000 0011 1001` = 9039
- Object code: `54` `9039` = **549039**

Without `,X` the address would stay 1039. Setting x adds 8000 hex to the address field.

**Case 4: no operand (`RSUB`)**

- `RSUB` = 4C, no operand, so the address field is 0000
- Object code: **4C0000**

**Case 5: data directives**

- `BYTE C'EOF'`: convert each character to ASCII: E = 45, O = 4F, F = 46, giving **454F46**
- `BYTE X'F1'`: already hex, giving **F1**
- `WORD 4096`: 4096 decimal = 1000 hex, padded to 3 bytes, giving **001000**
- `WORD 0` gives **000000**, and `WORD 3` gives **000003**

**Case 6: reserved storage (`RESW`, `RESB`)**

- These produce **no object code**. Pass 2 only skips ahead to the next LC.

#### Full result

|Loc|Statement|Opcode|Address|Object code|
|---|---|---|---|---|
|1000|STL RETADR|14|1033|141033|
|1003|JSUB RDREC|48|2039|482039|
|1006|LDA LENGTH|00|1036|001036|
|1009|COMP ZERO|28|1030|281030|
|100C|JEQ ENDFIL|30|1015|301015|
|100F|JSUB WRREC|48|2061|482061|
|1012|J CLOOP|3C|1003|3C1003|
|1015|LDA EOF|00|102A|00102A|
|1018|STA BUFFER|0C|1039|0C1039|
|101B|LDA THREE|00|102D|00102D|
|101E|STA LENGTH|0C|1036|0C1036|
|1021|JSUB WRREC|48|2061|482061|
|1024|LDL RETADR|08|1033|081033|
|1027|RSUB|4C|none|4C0000|
|102A|BYTE C’EOF’|||454F46|
|102D|WORD 3|||000003|
|1030|WORD 0|||000000|
|1033, 1036, 1039|RESW, RESW, RESB|||none|
|2039|LDX ZERO|04|1030|041030|
|203C|LDA ZERO|00|1030|001030|
|203F|TD INPUT|E0|205D|E0205D|
|2042|JEQ RLOOP|30|203F|30203F|
|2045|RD INPUT|D8|205D|D8205D|
|2048|COMP ZERO|28|1030|281030|
|204B|JEQ EXIT|30|2057|302057|
|204E|STCH BUFFER,X|54|9039|549039|
|2051|TIX MAXLEN|2C|205E|2C205E|
|2054|JLT RLOOP|38|203F|38203F|
|2057|STX LENGTH|10|1036|101036|
|205A|RSUB|4C|none|4C0000|
|205D|BYTE X’F1'|||F1|
|205E|WORD 4096|||001000|
|2061|LDX ZERO|04|1030|041030|
|2064|TD OUTPUT|E0|2079|E02079|
|2067|JEQ WLOOP|30|2064|302064|
|206A|LDCH BUFFER,X|50|9039|509039|
|206D|WD OUTPUT|DC|2079|DC2079|
|2070|TIX LENGTH|2C|1036|2C1036|
|2073|JLT WLOOP|38|2064|382064|
|2076|RSUB|4C|none|4C0000|
|2079|BYTE X’05'|||05|

### Packaging it: the object program

Generating the code isn’t the last step. The assembler also has to pack it into **records** that the loader reads.

**Header (H).** Name, start address, length: `H COPY 001000 00107A`. The 107A comes from Pass 1 (207A − 1000).

**Text records (T).** Each holds at most **30 bytes (1E hex)** of code and starts at a given address. A new T record is started in two situations:

1. The current record is full (30 bytes).
2. A `RESW` or `RESB` is reached. No code exists for that region, so the next code can’t continue in the same record. It needs a new starting address.

Applying this to the program:

|Record|Start|Length|Why it ended|
|---|---|---|---|
|T1|001000|1E|full: 10 words from 141033 to 00102D|
|T2|00101E|15|reached RESW at 1033 (21 bytes of code from 1033 − 101E)|
|T3|002039|1E|full: from RDREC (2039), 10 words|
|T4|002057|1C|full of remaining code up to 2073|
|T5|002073|07|rest: `382064 4C0000 05`|

T2 starts at 101E because T1 ended at 101D. Between T2 and T3 there is a gap (1033 to 2038, the reserved storage), and nothing about it appears in the file. T3 simply begins at 2039. This is how the file stays small even though the program occupies almost 8 KB of address space.

**End record (E).** `E 001000`, the operand of `END FIRST`, which tells the loader where to start executing.

**Final object program:**

```
H COPY 001000 00107A
T 001000 1E 141033 482039 001036 281030 301015 482061 3C1003 00102A 0C1039 00102D
T 00101E 15 0C1036 482061 081033 4C0000 454F46 000003 000000
T 002039 1E 041030 001030 E0205D 30203F D8205D 281030 302057 549039 2C205E 38203F
T 002057 1C 101036 4C0000 F1 001000 041030 E02079 302064 509039 DC2079 2C1036
T 002073 07 382064 4C0000 05
E 001000
```