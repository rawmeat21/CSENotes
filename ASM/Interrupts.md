An interrupt is a break in the flow of execution of program. The CPU is “interrupted” by a program. When an interrupt occurs, the CPU deals with the interruption, then carries on where it left off.

## Types of interrupts

### Hardware

An external signal is applied to the NMI input pin or the INTR input pin. Used to deal with I/O devices needing attention.

NMI - non-maskable interrupt
INTR - interrupt

### Software

Execution of the interrupt instruction (`INT`) causes this, or

Some error condition is produced by the execution of an instruction (divide by 0)


## `INT` instruction

-> On Execution, an INT interrupts processing and accesses the **interrupt services table** in low memory to determine the address of the required routine.

-> The operation then transfers to DOS or to BIOS for specified action and returns to main program to resume the processing.

-> Interrupts require a trail that facilitates exiting a program and, on successful completion , returning to it.

### Steps performed

1. Decrement the stack pointer by 2 and pushes the contents of flags register onto the stack
2. Clears the interrupt and trap flags
3. Decrements the stack pointer by 2 and pushes the CS register onto the stack
4. Decrements the stack pointer by 2 and pushes the instruction pointer onto the stack
5. Causes the required operation to be performed

To return from an interrupt, the routine issues an `IRET` (interrupt return),which pops the registers of the stack and returns to the instruction immediately following the INT in the program.


### Some interrupts

`INT 21H` is the DOS services interrupt. It is one entry point that provides dozens of different services: read a character, print a character, print a string, open a file, exit the program, and so on. The CPU only sees `INT 21h`, so something else must say which service you want. By convention, you put the service number in **AH** before executing the interrupt.

#### Read character from standard input with echo

```
mov ah, 1
int 21h
```
If there is no character in the keyboard buffer, the function waits until any key is pressed.

Result is stored in AL.

#### Write character to standard output

DL = character to write

```
mov ah, 2
mov dl, 'a'
int 21h
```
 After execution, AL = DL

#### Output a string

```
org 100h
mov dx, offset msg
mov ah, 9
int 21h
ret
msg db "hello world $"
```
**DS:DX** must point to the string. DX holds the _offset_ of the first character, and DS supplies the segment.

- `org 100h`: an assembler directive, not an instruction. It tells the assembler that the code will be loaded starting at offset 100H. This is the layout of `.COM` programs, which DOS loads at offset 100H of a single segment.
- `mov dx, offset msg`: puts the _address_ of `msg` into DX.`offset msg` gives the address, while plain `msg` would read the contents. Here we want the address, because the service needs to know where the string starts.
- `mov ah, 9`: selects the print-string service.
- `int 21h`: calls DOS, which prints the string.
- `ret`: ends the program. In a `.COM` file this works because DOS arranged for the stack to return to a point that terminates the program. In `.EXE` programs you would use `mov ah, 4Ch` and `int 21h` instead.
- `msg db "hello world $"`: reserves the bytes of the string in memory and gives the first one the name `msg`.

##### .exe variant

```
.model small
.stack 100h
.data
msg db "hello world $"
.code
mov ax, @data
mov ds, ax
mov dx, offset msg
mov ah, 9
int 21h

// this ends the program
mov ah, 4Ch
int 21h
end
```

`mov ax, @data` and `mov ds, ax`: make DS point at your data segment. These must come before `mov dx, offset msg`, because the print service reads from DS:DX.

