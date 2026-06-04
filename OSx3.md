# Direct Execution

## Limited Directed Execution protocol (LDE)- Just run the process

![[Pasted image 20260604141951.png]]


When the OS wishes to start a program running, it creates a process entry for it in a process list, allocates some memory for it, loads the program code into memory (from disk), locates its entry point (i.e., the main() routine or something similar), jumps to it, and starts running the user’s code

Problem1: How to prevent a process from doing anything it wants?
Problem2: How to pause it from running (time sharing)?



**WHY SYSTEM CALLS LOOK LIKE PROCEDURE CALLS

You may wonder why a call to a system call, such as open() or read(), looks exactly like a typical procedure call in C; that is, if it looks just like a procedure call, how does the system know it’s a system call, and do all the right stuff? The simple reason: it is a procedure call, but hidden inside that procedure call is the famous **trap instruction**. More specifically,
when you call open() (for example), you are executing a procedure call into the C library. Therein, whether for open() or any of the other system calls provided, the library uses an agreed-upon calling convention with the kernel to put the arguments to open() in well-known locations (e.g., on the stack, or in specific registers), puts the system-call number
into a well-known location as well (again, onto the stack or a register), and then executes the aforementioned trap instruction. The code in the library after the trap unpacks return values and returns control to the program that issued the system call. Thus, the parts of the C library that make system calls are hand-coded in assembly, as they need to carefully follow convention in order to process arguments and return values correctly, as well as execute the hardware-specific trap instruction. And now you know why you personally don’t have to write assembly code to trap into an OS; somebody has already written that assembly for you.


## Restricted Operations

What if the process wishes to perform some kind of restricted operation, such as issuing an I/O request to a disk, or gaining access to more system resources such as CPU or memory?

The approach we take is to introduce a new processor mode, known as **user mode**; code that runs in user mode is restricted in what it can do. For example, when running in user mode, a process can’t issue I/O requests; doing so would result in the processor raising an exception; the OS would then likely kill the process. 

In contrast to user mode is **kernel mode**, which the operating system (or kernel) runs in. In this mode, code that runs can do what it likes, including privileged operations such as issuing I/O requests and executing all types of restricted instructions.

**Special instructions to trap into the kernel and return-from-trap back to user-mode programs are also provided, as well as instructions that allow the OS to tell the hardware where the trap table resides in memory.

What should a user process do when it wishes to perform some kind of privileged operation, such as reading from disk? To enable this, virtually all modern hard-  
ware provides the ability for user programs to **perform a system call**.

### System calls

System calls allow the kernel to carefully expose certain key pieces of functionality to  
user programs, such as accessing the ﬁle system, creating and destroying processes, communicating with other processes, and allocating more memory.

To execute a system call, a program must execute a special trap instruction. This instruction simultaneously jumps into the kernel and raises the privilege level to kernel mode; once in the kernel, the system can now perform whatever privileged operations are needed (if allowed), and thus do the required work for the calling process. When ﬁnished, the OS calls a special return-from-trap instruction, which returns into the calling user program while simultaneously reducing the privilege level back to user mode



**How does the trap know which code to run inside the OS?** 

Clearly, the calling process can’t specify an address to jump to (as you would when making a procedure call); doing so would allow programs to jump anywhere into the  
kernel which clearly is a Very Bad Idea. Thus the kernel must carefully  
control what code executes upon a trap. 

The kernel does so by setting up a trap table at boot time. When the machine boots up, it does so in privileged (kernel) mode, and thus is free to conﬁgure machine hardware as need be. One of the ﬁrst things the OS thus does is to tell the hardware what code to run when certain exceptional events occur. For example, what code should run when a hard-disk interrupt takes place, when a keyboard interrupt occurs, or when a program makes a system call? The OS informs the hardware of the locations of these trap handlers, usually with some kind of special instruction. Once the hardware is informed, it remembers the location of these handlers until the machine is next rebooted, and thus the hardware  
knows what to do (i.e., what code to jump to) when system calls and other exceptional events take place.



To specify the exact system call, a **system-call number** is usually assigned to each system call. The user code is thus responsible for placing the desired system-call number in a register or at a speciﬁed location on the stack; the OS, when handling the system call inside the trap handler, examines this number, ensures it is valid, and, if it is, executes the corresponding code. This level of indirection serves as a form of protection; user code cannot specify an exact address to jump to, but rather must request a particular service via number.


## LDE phases

There are two phases in the limited direct execution (LDE) protocol. 

1. In the ﬁrst (at boot time), the kernel initializes the trap table, and the CPU remembers its location for subsequent use. The kernel does so via a privileged instruction. 
2. In the second (when running a process), the kernel sets up a few things (e.g., allocating a node on the process list, allocating memory) before using a return-from-trap instruction to start the execution of the process; this switches the CPU to user mode and begins running the process. When the process wishes to issue a system call, it traps back into the OS, which handles it and once again returns control via a return-from-trap to the process. The process then completes its work, and returns from main(); this usually will return into some stub code which will properly exit the program (say, by calling the exit() system call, which traps into the OS). At this point, the OS cleans up and we are done.

![[Pasted image 20260604145023.png]]


## Switching between processes

If a process is running on the CPU, this by deﬁnition means the OS is not running. If  
the OS is not running, how can it do anything at all?


### Cooperative Approach: Wait For System Calls

In this style, the OS trusts the processes of the system to behave reasonably. Processes  
that run for too long are assumed to periodically give up the CPU so that the OS can decide to run some other task.

Most processes transfer control of the CPU to the OS quite frequently by making system calls, for example, to open a ﬁle and subsequently read it, or to send a message to another machine, or to create a new process. 

**Systems like this often include an explicit yield system call, which does nothing except to transfer control to the OS so it can run other processes

Applications also transfer control to the OS when they do something illegal. For example, if an application divides by zero, or tries to access memory that it shouldn’t be able to access, it will generate a trap to the OS. The OS will then have control of the CPU again and prolly terminate the process.

But, what happens if I run an endless loop? 

### Non-Cooperative Approach: The OS Takes Control

How can the OS gain control of the CPU even if processes are not being cooperative (not generating syscalls)? What can the OS do to ensure a rogue process does not take over the machine?

Solution: a **Timer Interrupt**

A timer device can be programmed to raise an interrupt every so many milliseconds; when the interrupt is raised, the currently running process is halted, and a pre-conﬁgured interrupt handler in the OS runs. So, the OS has regained control of the CPU, and thus can do what it pleases: stop the current process, and start a different one.

OS must inform the hardware of which code to run when the timer interrupt occurs; thus,  
at boot time, the OS does exactly that. Also during the boot sequence, the OS must start the timer, which is of course a privileged operation. Once the timer has begun, the OS can thus feel safe in that control will eventually be returned to it, and thus the OS is free to run user programs.

Now that the OS has regained control, a decision has to be made: whether to continue running the currently-running process, or switch to a different one. This decision is made by a part of the operating system known as the scheduler.

If the decision is made to switch, the OS then executes a low-level piece of code which we refer to as a **context switch**

A context switch is conceptually simple: all the OS has to do is save a few register values  
for the currently-executing process (onto its kernel stack, for example) and restore a few for the soon-to-be-executing process (from its kernel stack). By doing so, the OS thus ensures that when the return-from-trap instruction is ﬁnally executed, instead of returning to the process that was running, the system resumes execution of another process.

![[Pasted image 20260604151519.png]]

In the timer interrupt handler, the OS decides to switch from running Process A to Process B. At that point, it calls the switch() routine, which carefully saves current register values (into the process structure of A), restores the registers of Process B (from its process  
structure entry), and then switches contexts, speciﬁcally by changing the stack pointer to use B’s kernel stack.

![[Pasted image 20260604151824.png]]


