## SIC/XE Machine Architecture

SIC/XE is extended SIC. It can run programs written SIC too, so SIC/XE has a superset of the functions in SIC.

### Memory

 2^20 bytes in the computer memory

### Registers

It contains all registers already in SIC, plus:

![](../assets/Pasted%20image%2020261020131927.png)

```
Name  Num  Size     Purpose
A     0    24 bits  Accumulator (arithmetic and logic)
X     1    24 bits  Index register
L     2    24 bits  Linkage register (stores return address for JSUB)
B     3    24 bits  Base register (base-relative addressing)
S     4    24 bits  General purpose
T     5    24 bits  General purpose
F     6    48 bits  Floating-point accumulator
PC    8    24 bits  Program counter (address of next instruction)
SW    9    24 bits  Status word (condition codes, etc.)
```

### Data Formats

Floating-point data type: frac * 2^(exp-1024)

fraction is a value between 0 and 1
exponent is an unsigned binary number between 0 and 2047

![](../assets/Pasted%20image%2020261020132043.png)

zero is represented as all 0
### Instruction formats

![](../assets/Pasted%20image%2020261020132356.png)


```
Stage 1: compute the Target Address (TA)   <- uses x, b, p, e and disp/address
Stage 2: decide what to do with TA         <- uses n, i
```

Every SIC/XE opcode has its lowest 2 bits equal to 0 (LDA = `00`, STA = `0C`, JSUB = `48`). Those 2 spare bits are reused as `n` and `i`.

#### Stage 1: computing the Target Address (x, b, p, e)

The 12-bit `disp` is too small to hold a real address (12 bits only reaches 4 KB, but memory is 1 MB). So the machine offers ways to _build_ an address from a small number.

**b=0, p=0 (direct):** TA = disp. Works only if the target is in addresses 0 to 4095.

**b=0, p=1 (PC-relative):** TA = (PC) + disp, where disp is a _signed_ 12-bit number (-2048 to +2047). “Target is N bytes ahead of or behind me.” Important detail: by the time the instruction executes, PC has already been advanced past it, so PC = address of the _next_ instruction.

**b=1, p=0 (base-relative):** TA = (B) + disp, where disp is _unsigned_ (0 to 4095). B is a register you set up yourself with `LDB` and `BASE`.

**x=1 (indexed):** after the above, add X as well: TA = TA + (X). This is how `TABLE,X` walks through an array.

**e=1 (format 4):** skip all of this. The 20-bit field _is_ the TA directly, and b and p are 0.


#### Stage 2: what to do with TA (n, i)

This is where “simple”, “immediate” and “indirect” get their real meaning. They differ in **how many times memory is read** and **what counts as the operand**.

Suppose TA = 1030, memory at 1030 holds `002000`, and memory at 2000 holds `000005`:

```
Memory:
 1030: 00 20 00      (a pointer-looking value)
 2000: 00 00 05      (the number 5)
```

```
n i   mode        what the CPU does                    LDA result
1 1   simple      read 3 bytes at TA                   A = 002000
0 1   immediate   use TA itself as the value           A = 001030
1 0   indirect    read 3 bytes at TA -> call it P,     A = 000005
                  then read 3 bytes at P
```

- **Simple** (`LDA TOTAL`): one memory read. TA points at the data. So, `A=M[TA]`.
- **Immediate** (`LDA #3`): zero memory reads. The computed TA _is_ the data. For `LDA #3`, disp = 3 with direct mode, so TA = 3 and A gets 3. For `LDA #TOTAL`, TA is TOTAL’s address, so A gets the _address_ of TOTAL, not its contents. So, `A=TA`.
- **Indirect** (`LDA @TOTAL`): two memory reads. TA points to a _pointer_, and the pointer points to the data. So, `A=M[M[TA]]`.

Restriction: `x=1` is only allowed with simple addressing, so `LDA #TABLE,X` is invalid.


#### But how do you set the nixbpe values?

```
Bit(s)   Source
------   ------------------------------------------------------
opcode   The table (18 for ADD). Top 6 bits go in the opcode field.
n, i     The PREFIX on the operand:   (none) -> n=1 i=1  simple
                                      #      -> n=0 i=1  immediate
                                      @      -> n=1 i=0  indirect
x        The ",X" SUFFIX on the operand. Present -> x=1, else 0.
e        The "+" PREFIX on the mnemonic. +ADD -> e=1 (format 4), else 0.
b, p     The ASSEMBLER decides, based on distance (see below).
disp     The ASSEMBLER computes it from the symbol's address.
```

Example:

```
ADD TOTAL      simple, format 3
ADD #5         immediate (add the constant 5)
ADD @PTR       indirect
ADD TABLE,X    indexed
+ADD TOTAL     format 4
```


**How assembler picks `b, p`:**
```
Does the operand need a computed address at all?
|
+-- No (constant, or no operand)         -> b=0 p=0  (direct)
|
+-- Is the mnemonic +format 4?           -> b=0 p=0  (20-bit address)
|
+-- Otherwise (format 3 with a symbol):
      |
      +-- (symbol - PC) in -2048..2047?  -> b=0 p=1  (PC-relative)
      |
      +-- BASE active and
      |   (symbol - B) in 0..4095?       -> b=1 p=0  (base-relative)
      |
      +-- neither                        -> ERROR (use +format 4)
```


**Examples:**

Opcode for ADD is 18, so the 6-bit opcode is `000110`.

**Case 1: direct, constant.** `ADD #5`  
There is no symbol, so nothing to compute. disp is just 5.

```
n=0 i=1 | x=0 b=0 p=0 e=0 | disp=005
0001 1001 | 0000 | 0000 0000 0101   ->  19 00 05
```

**Case 2: direct, no operand.** `RSUB` (opcode 4C)

```
n=1 i=1 | x=0 b=0 p=0 e=0 | disp=000
0100 1111 | 0000 | 0000 0000 0000   ->  4F 00 00
```

**Case 3: direct, format 4.** `+ADD TOTAL`, TOTAL at 2050  
b and p are 0 because the 20 bits already hold the whole address.

```
n=1 i=1 | x=0 b=0 p=0 e=1 | address=02050
0001 1011 | 0001 | 0000 0010 0000 0101 0000   ->  1B 10 20 50
```

**Case 4: PC-relative, target ahead.** `ADD TOTAL` at 2000, TOTAL at 2050

```
PC = 2003.  disp = 2050 - 2003 = 04D
n=1 i=1 | x=0 b=0 p=1 e=0 | disp=04D
0001 1011 | 0010 | 0000 0100 1101   ->  1B 20 4D
```

**Case 5: PC-relative, target behind.** `ADD TOTAL` at 2000, TOTAL at 1FC0

```
PC = 2003.  disp = 1FC0 - 2003 = -43 hex
-43 in 12-bit two's complement = 1000 - 043 = FBD
n=1 i=1 | x=0 b=0 p=1 e=0 | disp=FBD
0001 1011 | 0010 | 1111 1011 1101   ->  1B 2F BD
```

**Case 6: base-relative.** Instruction at 0000, TOTAL at 3010. Program has `LDB #BASEADDR` and `BASE BASEADDR`, where BASEADDR is 3000.

```
PC-relative check: 3010 - 0003 = 300D  -> way over 2047, fails
Base check:        3010 - 3000 = 010   -> fits 0..4095, works
n=1 i=1 | x=0 b=1 p=0 e=0 | disp=010
0001 1011 | 0100 | 0000 0001 0000   ->  1B 40 10
```

At runtime the CPU computes TA = (B) + 010 = 3010.

**Case 7: neither works.** Same as case 6 but with no `BASE` active. The assembler reports an error, and the fix is writing `+ADD TOTAL`.


Note that, 

- **Immediate with a symbol still uses PC-relative.** `ADD #TOTAL` is not a constant. The assembler still needs TOTAL’s address, so it tries PC-relative (p=1), then base. Only a literal number like `#5` gets b=0, p=0.
- **BASE is only an assembler directive.** `BASE LENGTH` generates no code. It just tells the assembler “assume register B holds LENGTH’s address.” You must also load B yourself with `LDB #LENGTH`, or the CPU will compute the wrong TA at runtime.

### Addressing modes

![](../assets/Pasted%20image%2020261020132658.png)

![](../assets/Pasted%20image%2020261020132729.png)

![](../assets/Pasted%20image%2020261020132753.png)

![](../assets/Pasted%20image%2020261020132815.png)

### Instruction set

-> load or store registers: LDB, STB, etc.
->  floating-point arithmetic: ADDF, SUBF, MULF, DIVF,  ADDR, SUBR, MULR, DIVR
-> register move: RMO
->  register-register arithmetic: ADDR, SUBR, MULR, DIVR
-> supervisor call: SVC (generates an interrupt for OS)

Full:

![](../assets/Pasted%20image%2020261020133139.png)
### Input and Output

 SIO, TIO, HIO: start, test, halt the operation of I/O device

Full:

![](../assets/Pasted%20image%2020261020133202.png)



