## 16-bit x86 Assembly (MASM, DOSBox) — Full Tutorial

---

### 1. The memory model you're working in

Real mode = 20-bit address bus, but registers are 16-bit. Addresses are formed as:

```
physical_address = (segment << 4) + offset
```

So `segment:offset` like `1000h:0002h` → `10002h`. This is why you have segment registers:

```
CS  - Code Segment   (paired with IP for instruction fetch)
DS  - Data Segment   (default segment for most data access)
SS  - Stack Segment  (paired with SP/BP)
ES  - Extra Segment  (used for string ops, extra data pointer)
```

**Two program models matter in DOSBox/MASM:**

- **`.COM`**: single segment, CS=DS=SS=ES, origin at `100h` (first 256 bytes reserved for PSP — Program Segment Prefix). Max 64KB total. Assembled with `.MODEL TINY`.
- **`.EXE`**: separate segments for code/data/stack, more flexible, what you'll use for anything nontrivial. Assembled with `.MODEL SMALL` (1 code seg + 1 data seg, both ≤64KB) typically.

#### General `.EXE` structure (this is the template you build everything from)

```asm
.MODEL SMALL          ; memory model: 1 code segment, 1 data segment
.STACK 100h            ; reserve 256 bytes for stack

.DATA                  ; ---- initialized data segment ----
    ; variables, strings, constants go here

.CODE                  ; ---- code segment ----
MAIN PROC
    MOV AX, @DATA       ; @DATA = segment address of data segment
    MOV DS, AX          ; DS must be set manually — CS is set by DOS loader,
                         ; but DS is NOT auto-pointed at .DATA

    ; ... your program ...

    MOV AH, 4Ch          ; DOS function: terminate program
    MOV AL, 0            ; return code
    INT 21h
MAIN ENDP
END MAIN                ; entry point
```

**Things to always keep in mind:**

1. **DS is not automatically set.** You must `MOV AX, @DATA` then `MOV DS, AX` at the start — you can't `MOV DS, @DATA` directly because you can't move an immediate into a segment register.
2. **Segment registers can only be loaded from a general register**, not an immediate: `MOV DS, 1234h` is illegal; `MOV AX, 1234h` / `MOV DS, AX` works.
3. **All DOS/BIOS interaction happens through interrupts** — `int 21h` (DOS services, function number in AH), `int 10h` (video/BIOS), `int 1Ah` (time), etc.
4. Registers are 16-bit (`AX, BX, CX, DX, SI, DI, BP, SP`), each of `AX/BX/CX/DX` splits into 8-bit halves (`AH/AL`, etc.) — no way to address SI/DI/BP as 8-bit.
5. Flags register (`FLAGS`) holds `ZF, CF, SF, OF, PF, AF` — comparisons and arithmetic set these implicitly; conditional jumps read them.

---

### 2. Data types & variable declaration

x86 asm has no real "types" — only **storage sizes**. This maps to C types by size, not by semantic meaning (no signed/unsigned distinction at declaration time — that's an interpretation you enforce with the instructions you use).

|Directive|Size|Roughly like C|
|---|---|---|
|`DB`|1 byte|`char`, `unsigned char`, `bool`|
|`DW`|2 bytes (word)|`short`, `int` (16-bit int in this era)|
|`DD`|4 bytes (doubleword)|`long`, `float`|
|`DQ`|8 bytes (quadword)|`double`, `long long`|
|`DT`|10 bytes|`long double` (x87 extended)|

asm

```asm
.DATA
    byteVar   DB   10          ; equivalent to: unsigned char byteVar = 10;
    wordVar   DW   1000        ; equivalent to: int wordVar = 1000;
    negVar    DW   -5          ; stored as two's complement, 0FFFBh
    charVar   DB   'A'         ; single char, stored as ASCII 41h
    str1      DB   'Hello$'    ; DOS string, '$'-terminated (for int 21h/09h)
    str2      DB   'Hi',0      ; C-style null-terminated string
    arr       DW   1,2,3,4,5   ; array of 5 words -> int arr[5] = {1,2,3,4,5};
    buffer    DB   50 DUP(?)   ; 50 uninitialized bytes -> char buffer[50];
    zeroed    DW   20 DUP(0)   ; 20 words all zero
```

- `?` means "reserve space, don't initialize" (like an uninitialized C variable).
- `DUP` is your array-repeat mechanism — no other way to declare arrays.
- There is no built-in floating point storage/arithmetic in base 8086 — floats need the **x87 FPU** (section 14) or manual fixed-point math.

---

### 3. Printing (stdout equivalent)

DOS gives you `int 21h` functions. Two you'll use constantly:

**Function 02h — print single character:**

asm

```asm
MOV AH, 02h
MOV DL, 'A'      ; character to print
INT 21h
```

**Function 09h — print `$`-terminated string:**

asm

```asm
.DATA
    msg DB 'Hello, World!$'

.CODE
    MOV AH, 09h
    LEA DX, msg      ; DX = offset of msg within DS
    INT 21h
```

`LEA` (Load Effective Address) puts the **offset** of `msg` into DX — this is your `&msg` in C terms, except it's just the offset; DS is assumed to hold the segment part.

Printing an integer (no built-in `printf("%d")` equivalent — you must convert to ASCII yourself):

asm

```asm
; print the number in AX (unsigned, 0-65535)
PRINT_NUM PROC
    PUSH AX
    PUSH BX
    PUSH CX
    PUSH DX

    MOV BX, 10
    XOR CX, CX          ; digit counter = 0

DIVIDE_LOOP:
    XOR DX, DX
    DIV BX               ; AX = AX/10, DX = AX%10 (DIV uses DX:AX as dividend)
    PUSH DX               ; save digit (reversed order)
    INC CX
    TEST AX, AX
    JNZ DIVIDE_LOOP

PRINT_LOOP:
    POP DX
    ADD DL, '0'           ; convert digit to ASCII
    MOV AH, 02h
    INT 21h
    LOOP PRINT_LOOP        ; LOOP decrements CX, jumps if CX != 0

    POP DX
    POP CX
    POP BX
    POP AX
    RET
PRINT_NUM ENDP
```

Key thing: `DIV BX` with a 16-bit operand divides `DX:AX` (32-bit dividend) by `BX`, quotient→AX, remainder→DX. You must zero DX first or you get garbage/overflow.

---

### 4. Taking input

**Function 01h — read single character (echoed):**

asm

```asm
MOV AH, 01h
INT 21h          ; character returned in AL
```

**Function 0Ah — buffered line input:**

asm

```asm
.DATA
    inputBuf DB 51           ; byte 0: max chars allowed (buffer size - 1)
             DB ?             ; byte 1: DOS fills in actual chars read
             DB 51 DUP(?)     ; byte 2+: actual characters go here

.CODE
    LEA DX, inputBuf
    MOV AH, 0Ah
    INT 21h
    ; actual string starts at inputBuf+2, length in inputBuf+1
```

This 3-field structure (max length, actual length, buffer) is mandatory — DOS expects exactly this layout.

Reading a number typed as text requires manual ASCII→integer conversion (there's no `scanf("%d")`):

asm

```asm
; converts digit string at SI (null or non-digit terminated) into AX
READ_NUM PROC
    XOR AX, AX          ; result = 0
    XOR CX, CX
CONVERT_LOOP:
    MOV CL, [SI]
    CMP CL, '0'
    JL DONE
    CMP CL, '9'
    JG DONE
    SUB CL, '0'          ; ASCII -> digit value
    IMUL AX, AX, 10       ; result *= 10
    ADD AX, CX            ; result += digit
    INC SI
    JMP CONVERT_LOOP
DONE:
    RET
READ_NUM ENDP
```

---

### 5. Variables, registers, and the stack

There's no automatic/local variable concept baked into the ISA the way C has stack frames — **you** build the stack frame using `BP` if you want local variables in a procedure. Global "variables" are just labels in `.DATA`.

asm

```asm
.DATA
    globalVar DW 0      ; like a global int in C — lives for program lifetime

.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    MOV globalVar, 42     ; direct memory write, no register needed to read later
```

Local variables (stack-based, like C automatic variables) — see section 8 (functions) for the full frame setup. Short version:

asm

```asm
    PUSH BP
    MOV BP, SP
    SUB SP, 4         ; reserve 4 bytes = two local words
    ; local1 = [BP-2], local2 = [BP-4]
```

---

### 6. Arithmetic and logical operators

|C op|ASM equivalent|Notes|
|---|---|---|
|`a + b`|`ADD AX, BX`|sets CF on unsigned overflow, OF on signed overflow|
|`a - b`|`SUB AX, BX`||
|`a * b`|`MUL BX` (unsigned) / `IMUL BX` (signed)|result in `DX:AX` (16×16→32-bit)|
|`a / b`|`DIV BX` (unsigned) / `IDIV BX` (signed)|`DX:AX / BX` → quotient AX, remainder DX|
|`a % b`|same as above, take DX||
|`a & b`|`AND AX, BX`||
|`a \| b`|`OR AX, BX`||
|`a ^ b`|`XOR AX, BX`|also the idiom for zeroing a register: `XOR AX, AX`|
|`~a`|`NOT AX`||
|`a << n`|`SHL AX, n` (or `SHL AX, CL` for variable shift)||
|`a >> n`|`SHR AX, n` (unsigned) / `SAR AX, n` (signed, sign-extends)||
|`-a`|`NEG AX`|two's complement negate|
|`a++`|`INC AX`|doesn't affect CF, unlike ADD 1|
|`a--`|`DEC AX`||

Logical (boolean, short-circuit `&&`/`||`) has **no dedicated instruction** — you build it from conditional jumps:

asm

```asm
; if (a != 0 && b != 0) ...
    CMP a, 0
    JE SHORT_CIRCUIT_FALSE
    CMP b, 0
    JE SHORT_CIRCUIT_FALSE
    ; both true — body here
    JMP END_IF
SHORT_CIRCUIT_FALSE:
    ; a==0 or b==0
END_IF:
```

---

### 7. Conditions (if/else), loops, break, continue

Everything reduces to `CMP` + conditional jump. `CMP a, b` internally does `a - b` and sets flags, without storing the result.

**Common conditional jumps** (unsigned vs signed matters!):

|Condition|Unsigned|Signed|
|---|---|---|
|equal|`JE` / `JZ`|same|
|not equal|`JNE` / `JNZ`|same|
|greater|`JA`|`JG`|
|greater or equal|`JAE`|`JGE`|
|less|`JB`|`JL`|
|less or equal|`JBE`|`JLE`|

#### if/else

asm

```asm
; if (x > 10) { ... } else { ... }
    MOV AX, x
    CMP AX, 10
    JG  THEN_BRANCH
    ; else branch
    JMP END_IF
THEN_BRANCH:
    ; if branch
END_IF:
```

#### while loop

asm

```asm
; while (i < 10) { ... i++; }
    MOV CX, 0            ; i = 0
WHILE_TOP:
    CMP CX, 10
    JGE WHILE_END
    ; body
    INC CX
    JMP WHILE_TOP
WHILE_END:
```

#### for loop using `LOOP` (CX-driven, this is the classic 8086 idiom)

asm

```asm
; for (i = 0; i < 10; i++) using CX as counter, LOOP counts DOWN
    MOV CX, 10
FOR_TOP:
    ; body (note: if you need the index value, count down and mentally
    ; invert, or keep a separate counter — LOOP only decrements CX)
    LOOP FOR_TOP          ; DEC CX; if CX != 0 jump FOR_TOP
```

`LOOP` is convenient but rigid (always uses CX, always decrements). For a real ascending-index for-loop, most people just write it manually like the `while` above.

#### break / continue

No dedicated instructions — just jump to labels placed at the right spots:

asm

```asm
    MOV CX, 0
LOOP_TOP:
    CMP CX, 10
    JGE LOOP_END              ; loop condition (exit like C's loop test)

    CMP CX, 5
    JE  LOOP_BREAK              ; break

    CMP CX, 3
    JE  LOOP_CONTINUE           ; continue

    ; loop body

LOOP_CONTINUE:
    INC CX
    JMP LOOP_TOP
LOOP_BREAK:
LOOP_END:
```

`continue` jumps to the increment step; `break` jumps past the whole loop.

#### switch/case

Also has no dedicated instruction — implemented as chained `CMP`/`JE`, or as a **jump table** for dense integer cases (this is what a compiler does under the hood):

asm

```asm
; switch(x) { case 0: ...; case 1: ...; case 2: ...; }
    MOV BX, x
    CMP BX, 2
    JA  DEFAULT_CASE
    SHL BX, 1                 ; word-size entries, so index*2
    JMP CS:JumpTable[BX]

.DATA
JumpTable DW OFFSET CASE0, OFFSET CASE1, OFFSET CASE2

.CODE
CASE0: ... JMP SWITCH_END
CASE1: ... JMP SWITCH_END
CASE2: ... JMP SWITCH_END
DEFAULT_CASE: ...
SWITCH_END:
```

---

### 8. Functions (PROC/ENDP, calling convention, stack frames)

A `PROC` is a label with metadata; `CALL`/`RET` handle control transfer, pushing/popping the return address automatically.

asm

```asm
; equivalent of: int add(int a, int b) { return a + b; }
ADD_NUMS PROC
    PUSH BP
    MOV  BP, SP           ; BP is your frame-relative base for args/locals
    ; caller pushed args right-to-left before CALL, so:
    ; [BP+0] = old BP, [BP+2] = return address (near call, 2 bytes)
    ; [BP+4] = first pushed arg = b (right-to-left convention),
    ; [BP+6] = a
    MOV AX, [BP+6]         ; a
    ADD AX, [BP+4]         ; a + b, result convention: return value in AX
    POP BP
    RET 4                   ; RET n: pop return addr, then discard n bytes
                             ; of args (caller-doesn't-clean, callee cleans —
                             ; this is like __stdcall / Pascal calling convention)
ADD_NUMS ENDP

; calling it:
    PUSH b_value            ; pushed right-to-left
    PUSH a_value
    CALL ADD_NUMS
    ; result in AX
```

**Local variables inside a function** (automatic vars in C terms):

asm

```asm
MY_FUNC PROC
    PUSH BP
    MOV  BP, SP
    SUB  SP, 4              ; two local words: [BP-2], [BP-4]

    MOV  WORD PTR [BP-2], 0    ; local1 = 0
    MOV  WORD PTR [BP-4], 10   ; local2 = 10

    MOV  SP, BP              ; deallocate locals
    POP  BP
    RET
MY_FUNC ENDP
```

**Preserving registers** — since there's no automatic register save, if a function uses BX, SI, DI etc. and the caller needs them preserved, push/pop them explicitly (this is the manual equivalent of callee-saved registers):

asm

```asm
    PUSH BX
    PUSH SI
    PUSH DI
    ; ... function body using BX, SI, DI ...
    POP  DI
    POP  SI
    POP  BX
```

---

### 9. Recursion

Recursion works exactly as in C — each call pushes a new stack frame. The classic factorial example, illustrating the recursive call stack explicitly:

asm

```asm
; int factorial(int n) { if (n <= 1) return 1; return n * factorial(n-1); }
FACTORIAL PROC
    PUSH BP
    MOV  BP, SP
    ; [BP+4] = n

    MOV  AX, [BP+4]
    CMP  AX, 1
    JG   RECURSE
    MOV  AX, 1               ; base case: return 1
    JMP  FACT_DONE

RECURSE:
    MOV  AX, [BP+4]
    DEC  AX
    PUSH AX                  ; push (n-1)
    CALL FACTORIAL             ; recursive call — new stack frame pushed
    ADD  SP, 2                 ; clean up the pushed arg (caller cleans here
                                 ; since we used a plain RET below, not RET n)
    MOV  BX, [BP+4]
    IMUL BX                    ; DX:AX = AX * BX  (n * factorial(n-1))
    ; assume result fits in AX

FACT_DONE:
    POP  BP
    RET
FACTORIAL ENDP
```

Each recursive call gets its **own** `[BP+4]` because BP is pushed/popped per call — this is precisely how C recursion works at the ABI level, just visible instead of hidden by the compiler. Stack depth is limited by `.STACK` size — deep recursion in a `.COM`/small `.EXE` will overflow the ~1-4KB stack you typically reserve, unlike C where the OS gives you a much larger default stack.

---

### 10. Pointers

A pointer is just an offset (near pointer, within current segment) or a segment:offset pair (far pointer). Registers `SI`, `DI`, `BX`, `BP` can be used for indirect addressing (like `*p` in C).

asm

```asm
.DATA
    val DW 1234h
    ptr DW ?          ; a "pointer" is just a word holding an offset

.CODE
    LEA BX, val         ; ptr = &val   (BX = offset of val)
    MOV ptr, BX

    MOV BX, ptr          ; load the pointer value
    MOV AX, [BX]           ; dereference: AX = *ptr
    MOV WORD PTR [BX], 99  ; *ptr = 99
```

**Pointer arithmetic** — since a word is 2 bytes, "advancing a pointer" over an array of words means adding 2, not 1 (compiler-implicit scaling in C, manual here):

asm

```asm
    ADD BX, 2         ; equivalent to ptr++ for an int* in 16-bit C
```

**Far pointers** (segment:offset, needed when addressing outside the current data segment, e.g. video memory at `B800:0000`):

asm

```asm
    MOV AX, 0B800h
    MOV ES, AX
    MOV DI, 0
    MOV BYTE PTR ES:[DI], 'A'    ; write directly to text video memory
```

`LES`/`LDS` load both a segment and offset in one instruction — the true equivalent of assigning a far pointer:

asm

```asm
    LES DI, farPtrVar    ; ES:DI = segment:offset stored at farPtrVar (dword)
```

---

### 11. Strings

Two string conventions coexist:

- **DOS-native**: `$`-terminated, only useful with `int 21h/09h`.
- **C-style**: null (`0`)-terminated, what you'll use for your own string routines.

x86 also has dedicated **string instructions** operating on `SI`(source)/`DI`(dest, in `ES`) with auto-increment/decrement controlled by `DF` (direction flag):

asm

```asm
CLD                 ; clear direction flag -> SI/DI increment (STD = decrement)

; strcpy equivalent
MOVSB               ; move byte [DS:SI] -> [ES:DI], SI++, DI++
                     ; prefix with REP to repeat CX times: REP MOVSB

; strlen equivalent (scan for null terminator)
STRLEN PROC
    PUSH DI
    PUSH CX
    MOV  DI, offset_of_string    ; ES:DI must point at the string
    MOV  AL, 0                     ; searching for null byte
    MOV  CX, 0FFFFh                ; max search length
    REPNE SCASB                    ; SCASB compares AL to [ES:DI], DI++,
                                     ; repeats while not-equal and CX != 0
    ; CX now holds (0FFFFh - actual_length - 1); invert to get length
    NOT  CX
    DEC  CX
    MOV  AX, CX                    ; result: string length
    POP  CX
    POP  DI
    RET
STRLEN ENDP
```

Manual concatenation/comparison (no `strcat`/`strcmp` built in — you write these once and reuse):

asm

```asm
; strcmp-like: compare byte-by-byte until mismatch or both null
STRCMP PROC
    ; SI = string1, DI = string2
CMP_LOOP:
    MOV AL, [SI]
    MOV BL, [DI]
    CMP AL, BL
    JNE NOT_EQUAL
    CMP AL, 0
    JE  EQUAL              ; both hit null together -> equal
    INC SI
    INC DI
    JMP CMP_LOOP
NOT_EQUAL:
    ; AL/BL differ, subtract to get comparison result if needed
EQUAL:
    RET
STRCMP ENDP
```

---

### 12. Arrays

Just a labeled contiguous block; indexing = base + (index × element_size), computed manually since there's no implicit type-size scaling like C's pointer arithmetic.

asm

```asm
.DATA
    arr DW 10, 20, 30, 40, 50      ; int arr[5]

.CODE
    ; arr[2] -> read
    MOV BX, 2
    SHL BX, 1                      ; multiply index by 2 (word size)
    MOV AX, arr[BX]                ; AX = 30

    ; arr[i] = 99  (write)
    MOV BX, 4
    MOV arr[BX], 99                ; arr[2] = 99  (BX already scaled)
```

**2D arrays** (row-major, like C): `arr[row][col]` at offset `(row * NUM_COLS + col) * elem_size`:

asm

```asm
.DATA
    ; int arr[3][4]
    arr2d DW 12 DUP(0)

.CODE
    ; access arr2d[row][col], row in AX, col in BX
    MOV CX, 4
    MUL CX               ; AX = row * 4
    ADD AX, BX             ; AX = row*4 + col
    SHL AX, 1               ; *2 for word size
    MOV SI, AX
    MOV DX, arr2d[SI]       ; DX = arr2d[row][col]
```

---

### 13. Structs

MASM's `STRUC`/`ENDS` maps directly onto C structs — a named layout of fields at fixed offsets.

asm

```asm
POINT STRUC
    xCoord  DW  0
    yCoord  DW  0
POINT ENDS

.DATA
    p1  POINT <5, 10>          ; equivalent to: struct Point p1 = {5, 10};
    p2  POINT <>                 ; zero-initialized

.CODE
    MOV AX, p1.xCoord             ; field access, like p1.x in C
    MOV p1.yCoord, 20

    LEA BX, p1
    MOV AX, [BX].POINT.xCoord     ; field access through a pointer, like p->x
```

Arrays of structs work the same way arrays of primitives do, just with `TYPE POINT` (4 bytes here) as the element size:

asm

```asm
.DATA
    points POINT 10 DUP(<>)      ; struct Point points[10];

.CODE
    MOV BX, 2
    MOV AX, TYPE POINT           ; AX = 4 (size of struct)
    MUL BX
    MOV SI, AX
    MOV AX, points[SI].xCoord    ; points[2].x
```

---

### 14. Floating point (x87 FPU)

Base 8086 has no float support — the original IBM PC needed an optional 8087 coprocessor; MASM assumes it's present when you use FPU instructions (DOSBox emulates it). The FPU has its own 8-register stack (`ST(0)`–`ST(7)`), not the general-purpose registers.

asm

```asm
.DATA
    a  DD  3.14           ; 4-byte (single-precision) float
    b  DD  2.0
    result DD ?

.CODE
    FLD  a          ; push a onto FPU stack -> ST(0) = a
    FLD  b            ; ST(0) = b, ST(1) = a
    FADD               ; ST(1) = ST(1) + ST(0), pop -> ST(0) = a+b
    FSTP result          ; store ST(0) into result, pop stack
```

Common FPU ops: `FADD`/`FSUB`/`FMUL`/`FDIV` (arithmetic), `FLD`/`FST`/`FSTP` (load/store), `FCOM`/`FCOMP` (compare — result must be moved to CPU flags via `FSTSW AX` then `SAHF` since FPU has its own status word, not the CPU `FLAGS`).

asm

```asm
; compare a and b (a > b?)
    FLD  a
    FCOMP b
    FSTSW AX             ; move FPU status word into AX
    SAHF                    ; copy AH into CPU FLAGS (so JG/JL etc. work)
    JA   A_GREATER          ; note: FPU comparisons map to unsigned-style jumps
```

If you don't want to deal with the FPU, the common alternative in 16-bit DOS programming is **fixed-point arithmetic** — represent a fraction as an integer scaled by e.g. 100 or 256, and adjust manually after multiply/divide. Worth knowing since a lot of period DOS code (games especially) avoided the FPU entirely for speed.

---

### 15. Reading date and time (BIOS/DOS services)

**`int 21h`, function 2Ah — get system date:**

asm

```asm
    MOV AH, 2Ah
    INT 21h
    ; CX = year, DH = month, DL = day, AL = day of week
```

**`int 21h`, function 2Ch — get system time:**

asm

```asm
    MOV AH, 2Ch
    INT 21h
    ; CH = hour, CL = minute, DH = second, DL = hundredths of a second
```

Example: print current time as HH:MM:SS (reusing `PRINT_NUM`-style digit conversion, or simpler since we know it's exactly 2 digits):

asm

```asm
    MOV AH, 2Ch
    INT 21h

    MOV AL, CH            ; hour
    CALL PRINT_TWO_DIGITS
    MOV DL, ':'
    MOV AH, 02h
    INT 21h

    MOV AL, CL              ; minute
    CALL PRINT_TWO_DIGITS
```

(`PRINT_TWO_DIGITS` would do `AAM`-style or `DIV 10` decomposition into tens/units then print each `+'0'`.)

---

### 16. File I/O

DOS file handles work much like C's `FILE*`/POSIX `fopen`/`read`/`write`, via `int 21h`.

asm

```asm
.DATA
    filename DB 'TEST.TXT', 0
    fileHandle DW ?
    buffer DB 100 DUP(?)
    bytesRead DW ?

.CODE
    ; open file (like fopen(name, "r"))
    MOV AH, 3Dh
    MOV AL, 0             ; access mode: 0 = read-only
    LEA DX, filename
    INT 21h
    JC  OPEN_ERROR          ; CF set on error, error code in AX
    MOV fileHandle, AX

    ; read file (like fread)
    MOV AH, 3Fh
    MOV BX, fileHandle
    MOV CX, 100             ; bytes to read
    LEA DX, buffer
    INT 21h
    JC  READ_ERROR
    MOV bytesRead, AX        ; actual bytes read returned in AX

    ; write to a file (like fwrite) — function 40h, same register setup as 3Fh
    MOV AH, 40h
    MOV BX, fileHandle
    MOV CX, bytesRead
    LEA DX, buffer
    INT 21h

    ; close file (like fclose)
    MOV AH, 3Eh
    MOV BX, fileHandle
    INT 21h

OPEN_ERROR:
READ_ERROR:
```

Creating a new file: function `3Ch` (`CREATE FILE`, similar setup, `CX` = file attributes). Error checking convention throughout DOS `int 21h` calls: **carry flag (CF) set means error**, error code returned in `AX`.

---

### 17. Other things worth knowing

**Screen/cursor control via `int 10h` (BIOS video services)** — the closest thing to `<conio.h>`'s `clrscr()`/`gotoxy()`:

asm

```asm
    ; set cursor position
    MOV AH, 02h
    MOV BH, 0             ; page number
    MOV DH, 10              ; row
    MOV DL, 20               ; column
    INT 10h

    ; clear screen (scroll entire window up by 0 lines = clear + fill)
    MOV AH, 06h
    MOV AL, 0
    MOV BH, 07h              ; attribute (white on black)
    MOV CX, 0                 ; upper-left corner
    MOV DX, 184Fh              ; lower-right corner (row 24, col 79)
    INT 10h
```

**Interrupts vs function calls** — `int 21h` isn't a `CALL`; it's a software interrupt that looks up a handler address in the **interrupt vector table** (first 1KB of memory, 256 entries × 4 bytes each, at physical address 0). This is architecturally distinct from `CALL`/`RET` and matters if you ever write your own interrupt handler (`int 09h` for keyboard, etc.) — you'd save the old vector, install yours, and chain to the original.

**Macros (`MACRO`/`ENDM`)** — MASM's text-substitution mechanism, closer to C preprocessor macros than to functions (no call overhead, expanded inline at assembly time):

asm

```asm
PRINT_STR MACRO msg
    MOV AH, 09h
    LEA DX, msg
    INT 21h
ENDM

; usage:
    PRINT_STR greeting
```

**`PROC`/local labels and scoping** — MASM doesn't enforce label scoping the way C scopes variables; all labels are effectively global unless you use `LOCAL` inside a macro or `.MODEL` with a proc-local option. Naming collisions across procedures are a real hazard — a common convention is prefixing labels per-procedure (`FACT_LOOP`, `FACT_DONE`, etc., as used above).

**Signed vs unsigned is a matter of instruction choice, not declaration** — the same bit pattern in a register is interpreted as signed or unsigned purely based on whether you used `JG`/`IDIV`/`IMUL` (signed) or `JA`/`DIV`/`MUL` (unsigned). Get this wrong and your loop bounds or arithmetic silently misbehave — there's no compiler to catch a type mismatch for you.

**Assembling and running in DOSBox:**

```
MASM myprog.asm;
LINK myprog.obj;
myprog.exe
```

(or with a modern MASM/TASM setup, often a single `ml myprog.asm` for MASM32-style toolchains — but classic DOSBox MASM 5.x/6.x uses the two-step MASM+LINK shown above.)

---

### Summary: what goes where

```
.MODEL / .STACK        <- memory model + stack size declaration
.DATA                    <- all variables, strings, arrays, structs, constants
.CODE                     <- all PROCs; MAIN sets DS, does work, exits via 4Ch
    MAIN PROC
        (set DS)
        (call your functions / inline logic)
        (exit via AH=4Ch)
    MAIN ENDP
    (other PROCs — helper functions, string routines, etc.)
END MAIN   
```