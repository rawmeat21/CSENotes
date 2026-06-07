There is a body of software that is responsible for making it easy to run programs (even allowing you to seemingly run many at the same time), allowing programs to share memory, enabling programs to interact with devices, and other fun stuff like that. That body of software is called the operating system (OS).

The primary way the OS does this is through a general technique that  
we call **virtualization**. That is, the OS takes a physical resource (such as  
the processor, or memory, or a disk) and transforms it into a more gen-  
eral, powerful, and easy-to-use virtual form of itself. Thus, we sometimes  
refer to the operating system as a virtual machine.

Because virtualization allows many programs to run (thus sharing the CPU), and many programs to concurrently access their own instructions and data (thus sharing memory), and many programs to access devices (thus sharing disks and so forth), the OS is sometimes known as  a **resource manager**.

Each of the CPU, memory, and disk is a resource  of the system; it is thus the operating system’s role to manage those resources, doing so efﬁciently.

To run programs, and stop them, and otherwise tell the OS which programs to run, there need to be some interfaces (APIs) that you can use to communicate your desires to the OS

![[Pasted image 20260603230836.png]]

![[Pasted image 20260603230900.png]]
Single instance

![[Pasted image 20260603230913.png]]
Multiple instances

OS is virtualizing memory. Each process accesses its own private virtual address space
(or address space), which the OS somehow maps onto the physical memory of the machine. A memory reference within one running program does not affect the address space of other processes (or the OS itself); as far as the running program is concerned, it has physical memory all to itself.

## Goals of OS:


1. Provide high performance; another way to say this is our goal is to minimize the overheads of the OS
2. Provide protection between applications, as well as between the OS and applications
3. Security, energy efficiency, reliability

