### The 8086 Mindset & Program Structure

The 8086 processor views memory in 64-kilobyte chunks called **Segments**. You cannot access all memory at once; you must point specific hardware registers (CS, DS, SS, ES) to the segment you want to use.

A standard `.model small` MASM program perfectly mirrors this architecture:

Code snippet

```
.model small       ; Tells assembler: 1 Code segment (64KB), 1 Data segment (64KB)
.stack 100h        ; Allocates 256 bytes for the Stack Segment (SS)

.data              ; Initializes Data Segment (DS) - Global variables go here
    ; Variable declarations...

.code              ; Initializes Code Segment (CS) - Instructions go here
main proc
    mov ax, @data  ; Hardware Rule: Cannot move memory address directly to DS
    mov ds, ax     ; DS now points to your .data segment

    ; ... Your code logic ...

    mov ah, 4Ch    ; DOS function: Exit program
    int 21h        ; Call DOS interrupt
main endp
end main
```

### Data Types, Variables & Structs

MASM does not have strict types like `int` or `char`. It only cares about byte size.

|C++ Feature|16-bit MASM Equivalent|MASM Example|
|---|---|---|
|`char x = 'A';`|`db` (Define Byte, 8-bit)|`x db 'A'`|
|`int y = 1000;`|`dw` (Define Word, 16-bit)|`y dw 1000`|
|`int arr[3] = {1,2};`|`dup` (Duplicate/Array)|`arr dw 1, 2, 0`|
|Uninitialized `int z;`|`?` (Uninitialized)|`z dw ?`|
|`float f = 3.14;`|`dd` (Define Double, 32-bit)|`f dd 3.14` _(Requires 8087 FPU coprocessor instructions like `fld`, `fstp` to manipulate)_|

**Structs in MASM:**

Code snippet

```
; Declaration (outside .data)
Point STRUC
    x dw ?
    y dw ?
Point ENDS

.data
    p1 Point <10, 20>  ; Instantiation
.code
    mov ax, p1.x       ; Accessing member
```

### I/O & System Calls (cin / cout / time)

Instead of streams, you load a specific "function number" into the `AH` register and trigger `int 21h` (the DOS API).

|C++ Action|DOS Function (`AH`)|Required Setup|
|---|---|---|
|`cout << "Hi";`|`09h`|`lea dx, string` (string must end with `$`)|
|`cout << 'A';`|`02h`|`mov dl, 'A'`|
|`cin >> char;`|`01h`|Result returns in `AL`|
|`cin >> string;`|`0Ah`|`lea dx, buffer` (buffer must have specific struct layout)|
|Get Time|`2Ch`|Returns: `CH`=Hr, `CL`=Min, `DH`=Sec, `DL`=1/100 Sec|
|Get Date|`2Ah`|Returns: `CX`=Year, `DH`=Month, `DL`=Day|

### Arithmetic & Logical Operators

|C++ Operator|MASM Instruction|Notes|
|---|---|---|
|`a + b`, `a - b`|`ADD`, `SUB`|`add ax, bx`|
|`a * b`|`MUL` (Unsigned), `IMUL` (Signed)|Hardcoded: Multiplies `AL`/`AX` by operand. Result in `AX`/`DX:AX`.|
|`a / b`|`DIV` (Unsigned), `IDIV` (Signed)|Hardcoded: Divides `AX` by operand. Quotient in `AL`, Remainder in `AH`.|
|`a & b`, `a \| b`|`AND`, `OR`|Bitwise operations.|
|`a ^ b`, `~a`|`XOR`, `NOT`|`xor ax, ax` is the fastest way to set `AX` to 0.|
|`a << 1`, `a >> 1`|`SHL`, `SHR`|Shifts bits. Strict 8086 requires shifting by 1, or using `CL` for >1.|

### Control Flow (If/Else, Loops, Break, Continue)

In assembly, everything is a `CMP` (Compare) followed by a Jump (`JMP`, `JE`, `JNE`, `JG`, `JL`).

**If/Else & Conditions:**

C++

```
if (x == 5) { a = 1; } else { a = 2; }
```

Code snippet

```
    cmp x, 5
    jne else_block
    mov a, 1
    jmp end_if
else_block:
    mov a, 2
end_if:
```

**For/While Loops, Break & Continue:** MASM has a built-in `LOOP` instruction that automatically decrements the `CX` register until it hits 0.

C++

```
for(int i = 10; i > 0; i--) {
    if(skip) continue;
    if(stop) break;
}
```

Code snippet

```
    mov cx, 10
loop_start:
    ; ... do work ...
    cmp skip, 1
    je next_iter    ; C++ 'continue'

    cmp stop, 1
    je loop_end     ; C++ 'break'

next_iter:
    loop loop_start ; Decrements CX. If CX != 0, jumps to loop_start
loop_end:
```

### Pointers, Arrays, and Strings

Pointers in 8086 are memory offsets. You load the address using `LEA` (Load Effective Address) into specific index registers: `SI` (Source Index), `DI` (Destination Index), or `BX` (Base Register).

C++

```
char arr[] = {10, 20};
char* ptr = arr;
char val = *(ptr + 1);
```

Code snippet

```
.data
    arr db 10, 20
.code
    lea si, arr      ; SI is now a pointer to arr
    mov al, [si+1]   ; Dereference pointer + offset 1. AL = 20
```

_Note: Brackets `[ ]` in MASM mean "dereference this memory address."_

### Functions, Recursion & The Stack

Calling functions requires managing the stack. The stack grows downwards in memory. When you `CALL` a procedure, the CPU pushes the return address to the stack. `RET` pops it off.

**Recursion & Stack Frames:** To pass arguments without polluting global variables, C++ pushes them to the stack. You access them using the Base Pointer (`BP`).

Code snippet

```
Factorial proc
    push bp             ; Save old base pointer
    mov bp, sp          ; Set up new stack frame
    
    mov ax, [bp+4]      ; Get argument pushed before CALL
    cmp ax, 1
    jle end_fact
    
    dec ax
    push ax             ; Push (n-1)
    call Factorial      ; Recursive call
    ; ... multiply result ...
    
end_fact:
    pop bp              ; Destroy stack frame
    ret 2               ; Return and clean up 2 bytes of arguments
Factorial endp
```

### File I/O

File handling requires specific DOS interrupts, returning a File Handle (an ID number) in `AX`.

|Action|`AH` Code|Required Setup|
|---|---|---|
|**Open**|`3Dh`|`AL`=Access Mode (0=Read), `DX`=Filename pointer.|
|**Create**|`3Ch`|`CX`=Attributes (0=Normal), `DX`=Filename pointer.|
|**Read**|`3Fh`|`BX`=File Handle, `CX`=Bytes to read, `DX`=Buffer pointer.|
|**Write**|`40h`|`BX`=File Handle, `CX`=Bytes to write, `DX`=Data pointer.|
|**Close**|`3Eh`|`BX`=File Handle.|