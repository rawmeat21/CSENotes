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



```c
	int i;
    for (i = 0; i < N; i++) {
	    int j;
        for (j = 0; j < M; j++) {
            // do something
        }
    }
```

```
    mov word ptr [bp-2], 0      ; i = 0
outer_cond:
    mov ax, [bp-2]
    cmp ax, [bp+4]              ; i < N
    jge outer_end

    mov word ptr [bp-4], 0      ; j = 0
    
    ; do something before entering inner loop
    ...
    
inner_cond:
    mov ax, [bp-4]
    cmp ax, [bp+6]              ; j < M
    jge inner_end
    
    ; do something in inner loop
    ...
	
    ; continue in inner loop  --> jmp inner_next
    ; break in inner loop     --> jmp inner_end

inner_next:
    inc word ptr [bp-4]         ; j++
    jmp inner_cond
inner_end:

	; do something after inner loop is complete
	...
	
    ; continue in outer loop  --> jmp outer_next
    ; break in outer loop     --> jmp outer_end

outer_next:
    inc word ptr [bp-2]         ; i++
    jmp outer_cond
outer_end:
```


Let's look at an example, we will convert the following C code into assembly:

```c

void input_str(char* s)// s is bp+4
{
	int i;// bp-2
	for(i=0;;++i)
	{
		char c;// we don't really need to allocate this, we can take input in ax
		// take input in c
		
		if(c=='\n') break;// we'll compare with code 0dh (which is carriage return)
		
		s[i]=c;
	}
	
	s[i]='$' // add in our null terminator
}
```

```
input_str proc

;base push
push bp
mov bp, sp

;local vars allocate
sub sp, 2

;register save
push bx
push si
push di

;body
mov si, [bp+4]                  ;string pointer is supplied in stack
mov word ptr [bp-2], 0          ;i=0
l1:

mov ah, 01h                     ;take input
int 21h

cmp al, 0dh                     ;is input char = '\n'?
je l1_end

mov bx, [bp-2]                  ;bx=i
mov byte ptr [si+bx], al        ;s[i]=input char

inc word ptr [bp-2]             ;i++

jmp l1

l1_end:
mov bx, [bp-2]                  ;bx=i
mov byte ptr [si+bx], '$'       ;s[i]='$'

;save return value

;register restore
pop di
pop si
pop bx

;local vars deallocate
mov sp, bp

;base pop
pop bp

;return
ret

input_str endp
```

The main function looks like:

```c
int main()
{
	char str[50];
	input_str(str);
}
```

```
main proc

push bp
mov bp, sp

sub sp, 100

push bx
push si
push di

lea si, [bp-100]      ;bp-100=str+0, bp-2=str+49

push si
call input_str
pop si

pop di
pop si
pop bx

mov sp, bp

pop bp

mov ah, 4ch
int 21h

main endp

end main
```

Main thing to take away from here is: Suppose I create `str[50]` in main. This does nothing but put 50 2-byte values on the stack. But, how can we access those values? 

Well, its intuitive that `str[3]` has a higher address than `str[0]`. This holds here too. lower indices are at a lower address than higher indices. That's why `str[0]` is at `bp-100`  and `str[49]` is at `bp-2`.


So far, so good. Let's try writing a sorting algorithm. The simplest is a bubble sort. The code is literally just this:


```c
/*
arr=[bp+4], n=[bp+6]
i=[bp-2]
j=[bp-4]
*/

void sort(int* arr, int n)
{
	int i;
	for(i=0;i<n;++i)
	{
		int j;
		for(j=0;j<n-1;++j)
		{
			if(arr[j]>arr[j+1]) 
			{
				swap(&arr[j], &arr[j+1]);
			}
		}
	}
}
```


```
sort proc

push bp
mov bp, sp

sub sp, 4

push bx
push si
push di

mov si, [bp+4]
mov word ptr [bp-2], 0

outer:
mov bx, [bp+6]
cmp word ptr [bp-2], bx

jge outer_end

mov word ptr [bp-4], 0

inner:

mov bx, [bp+6]
dec bx

cmp word ptr [bp-4], bx
jge inner_end

mov bx, [bp-4]
shl bx, 1
mov ax, ss:[si+bx]

mov bx, [bp-4]
inc bx
shl bx, 1
mov cx, ss:[si+bx]

cmp ax, cx
jle nope

mov bx, [bp-4]
shl bx, 1
lea ax, ss:[si+bx]
push ax

add bx, 2
lea ax, ss:[si+bx]
push ax

call swap

pop ax
pop ax

nope:
inc word ptr [bp-4]
jmp inner

inner_end:

inc word ptr [bp-2]
jmp outer

outer_end:

pop di
pop si
pop bx

mov sp, bp

pop bp

ret

sort endp

```


How does swap look?

```c
void swap(int* a, int * b)
{
	int c=*a;
	*a=*b;
	*b=c;
}
```


```
swap proc

push bp
mov bp, sp

push bx
push si
push di

mov bx, [bp+4]   ;bx=an address
mov cx, ss:[bx]     ;cx=value at that address

mov bx, [bp+6]   ;bx=the other address
mov ax, ss:[bx]     ;ax=value at the other address

mov bx, [bp+4]
mov ss:[bx], ax

mov bx, [bp+6]
mov ss:[bx], cx

pop di
pop si
pop bx

mov sp, bp
pop bp

ret
swap endp
```


Now, main:

```c
int main()
{
	int arr[50];
	input_array(arr, 12); // imagine this exists
	sort(arr, 12);
	print_array(arr, 12); // imagine this exists
}
```

```
main proc

push bp
mov bp, sp

sub sp, 100

push si
push di
push bx

lea si, [bp-100]
mov ax, 0ch      ;length=12

push ax
push si
call input_array
pop si
pop ax

push ax
push si
call sort
pop si
pop ax

push ax
push si
call print_array
pop si
pop ax

pop bx
pop di
pop si

mov sp, bp
pop bp

mov ah, 4ch
int 21h

main endp
```







