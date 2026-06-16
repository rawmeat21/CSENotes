### Why do we need system calls

There are two main reasons why such instructions are not available directly for the process:

- **Protection**: The OS must do many checks to evaluate the right of the process to request such an instruction.
- The second one is that the OS needs to update its data structures when the instruction is performed.


### user mode -> kernel mode

When a process is executing, it can run in two modes: user mode or kernel mode. 

It runs in user mode when it is executing normal CPU instructions that don’t require a privilege such as _jump to address, load from memory, write to memory, …_ However, when the process has to execute privileged instructions it should give the hand to the OS to _execute on behalf of it_, this is what we call kernel mode.

To switch from user mode to kernel mode, there is no means but _interrupts_.

In general, an interrupt can be a hardware interrupt such as a timer interrupt that makes the CPU switch to kernel mode and execute a process switch for example, or it can be a software interrupt which are caused by the program itself, such as a division by zero, a page fault or a system call.


![[Pasted image 20260616213054.png]]

