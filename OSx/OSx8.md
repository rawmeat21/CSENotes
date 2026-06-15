### Hardware-based address translation

or just address translation. 

With address translation, the hardware transforms each memory access (e.g., an instruction fetch, load, or store), changing the virtual address provided by the instruction to a physical address where the desired information is actually located. 

Thus, on each and every memory  reference, an address translation is performed by the hardware to redirect application memory references to their actual locations in memory.

![[Pasted image 20260615180753.png]]

![[Pasted image 20260615180815.png]]

![[Pasted image 20260615180833.png]]

(what the process thinks its address space is)

![[Pasted image 20260615180845.png]]

(what it actually is, process is at address 32kb, not 0kb)

### Base and bounds approach (Dynamic relocation, hardware based)

We have 2 registers: Base and Bound 

Base- will hold the starting address of the process's start (where 0th address of the process starts)

Bounds / Limit- It either holds the ending address of the process OR the size of the address space of the process.

In this setup, each program is written and compiled as if it is loaded at address zero. However, when a program starts running, the OS decides where in physical memory it should be loaded and sets the base register to that value. In the example above, the OS decides to load the process at physical address 32 KB and thus sets the base register to this value.

**physical address = virtual address + base**

![[Pasted image 20260615182501.png]]

128 is the address on the process' address space. 

The program counter (PC) is set to 128; when the hardware needs to fetch this instruction, it ﬁrst adds the value to the base register value of 32 KB (32768) to get a physical address of 32896; the hardware then fetches the instruction from that physical address. Next, the processor begins executing the instruction.

At some point, the process then issues the load from virtual address 15 KB, which the processor takes and again adds to the base register (32 KB), getting the ﬁnal physical address of 47 KB and thus the desired contents.

Because this relocation of the address happens at runtime, and because we can move address spaces even after the process has started running, the technique is often referred to as dynamic relocation.

**Protection is provided through the bounds register.**

**MMU (Memory management unit) - Part of the CPU that does address translation.**

The CPU must be able to generate exceptions in situations where a user program tries to access memory illegally (with an address that is “out of bounds”); in this case, the CPU should stop executing the user program and arrange for the OS “out-of-bounds” exception handler to run.


### OS issues

-> The OS must take action when a process is created, ﬁnding space for its address space in memory.

Given our assumptions that each address space is 
(a) smaller than the size of physical memory and  
(b) the same size, 

this is quite easy for the OS; it can simply view physical memory as an array of slots, and track whether each one is free or in use. When a new process is created, the OS will have to search a data  structure (often called a **free list**) to ﬁnd room for the new address space  
and then mark it used.

![[Pasted image 20260615183657.png]]

-> the OS must do some work when a process is terminated (i.e., when it exits gracefully, or is forcefully killed because it misbehaved), reclaiming all of its memory for use in other processes or the OS. Upon termination of a process, the OS thus puts its memory back on the free list, and cleans up any associated data structures as need be.

-> The OS must also perform a few additional steps when a context switch occurs. 

There is only one base and bounds register pair on each CPU, after all, and their values differ for each running program, as each program is loaded at a different physical address in memory. 

Thus, the OS must **save and restore** the base-and-bounds pair when it switches between processes. Speciﬁcally, when the OS decides to stop running a process, it must save the values of the base and bounds registers to memory, in some per-process structure such as the process structure or process control block (PCB). 

Similarly, when the OS resumes a running process (or runs it the ﬁrst time), it must set the values of the base and bounds on the CPU to the correct values for this process.



#### Note that when a process is stopped (i.e., not running), it is possible for the OS to move an address space from one location in mem-ory to another rather easily. 

To move a process’s address space, the OS ﬁrst deschedules the process; then, the OS copies the address space from the current location to the new location; ﬁnally, the OS updates the saved base register (in the process structure) to point to the new location. 

When the process is resumed, its (new) base register is restored, and it begins running again, oblivious that its instructions and data are now in a completely new spot in memory.


-> The OS must provide exception handlers. The OS installs these handlers at boot time (via  privileged instructions). For example, if a process tries to access memory outside its bounds, the CPU will raise an exception; the OS must be prepared to take action when such an exception arises. 

The common reaction of the OS will be one of hostility: it will likely terminate the offending process. The OS should be highly protective of the machine it is running, and thus it does not take kindly to a process trying to access memory or execute instructions that it shouldn’t. 

### Disadvantage:

Unfortunately, this simple technique of dynamic relocation does have  
its inefﬁciencies. For example, as you can see, the relocated process is using physical memory from 32 KB to 48 KB; however, because the process stack and heap are not too big, all of the space between the two is simply wasted. This type of waste is usually called internal fragmentation, as the space inside the allocated unit is not all used  
(i.e., is fragmented) and thus wasted.





