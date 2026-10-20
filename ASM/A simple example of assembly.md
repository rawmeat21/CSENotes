## Adding two numbers: source to object code

### The program

It stores 5 and 7 in memory, loads the first into the accumulator, adds the second, and stores the sum.

```
ADDNUM  START  1000
FIRST   LDA    NUM1
        ADD    NUM2
        STA    RESULT
        RSUB
NUM1    WORD   5
NUM2    WORD   7
RESULT  RESW   1
        END    FIRST
```

Every operand here (`NUM1`, `NUM2`, `RESULT`) is defined **after** the instruction that uses it. These are all forward references, so the assembler cannot do this in one pass.

### Pass 1: build the symbol table

Pass 1 does not generate code. It tracks the location counter (LC) and records each label’s address. Every instruction and `WORD` takes 3 bytes, and `RESW 1` takes 3 bytes.

|Line|LC (hex)|Effect|LC after|
|---|---|---|---|
|`START 1000`||sets LC|1000|
|`FIRST LDA NUM1`|1000|record FIRST = 1000|1003|
|`ADD NUM2`|1003||1006|
|`STA RESULT`|1006||1009|
|`RSUB`|1009||100C|
|`NUM1 WORD 5`|100C|record NUM1 = 100C|100F|
|`NUM2 WORD 7`|100F|record NUM2 = 100F|1012|
|`RESULT RESW 1`|1012|record RESULT = 1012|1015|
|`END FIRST`||||

The final LC is 1015. Program length = 1015 − 1000 = **15** (hex), which is 21 bytes.

**Symbol table at the end of Pass 1:**

|Symbol|Address|
|---|---|
|FIRST|1000|
|NUM1|100C|
|NUM2|100F|
|RESULT|1012|

### Pass 2: generate the object code



Now every symbol has an address. For each line, look up the opcode in the MOT and the operand address in the symbol table, then combine them (opcode 8 bits, x bit 1, address 15 bits).

| Line            | Opcode (MOT) | Operand address | Object code       |
| --------------- | ------------ | --------------- | ----------------- |
| `LDA NUM1`      | 00           | 100C            | **00100C**        |
| `ADD NUM2`      | 18           | 100F            | **18100F**        |
| `STA RESULT`    | 0C           | 1012            | **0C1012**        |
| `RSUB`          | 4C           | none, so 0000   | **4C0000**        |
| `NUM1 WORD 5`   |              |                 | **000005**        |
| `NUM2 WORD 7`   |              |                 | **000007**        |
| `RESULT RESW 1` |              |                 | none (space only) |

`RSUB` takes no operand, so its address field is zeros. `RESW` puts nothing in the object code, as in the directive table.

### The final object code

Listing with addresses:

|Address|Object code|
|---|---|
|1000|00100C|
|1003|18100F|
|1006|0C1012|
|1009|4C0000|
|100C|000005|
|100F|000007|
|1012|(reserved, no code)|

In the object program format commonly used with this example, it is written as three kinds of records:

```
H^ADDNUM^001000^000015
T^001000^12^00100C^18100F^0C1012^4C0000^000005^000007
E^001000
```

- **H (header):** program name, start address, length in bytes (hex).
- **T (text):** starting address of this block, the length of the block in bytes (`12` hex = 18 bytes, the six words), then the code itself.
- **E (end):** the address of the first instruction to execute. This comes from the operand of `END FIRST`.

The `^` separators are only for readability.