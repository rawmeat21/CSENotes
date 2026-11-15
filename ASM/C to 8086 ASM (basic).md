Before trying to convert C to MASM, you should remember this structure of the assembly code:


```
.model small
.stack 100h
.data

;define global variables here

.code

f1 proc

;base regiter push
push bp
mov bp, sp

;local variables allocate
sub sp, x ;x=2*(number of local variables needed)

;register save
push si
push dx
push di

;body of function, write the logic below
...

;save return value
...

mov ax, [bp-2] ;bp-2 will always contain our return value and after returning,
               ;ax will always contain the return value from the caller's 
               ;perspective

;register restore
push di
push dx
push si

;local variables deallocate
mov sp, bp

;base register pop
pop bp

;return
ret

f1 endp
```

Sections which are followed by a `...` are incomplete sections, their full content will vary as program logic changes.

The stack will look like this:

```
----------
f2 local var 1            <-- f2_bp - 4
----------
f2 local var 2            <-- f2_bp - 2
----------
f1's local base pointer   <-- f2_bp + 0
----------
return address of f1      <-- f2_bp + 2
----------
f2 arg1                   <-- f2_bp + 4
----------
f2 arg2                   <-- f2_bp + 6
----------
f1 saved registers        
----------
f1 local var 1            <-- f1_bp - 4
----------
f1 local var 2            <-- f1_bp - 2
----------
main's local base pointer <-- f1_bp + 0
----------
return address of main    <-- f1_bp + 2
----------
f1 arg1                   <-- f1_bp + 4
----------
f1 arg2                   <-- f1_bp + 6

```

Here, I have drawn the stack after `main` has called `f1` inside it, and `f1` has called `f2` inside it. Btw, `f1_bp` is what the `bp` register holds when `f1` is being executed. Same for `f2_bp`.

What I need you to understand is that for any function, the arguments supplied to it are at `bp+4`, `bp+6`, `bp+8`, etc. and it's local variables are at `bp-2`, `bp-4`, `bp-6`, `bp-8`, etc.


**How to call other functions?**

When a function `f1` calls another function `f2`, it should do so in the following manner:

```
;;save registers, I have written a full list but you should only save the 
;;registers you are using
 
push ax
push cx
push dx

;;push arguments
push ax
push dx

call f2

;;pop arguments
pop dx
pop ax

;;restore registers
pop dx
pop cx
pop ax
```


**What about blocks of code?**

There is no concept of scope in assembly. So, basically we'll ignore blocks completely and give each variable a place in the stack. Watch and learn:

```c
int process_data(int n) {
    int total = 0;                  // bp-2
    int status = 1;                 // bp-4

    if (n > 0) {
        int i;                      // bp-6
        long acc = 0;               // bp-8 (note that long means 4 bytes, so it takes 4 bytes in stack, causing the next offset to be bp-12)

        for (i = 0; i < n; i++) {
            int step = i * 2;       // bp-12
            int buf[2];             // bp-14 (again 2 int values, so 4 bytes)
            buf[0] = step;
            buf[1] = step + 1;
            acc += buf[0] + buf[1];
        }

        {
            int bonus = 10;         // bp-18
            acc += bonus;
        }

        total = (int)acc;
    } else {
        int err_code = -1;          // bp-20
        int retries = 3;            // bp-22
        int fallback[4];            // bp-24
        fallback[0] = err_code;
        total = fallback[0] + retries;
        int k;                      // bp-32
    }

    return total + status;
}
```

This is kinda inefficient, but it works. We reached till `bp-32`, which means we have 16 local variables. So, in stack we need 2 * 16 = 32 bytes of space:

```
sub sp, 32
```

Now, you know how to assign offsets to each variable. Things just got a lot easier. Let's look how.

Suppose you would like to initialise `status` to 1 like in the code. Simply do:

```
mov word ptr [bp-4], 1
```

That was easy.

Ok, what about something like `acc+=bonus`? 

```
mov bx, [bp-18]
add word ptr [bp-8], bx
```

The rule you can follow is: just move the variable in RHS into `bx` register and perform the operation.


Let's deal with the loops then:








