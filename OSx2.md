# Processes

It is a running program. 

HOW TO PROVIDE THE ILLUSION OF MANY CPUS?  
Although there are only a few physical CPUs available, how can the  OS provide the illusion of a nearly-endless supply of said CPUs?

The OS creates this illusion by **virtualizing the CPU**

By running one process, then stopping it and running another, and so forth, the OS can  
promote the illusion that many virtual CPUs exist when in fact there is only one physical CPU (or a few). This basic technique, known as time sharing of the CPU, allows users to run as many concurrent processes as they would like; the potential cost is performance, as each will run more slowly if the CPU(s) must be shared.

**Time sharing** is a basic technique used by an OS to share a resource. By allowing the resource to be used for a little while by one entity, and then a little while by another, and so forth, the resource in question (e.g., the CPU, or a network link) can be shared by many. 

The counterpart of time  sharing is **space sharing**, where a resource is divided (in space) among those who wish to use it. For example, disk space is naturally a space-  
shared resource; once a block is assigned to a ﬁle, it is normally not assigned to another ﬁle until the user deletes the original ﬁle.

![[Pasted image 20260603233018.png]]


## How a process is created

![[Pasted image 20260603233043.png]]

1. The ﬁrst thing that the OS must do to run a program is to load its code  and any static data (e.g., initialized variables) into memory, into the address space of the process
2. Some memory must be allocated for the program’s run-time stack. Programs use the stack for local variables, function parameters, and return addresses; the OS allocates  this memory and gives it to the process. The OS will also likely initialize the stack with arguments; speciﬁcally, it will ﬁll in the parameters to the main() function, i.e., argc and the argv array.
3. The OS may also allocate some memory for the program’s heap. The heap is needed for data structures such as linked lists, hash tables, trees. 
4. The OS will also do some other initialization tasks related to input/output (I/O). For example, in UNIX systems, each process by default has three open ﬁle descriptors, for standard input, output, and error
5. Start the program running at the entry point, namely main(). By jumping to the main() routine, the OS transfers control of the CPU to the newly-created process, and thus the program begins its execution.

## Process states

