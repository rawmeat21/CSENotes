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

![[Pasted image 20260603233043.png]]

