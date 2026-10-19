## Immediate addressing mode

In Immediate Addressing mode the datafollows immediately after the opcode

ex: `mov dx, 1234h`

## Register addressing mode

![](../assets/Pasted%20image%2020261019134128.png)


## Memory addressing mode

### Direct addressing

![](../assets/Pasted%20image%2020261019134354.png)

Remember that lower byte means lower address and higher byte means higher address in big endian.

![](../assets/Pasted%20image%2020261019134626.png)

![](../assets/Pasted%20image%2020261019134716.png)

**DB** is “Define Byte”. It’s an assembler directive, not a CPU instruction. It never becomes machine code itself. It tells the assembler to reserve 1 byte in the data segment and fill it with the given value. `LOC DB 78H` means: allocate one byte, put 78H in it.

**LOC** is a label (a symbol). It is just a name for the memory location where that byte was placed. Internally, the assembler replaces it with a number: the offset of that byte from the start of the data segment. If `LOC` is the first thing in `.DATA`, its offset is 0000H. LOC doesn’t exist at runtime. Only the number does.

**What `MOV BH, LOC` does, step by step**

1. The assembler substitutes the offset for `LOC`, so the instruction is effectively `MOV BH, [0000H]` (direct addressing).
2. At runtime, the CPU computes the physical address from DS (the data segment register) and that offset.
3. It reads the byte stored there, which is 78H.
4. It copies that byte into BH, so BH changes from C5H to 78H.

So the instruction copies the _contents_ of the memory location, not its address. To get the address, you’d write `MOV BX, OFFSET LOC`, which loads the offset number itself.

**Two things to be aware of**

- The assembler remembers that `LOC` was declared with DB, so it knows it’s 1 byte. That’s why it can be moved into an 8-bit register like BH. Moving it into a 16-bit register like BX would be a size mismatch error.
- For this to work in a real program, DS must be set up first (typically `MOV AX, @DATA` then `MOV DS, AX`)


### Indirect addressing

![](../assets/Pasted%20image%2020261019135612.png)

#### Register indirect addressing

![](../assets/Pasted%20image%2020261019135711.png)

It is kinda like: `CL = *SI`.

![](../assets/Pasted%20image%2020261019135759.png)

The size of the register that we are pushing into determines how many memory locations we go through. As `DX` is 2 bytes, we go through 2 locations (remember memory is byte addressable here).

![](../assets/Pasted%20image%2020261019140143.png)


**Only SI, DI , and BX can be used inside `[]` from memory addressing point of view. From user point of view `[BP]` is also possible.**

In 8086, only four registers can go inside brackets: BX, SI, DI, and BP. AX, CX, and DX cannot. `MOV AL, [AX]` is an assembler error, because the hardware has no way to encode it.

**What “memory addressing point of view” means**

Each memory access uses a default segment register:

- BX, SI, and DI default to **DS** (the data segment).
- BP defaults to **SS** (the stack segment).

So when the slide says “from memory addressing point of view”, it means ordinary _data_ access, the variables you declared in `.DATA`. For that, BX, SI, and DI are the registers, because they point into DS. BP is not used for that purpose.

**What “user point of view” means**

This is loose wording on the slide, but I read it as “what you are allowed to write.” As a programmer, `[BP]` is valid syntax, so you can write `MOV AL, [BP]`. It just reads from the stack segment instead of the data segment. BP exists mainly for accessing stack contents, such as parameters and local variables inside a procedure.

You can force a different segment with an override, for example `MOV AL, DS:[BP]`, but you rarely need that.

**Note:** SI, DI, BX are offsets into the data segment (DS) by default. The physical address for `[BX]` is `DS * 16 + BX`. If DS = 2000H and BX = 0005H, the CPU reads physical address 20005H.

IMP: **They are not restricted to holding offsets.** BX, SI, and DI are general-purpose in the sense that you can also use them for arithmetic or as plain values. They become an address only at the moment you put them inside brackets.

#### Based address with displacement

**Base register can only be BX or BP**

![](../assets/Pasted%20image%2020261019141011.png)

![](../assets/Pasted%20image%2020261019141137.png)


#### Indexed addressing with displacement

**Index register can only be SI or DI**

![](../assets/Pasted%20image%2020261019141701.png)

![](../assets/Pasted%20image%2020261019141708.png)


#### Based indexed addressing

**One Register must be a Base register and the other must be an Index register**

![](../assets/Pasted%20image%2020261019141756.png)

![](../assets/Pasted%20image%2020261019141818.png)


`MOV CX, [BX][BP]` is an invalid instruction.


#### Based indexed addressing with displacement

![](../assets/Pasted%20image%2020261019141937.png)

![](../assets/Pasted%20image%2020261019141946.png)



## IO port addressing

![](../assets/Pasted%20image%2020261019142110.png)


### Fixed port addressing

**IN and OUT instructions are allowed to use only AL or AX registers**

**Port address in the range 00 to FFH is provided in the instruction directly**

![](../assets/Pasted%20image%2020261019142137.png)

![](../assets/Pasted%20image%2020261019142147.png)

![](../assets/Pasted%20image%2020261019142157.png)

![](../assets/Pasted%20image%2020261019142203.png)


### Variable port addressing

**I/O port address should be provided in DX register only**

**Port addresses range from 0000H to FFFFH**

**Data transfer with AL or AX only**

![](../assets/Pasted%20image%2020261019142320.png)

![](../assets/Pasted%20image%2020261019142327.png)

![](../assets/Pasted%20image%2020261019142338.png)

![](../assets/Pasted%20image%2020261019142350.png)



