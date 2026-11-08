
**Machine Dependent Assembler Features:**

- instruction formats
- addressing modes
- program relocation

**Machine Independent Assembler Features:**

- literals
- symbol-defining statements
- Expressions handling in the program
- program blocks
- control sections and
- program linking


### Machine Dependent Assembler Features


#### Instruction formats

Already discussed for SIC/XE.

#### Addressing modes

Already discussed for SIC/XE.


### Machine Independent Assembler Features

#### Literals

![](../assets/Pasted%20image%2020261108124943.png)


The `=` is what marks it as a literal. `=C'EOF'` means “the 3 characters E, O, F”. `=X'05'` means “the hex byte 05”.

Columns: line number, address, label, opcode, operand, object code:
```
45   001A   ENDFIL   LDA   =C'EOF'     032010
93                   LTORG
     002D   *        =C'EOF'            454F46
```

At address 001A, `LDA =C'EOF'` is assembled. The assembler doesn’t know yet where the literal will live, so it just records the literal in a table called **LITTAB** (literal table).

`LTORG` is a directive meaning “place all the pending literals here.” The assembler dumps the literal’s bytes at that point, at address 002D. The `*` in the label column just marks “this is a literal, not a labeled symbol”. The object code `454F46` is the ASCII for E, O, F (45 = E, 4F = O, 46 = F).

Now the literal has an address, 002D, so `LDA` can be finished.

```
PC after LDA = 001A + 3 = 001D
disp = 002D - 001D = 10 (hex)

03 20 10  ->  opcode 00, n=1 i=1, p=1, disp=010
```


Difference between literal and immediate addressing: In immediate operands, the operand value is assembled as part of the machine instruction. Ex- `LDA #3`. For literals, the assembler generates the specified value as a constant at some other memory location. Ex- `LDA =C’EOF’`.


##### Literals implementation

**Literal pools**

-> Normally literals are placed into a pool at the end of the program. In some cases, it is desirable to place literals into a pool at some other location in the object program. For this, assembler directive `LTORG` is used. 

**Duplicate pools**

-> The assemblers should recognize duplicate literals and store only one copy of the specified data value. 

```
215   1062   WLOOP   TD   =X'05'
230   106B           WD   =X'05'
```

Approaches to detect duplicates:

**Compare the defining expression** 

The assembler compares the literal’s text: is `=X'05'` the same as `=X'05'`? Simple string match. The slide’s note about `LOCCTR=*` is the catch: the symbol `*` means “the current value of the location counter”, which changes with every line. So:

```
LDA  =*     at address 1000  -> means 1000
LDA  =*     at address 2000  -> means 2000
```

The text is identical, but the values are different, so these are **not** duplicates. A pure text comparison would wrongly merge them. The assembler has to treat `=*` as a special case.

**Compare the generated data value**

The assembler converts each literal to its actual bytes, then compares those. This would catch cases like `=C'A'` and `=X'41'`, which look different but produce the same byte (41 is ASCII for A). This method has more complexity.


**LITTAB**

The literal table contains: literal name, the operand value and length, the address assigned to the operand. Ex- 

```
Literal name | Operand value | Length | Address
-------------+---------------+--------+--------
=C'EOF'      | 454F46        | 3      | 002D
=X'05'       | 05            | 1      | 1B50
```

**Pass 1**

1. Build LITTAB with literal name, operand value and length, leaving the address blank.
2. When `LTORG` is encountered, assign an address to each literal not yet assigned an address

```
LOCCTR = 002D at LTORG
  =C'EOF'  gets address 002D, LOCCTR becomes 0030
  =X'05'   gets address 0030, LOCCTR becomes 0031
```

**Pass 2**

1. Search LITTAB for each literal operand encountered
2. Generate the data values using BYTE or WORD
3. Generate modification records for literals that represent an address


#### Symbol defining statements

Normally, a label’s value is the LOCCTR at the line where we write it:

```
TOTAL   RESW   1        TOTAL = address of this line (set by LOCCTR)
```

With EQU, we can choose the value:

```
MAXLEN  EQU    4096     MAXLEN = 4096
```

EQU generates **no object code** and uses no memory. It only adds an entry to SYMTAB, with the value you specified.

Example 2 (register naming):
```
BASE    EQU    R1
COUNT   EQU    R2
INDEX   EQU    R3
```

Example 3 (a value computed from other symbols):

```
MAXLEN  EQU    BUFEND-BUFFER
```

The value is an **expression**: the distance between two labels. If BUFFER is at 1036 and BUFEND is at 2036, then MAXLEN = 1000 (hex 1000 - hex 36 etc., whatever the subtraction gives). So the buffer’s size is calculated automatically, and if you resize the buffer, MAXLEN updates by itself.

This one has a catch: the assembler must already know BUFEND and BUFFER when it reaches the EQU line. That is the **forward reference rule**. For EQU, every symbol in the expression must be defined _earlier_ in the program, because Pass 1 needs the value immediately to store in SYMTAB.


**ORG (origin)**

**ORG** is an assembler directive that **resets the location counter (LOCCTR) to a value you choose**.

(This note is incomplete)









