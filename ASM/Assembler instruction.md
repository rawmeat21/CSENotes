![](../assets/Pasted%20image%2020261020121613.png)

## The x bit: indexed addressing

The x bit tells the CPU whether to add the **X register** to the address field when it computes the address of the operand.

- **x = 0:** the operand is at the address in the instruction.
- **x = 1:** the operand is at `address field + (X)`, where (X) is the current value of the X register. 

This is called **indexed addressing**. The address is computed when the program runs, not when it is assembled.


The program is written for **SIC** (Simplified Instructional Computer).