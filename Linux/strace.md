Snoop on the process at a lower level.

This command displays every system call that a process makes and every signal it receives.

Not only does strace show you the name of every system call made by the process, but it also decodes the arguments and shows the result code that the kernel returns.

![[Pasted image 20260911234332.png]]

A glimpse of strace output on my reciever file for CN assignment 2. 

**System call output can often reveal errors that are not reported by the process itself**. For example, filesystem permission errors or socket conflicts are usually quite obvious in the output of strace or truss. Look for system calls that return error indications, and check for nonzero values.

