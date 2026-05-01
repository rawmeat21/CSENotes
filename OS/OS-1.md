
Operating system- interface between hardware and user
It is a way to access hardware and make it do things like access keyboard, bluetooth, anything
OS provide an API for devs to interact with the hardware through it

![[Pasted image 20260424001904.png]]


**An OS needs to have ==program execution== at minimum


![[Pasted image 20260424002851.png]]

Kernel- Core of the OS. Everything that the OS does is stored as functions in the kernel
Shell- Interface to use those functionalites. It is a frontend for kernel

There are 2 types of interfaces:

GUI
CLI

System call- Literally calling the functions of the kernel is making a system call.
Every program makes system calls to interact with the kernel/OS

Process- is an instance of a program in memory

But a process cannot do whatever tf it wants, in other words, we need protection

![[Pasted image 20260424003831.png]]

![[Pasted image 20260424003924.png]]
