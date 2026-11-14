
**Machine Dependent Assembler Features:**

- instruction formats
- addressing modes
- program relocation

**Machine Independent Assembler Features:**

- literals
- symbol-defining statements
- Expressions handling in the program
- program blocks
- control sections
- program linking


### Machine Dependent Assembler Features


### Instruction formats

Already discussed for SIC/XE.

### Addressing modes

Already discussed for SIC/XE.


## Machine Independent Assembler Features

### Literals

![](../assets/Pasted%20image%2020261114131701.png)

On the left, you see the program without literals. At line 45, 3 bytes of data are loaded into the accumulator. The 3 bytes of data are placed at line 80.

On the right, you see a program with literals. At line 45, however, we directly assign the value to LDA and don't write it again afterwards. How to make this work?

![](../assets/Pasted%20image%2020261108124943.png)


The `=` is what marks it as a literal. `=C'EOF'` means “the 3 characters E, O, F”. `=X'05'` means “the hex byte 05”.

![](../assets/Pasted%20image%2020261114132150.png)

Difference between literal and immediate addressing:

![](../assets/Pasted%20image%2020261114132513.png)


#### Literals implementation

##### Literal pools

-> Normally literals are placed into a pool at the end of the program. In some cases, it is desirable to place literals into a pool at some other location in the object program. For this, assembler directive `LTORG` is used. It creates a literal pool that contains all of the literal operands used since previous `LTORG`.

![](../assets/Pasted%20image%2020261114132704.png)

![](../assets/Pasted%20image%2020261114132851.png)

Look at the left part. At line 45, the object code is `032010`, and the EOF label is at `002D`. This is basically PC relative addressing.

On the right, note that the object code remained same. We placed `LTORG` at line 93. This dumped the label for the literal (`=C'EOF'`) after `LTORG`.

##### Duplicate pools

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


##### LITTAB

The literal table contains: literal name, the operand value and length, the address assigned to the operand. It is organised as a hash table. Ex- 

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


### Symbol defining statements

#### EQU

Normally, a label’s value is the LOCCTR at the line where we write it:

```
TOTAL   RESW   1        TOTAL = address of this line (set by LOCCTR)
```

EQU is used to define symbols. Syntax: `symbol EQU value`. A `value` can be a constant, expression or some other symbol.

Example 1:

![](../assets/Pasted%20image%2020261114134501.png)

EQU generates **no object code** and uses no memory. It only adds an entry to SYMTAB, with the value we specify.

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


#### ORG (origin)

**ORG** is an assembler directive that **resets the location counter (LOCCTR) to a value we choose**.

Syntax: `ORG value` (Sets `LOCCTR = value)

Suppose you want to build a symbol table. There are 100 entries. Each entry is 11 bytes (6+3+2). So, you need 1100 bytes in total.

![](../assets/Pasted%20image%2020261110120930.png)

![](../assets/Pasted%20image%2020261110121041.png)

You want to write `LDA VALUE,X`, meaning “load the VALUE field of entry number X”
Using EQU, it works, but you must count the bytes by hand.

ORG explained:

Suppose STAB is at address 1000.

```
STAB    RESB   1100       STAB = 1000, LOCCTR becomes 1000 + 1100 = 144C
        ORG    STAB       LOCCTR is reset back to 1000
SYMBOL  RESB   6          SYMBOL = 1000, LOCCTR -> 1006
VALUE   RESW   1          VALUE  = 1006, LOCCTR -> 1009
FLAGS   RESB   2          FLAGS  = 1009, LOCCTR -> 100B
        ORG    STAB+1100  LOCCTR jumps forward to 144C
```


### Expressions 

![](../assets/Pasted%20image%2020261114135204.png)

![](../assets/Pasted%20image%2020261110122424.png)

The reason is that a relative term is really “program start address + some fixed offset”, and two such terms with opposite signs cancel the unknown part.

- **Absolute**: a value that is the same no matter where the program is loaded (`4096`, a size, a count).
- **Relative**: a value that depends on where the program is loaded. Every label is like this, because its address shifts if the whole program moves.

#### SYMTAB modification for expressions

A little bit of context:

There are 3 times addresses can be bound:

```
Assembly time:   addresses fixed by the assembler (absolute program)
Load time:       the loader patches addresses when placing the program
Execution time:  addresses stay logical; hardware translates on every access
```

**SIC/XE has no MMU and no logical-to-physical translation. The address in an instruction goes straight to memory. So if the program lands at 4000, the loader must rewrite every absolute address inside it.** So, even if your code seems relocatable, it is still technically converted into an absolute program and hence should face the same problems as absolute programs do.

This results in the following problem. Every label is S+(its value in the assembly program).

-> None of the relative terms may enter into a multiplication or division operation

```
3 * BUFFER = 3 * (S + 36)
           = 3S + 108 (Not of form S+something)
```

Other errors:

![](../assets/Pasted%20image%2020261110124304.png)

Let BUFFER = S + 36 and BUFEND = S + 1036.

```
BUFEND + BUFFER = (S + 1036) + (S + 36)
                = 2S + 1072
```

```
100 - BUFFER = 100 - (S + 36)
             = -S + 64
```

R-R is fine, R+R is an error, 3R is an error

To apply them, the assembler must know whether each symbol in an expression is relative or absolute. The value doesn’t tell you:

```
BUFFER  value 36    (a label: an address)
MAXLEN  value 36    (an EQU constant: a plain number)
```

Both are 36, but `BUFFER+BUFFER` is an error while `MAXLEN+MAXLEN` is fine. Only the type column separates them.

The solution is this:

![](../assets/Pasted%20image%2020261110124707.png)




### Program blocks

-> refers to segments of code that are rearranged within a single object program unit

A little bit of context: In a plain program, the object code follows the source order. Suppose you write code, then a data area, then more code:

```
        LDA    LENGTH
        ...
LENGTH  RESW   1        <- data in the middle
BUFFER  RESB   4096     <- 4096-byte buffer in the middle
        ...
        STA    LENGTH
```

Two issues appear:

1. **Reach.** Format 3 with PC-relative can only reach -2048..+2047 bytes. The 4096-byte buffer sitting between two pieces of code pushes things out of range, forcing you into format 4 (4 bytes, plus a modification record) or base-relative tricks.
2. **Readability.** You’d like to put `LENGTH` and `BUFFER` next to the code that uses them for readability, but you’d also like them physically grouped at the end of the program.

Program blocks helps address these problems.


![](../assets/Pasted%20image%2020261114144217.png)
![](../assets/Pasted%20image%2020261114144240.png)
![](../assets/Pasted%20image%2020261114144229.png)

In main subroutine, you can see a `USE` instruction: `USE CDATA`. This means that all instructions after this directive and before the next `USE` directive belong to block CDATA. So, lines 95, 100 belong to CDATA. 

In RDREC, at line 123, you see a single `USE` directive but no blockname. This refers to the default block.

At line 252, we again have the CDATA block. This causes the lines 95, 100, 253, 255 to be grouped together.

```
USE [blockname]
```

`USE` is an assembler directive (no object code). 

- `USE` with no name switches to the **default (unnamed) block**.
- Everything before the first `USE` is also in the default block.
- You can switch back and forth. Each block may appear in **several separate pieces** of the source.


![](../assets/Pasted%20image%2020261114145058.png)
![](../assets/Pasted%20image%2020261114145247.png)
![](../assets/Pasted%20image%2020261114145351.png)

Let's talk about pass 1:

The assembler goes through each line and assigns a block number. Be mindful that the LC is not continuous. It resets to `0000` when it finds a new block and continues from there. 

At line 123, note that LC=`0027`. This is because block 0 paused at `0024` at line 70 before.

Now, note the following from the figure:

- First instruction of default block (0) is at 0000. Last is at 0063.
- After this the CDATA block starts at 0066. Now, last instruction of CDATA occurs at 000A. This gives the next block's (CBLKS) starting address, which is 0066+000A+1=0071 (the last object code is 1 byte long)

![](../assets/Pasted%20image%2020261114150717.png)


#### Program blocks implementation

![](../assets/Pasted%20image%2020261110130936.png)



### Control sections and program linking

-> refers to segments of code that are translated into independent object program units.
-> Most often used for subroutines or other logical subdivisions of a program
-> Programmers can assemble, load, and manipulate each of these control sections separately
-> Instruction in one control section may need to refer to instructions or data in another section. Because of this, there should be some means for linking control sections together

![](../assets/Pasted%20image%2020261114151903.png)
![](../assets/Pasted%20image%2020261114151927.png)
![](../assets/Pasted%20image%2020261114151938.png)

CSECT is used to define a control section.

For example:

RDREC is one control section (defined at line 109). `EXTDEF` at 122 tells us about symbols defined outside RDREC which are used in RDREC.

The main routine is also a control section. It has `EXTREF` and `EXTDEF`. `EXTDEF` tells us about the symbols defined in the main control section and usable elsewhere.
#### External definitions and references

These are used to provide the linking between control sections. There are two of these directives: `EXTDEF` and `EXTREF`.

![](../assets/Pasted%20image%2020261110131805.png)


#### Implementation

Two new records are used (D and R).

D is for EXTDEF
R is for EXTREF

![](../assets/Pasted%20image%2020261110131845.png)

![](../assets/Pasted%20image%2020261110131900.png)


#### External references in expression

![](../assets/Pasted%20image%2020261110132012.png)