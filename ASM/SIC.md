SIC is a hypothetical computer that includes the hardware features most often found on real machines.

There are 2 versions: Standard model (SIC) and Extended model (SIC/XE)

## SIC Machine Architecture

### Memory

2^15 (32,768) bytes in the computer memory
Word is 3 bytes.
1 byte is 8 bits

### Registers

![](../assets/Pasted%20image%2020261020122700.png)

### Data Formats

Integers are stored as 24-bit binary numbers; 2’s complement representation is used for negative values. No floating-point hardware is present.

### Instruction format

![](../assets/Pasted%20image%2020261020131205.png)

### Addressing Modes

![](../assets/Pasted%20image%2020261020131242.png)


### Instruction set

-> load and store: LDA, LDX, STA, STX, etc.
-> integer arithmetic operations: ADD, SUB, MUL, DIV, etc. All arithmetic operations involve register A and a word in memory, with the result being left in the register
-> comparison: COMP. COMP compares the value in register A with a word in memory. This instruction sets a condition code CC to indicate the result.
-> conditional jump instructions: JLT, JEQ, JGT. These instructions test the setting of CC and jump accordingly.
-> subroutine linkage: JSUB, RSUB. JSUB jumps to the subroutine, placing the return address in register L. RSUB returns by jumping to the address contained in register L. 

### Input and Output

-> Input and output are performed by transferring 1 byte at a time to or from the rightmost 8 bits of register A
-> The Test Device (TD) instruction tests whether the addressed device is ready to send or receive a byte of data

Test Device (TD): < (ready), = (not ready)

CC of < means device is ready
CC of = means device is not ready

-> Read Data (RD) for reading data and Write Data (WD) for writing data.

