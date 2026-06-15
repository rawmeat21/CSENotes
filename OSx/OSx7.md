## Address spaces

The address space of a process contains all of the memory state of the  
running program.

It contains things like:

code of program
the program's stack
the heap

![[Pasted image 20260615154625.png]]

This placement of stack and heap is just a convention; you could arrange the address space in a different way if you’d like.

When we describe the address space, what we are describing is the abstraction that the OS is providing to the running program. The program really isn’t in memory at physical addresses 0 through 16KB; rather it is loaded at some arbitrary physical address(es).

![[Pasted image 20260615154916.png]]

When the OS does this, we say the OS is virtualizing memory, because the running program thinks it is loaded into memory at a particular address (say 0) and has a potentially very large address space (say 32-bits or 64-bits), but the reality is quite different.


### Goals of VM

Transparency- The OS should implement virtual memory in a way that is invisible to the running program.

Efficiency- OS should strive to make the virtualization as efficient as possible, both in terms of time (i.e., not making programs run much more slowly) and space (i.e., not using too much memory for structures needed to support virtualization).

Protection- The OS should make sure to protect processes from one another as well as the OS itself from processes. When one process performs a load, a store, or an instruction fetch, it should not be able to access or affect in any way the memory contents of any other process or the OS itself (that is, anything outside its address space)



