## SIC/XE Machine Architecture

SIC/XE is extended SIC. It can run programs written SIC too, so SIC/XE has a superset of the functions in SIC.

### Memory

 2^20 bytes in the computer memory

### Registers

It contains all registers already in SIC, plus:

![](../assets/Pasted%20image%2020261020131927.png)

### Data Formats

Floating-point data type: frac * 2^(exp-1024)

fraction is a value between 0 and 1
exponent is an unsigned binary number between 0 and 2047

![](../assets/Pasted%20image%2020261020132043.png)

zero is represented as all 0
### Instruction formats



![](../assets/Pasted%20image%2020261020132356.png)

![](../assets/Pasted%20image%2020261020132658.png)

![](../assets/Pasted%20image%2020261020132729.png)

![](../assets/Pasted%20image%2020261020132753.png)

![](../assets/Pasted%20image%2020261020132815.png)

### Instruction set

-> has new registers: LDB, STB, etc.
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



