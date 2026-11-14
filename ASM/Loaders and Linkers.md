![](../assets/Pasted%20image%2020261114101958.png)



To execute an object program, we need:

- **Relocation**: Modifies the object program so it can be loaded at an address different from the one originally specified
- **Linking**
  -> Combines two or more separate object programs
  -> Supplies the information needed to allow references between them
- **Loading and Allocation**
  -> Allocates memory locations
  -> Brings the object program into memory for execution


- **Loader**: brings an object program into memory and starts its execution
- **Linker**: performs only the linking operations
  - A separate loader handles relocation and loading



![](../assets/Pasted%20image%2020261114102809.png)


## Assemble-and-go Loader

- **Characteristic** 
  -> The object code is stored directly in memory after assembly 
  -> A single JUMP instruction transfers control to the start of the program

- **Advantage** 
  -> Simple, convenient development environment 

- **Disadvantage** 
  -> The program must be assembled again every time it is executed
  -> All program parts must be coded in the same language


## Absolute loader

An absolute loader loads the program at the **fixed address written in the object program**

-> Does not perform linking and program relocation.
-> The contents of memory locations for which there is no text record are shown as xxxx.
-> Each byte of assembled code is given using its Hex representation in character form.

![](../assets/Pasted%20image%2020261114103255.png)

### Algorithm

![](../assets/Pasted%20image%2020261114103316.png)


### Design

For SIC, **object program is a text file**, ie, every byte of code is written as two readable hex characters. This makes it easy to read by human beings. So the loader needs a conversion step. For example:

- STL instruction has opcode hex `14`, written in the file as the two characters `'1'` and `'4'`
- Each character takes one byte: `'1'` = hex 31, `'4'` = hex 34
- So `14` in the file occupies **two bytes** in memory if copied as-is
- For execution, the opcode must be stored in **one byte** with value hex `14` = `00010100`
- Each printed character represents only **one half-byte** (4 bits or 1 hex digit)
- So the loader must **pack each pair of characters into one byte**

However in general,

- Each byte of object code is also stored as a single byte
- Most machines store object programs in binary form
- File and device conventions must make sure program bytes are not interpreted as **control characters** (for example newline or end-of-file)


### Bootstrap loader

-> When a computer is first tuned on or restarted, a special type of absolute loader, called bootstrap loader is executed. This bootstrap loads the first program to be run by the computer - usually an operating system.

Its only job is to read a program from device F1 into memory and run it.

![](../assets/Pasted%20image%2020261114105128.png)

It does the following:

- Reads characters from device F1 (the program is stored as hex digit ASCII characters)
- Converts each pair of characters into one byte
- Stores the bytes in consecutive memory locations starting at address 80 (hex)
- When it reads the end-of-file character (hex 04), it jumps to address 80 and the loaded program starts running


```
Begin
X = 0x80         X = the address of the next memory location to be loaded
Loop:
  A <- GETC                    first hex digit  (value 0-15)
  S <- A in high 4 bits   ;save the value in the high-order 4 bits of S
  A <- GETC                    second hex digit (value 0-15)
  A <- A + S                   combine A and S into one byte
  store the value (in A) to the address in register X
  X <- X + 1                   next location
End
```

GETC reads one character and turns it into its numeric hexadecimal value.

Example, file contains the characters `'1' '4'`:

```
GETC -> 1      S = 0001 0000   (1 shifted left 4 bits)
GETC -> 4      A = 0000 0100
A + S          = 0001 0100  = hex 14   -> stored at address 80
```


In assembly:


```
BOOT    START 0         bootstrap loader for SIC/XE


		CLEAR A         A = 0
		LDX #128        X = 0x80 (128 decimal)
		LOOP:
LOOP:	JSUB GETC       read hex digit from program being loaded into A
		RMO A,S         copy A into S
		SHIFTL S,4      shift S left 4 bits (high-order half of the byte)
		JSUB GETC       read second digit into A
		ADDR S,A        A <- A + S  (one complete byte)
		STCH 0,X        store the byte at the address in X
		TIXR X,X        X <- X + 1
		J LOOP          repeat


GETC	TD INPUT            test input device
		JEQ GETC                loop until ready
		RD INPUT                read one character into A
		COMP #4                 if the character is hex 0x04
		JEQ 80                  jump to start of program just loaded
		COMP #48                compare to hex 0x30 ('0')
		JLT GETC                character below '0' -> skip it
		SUB #48                 subtract hex 0x30 to get ASCII -> numeric value
		COMP #10
		JLT RETURN              if result is <10, conversion complete
		SUB #7                  else we need to handle A-F case
RETURN	RSUB                    return to caller
	
INPUT   BYTE X'F1'        the code for the input device
		END LOOP
```

![](../assets/Pasted%20image%2020261114111633.png)
![](../assets/Pasted%20image%2020261114111644.png)


### Disadvantages

![](../assets/Pasted%20image%2020261114111706.png)




## Relocating loaders

Benifit of relocation loaders:

-> efficient sharing of the machine with larger memory when several independent programs are to be run together
-> support the use of subroutine libraries efficiently


There are 2methods for specifying relocation:

-> Modification record

- The assembler adds an extra record type, the **M record**, to the object program
- Each M record says “the address field at this location needs to be adjusted”
- The loader adds the program’s actual starting address to each field listed

-> Relocation bit

- Each instruction gets **one relocation bit**
    - 1 = modification is necessary
    - 0 = no modification is needed
- The relocation bits for a Text record are **gathered into a bit mask**, which is stored in that Text record
- The loader reads the mask and relocates only the instructions whose bit is 1


