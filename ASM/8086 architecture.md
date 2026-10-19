The internal structure of the 8086 consists of two separate internal processing units:

**The Bus Interface Unit (BIU)**: responsible for performing all bus operations, such as instruction fetching, reading and writing operands for memory, and inputting or outputting of data for peripherals

**EU**: responsible for executing instructions.

Both units operate asynchronously to give the 8086 an overlapping instruction fetch and instruction execution mechanism.

## Bus Interface Unit

-> The bus interface unit is the 8086’s interface to the outside world. It provides a full 16-bit bidirectional data bus and 20-bit address. 

-> The bus interface unit is responsible for performing all external bus operations. It has the following functions:

1. instruction fetch
2. instruction queuing
3. operand fetch and storage
4. address relocation
5. bus control

-> The BIU is also responsible for generating bus control signals such as those for memory read or write and I/O read or write. These signals are needed for control of the circuits in the memory and I/O subsystems.

## Execution unit

-> The execution unit is responsible for decoding and executing all instructions. 

-> It consists of an ALU, status and control flags, 8 general-purpose registers, temporary registers, and queue control logic.

-> The EU extracts instructions from the top of the queue in the BIU, decodes them, generates operand addresses if necessary, passes them to the BIU and requests it to perform the read or write bus cycles to memory or I/O, and performs the operation specified by the instruction on the operands.

-> During execution of the instruction, EU tests the status and control flags and updates them based on the results of executing the instruction. 

-> If the queue is empty, the EU waits for the next instruction byte to be fetched and shifted to the top of the queue.

-> When the EU executes a branch or jump instruction, it transfers control to a location corresponding to another set of sequential instructions. Whenever this happens, the BIU automatically resets the queue and then begins to fetch instructions from this new location to refill the queue.

![](../assets/Pasted%20image%2020261019132910.png)






