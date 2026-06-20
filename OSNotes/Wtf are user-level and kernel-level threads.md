## User Level Threads

-> Threads that are managed entirely by the user-level thread library without any direct involvement of the operating system kernel.

-> execute one at a time

![[Pasted image 20260620112508.png]]

### Features

-> The User-level Threads are implemented by the user-level software. 

-> These threads are created and managed by the thread library, which the operating system provides as an API for creating, managing, and synchronizing threads.

-> Faster than the kernel-level threads, it is basically represented by the program counter, stack, register, and PCB.

-> Typically employed in scenarios where fine control over threading is necessary, but the overhead of kernel threads is not desired.

-> Useful in systems that lack native multithreading support, allowing developers to implement threading in a portable way.

-> **Example:** User threads library includes POSIX threads, Mach C-Threads


## Kernel-Level Threads

-> Managed and scheduled directly by the operating system’s kernel.

-> The kernel handles all operations like creation, scheduling, suspension, and termination, giving it full control. This ensures proper coordination and complete awareness of all threads within a process.

-> Each kernel-level thread has its own context, including information about the thread's status, such as its name, group, and priority.

-> Kernel threads CAN share the code segment

-> **Example:** The example of Kernel-level threads are Java threads, POSIX thread on Linux, etc.

| User-Level Thread (ULT)                     | Kernel-Level Thread (KLT)                   |
| ------------------------------------------- | ------------------------------------------- |
| Implemented by user-level libraries         | Implemented by the Operating System         |
| Not recognized by the OS                    | Recognized by the OS                        |
| Fast context switching with less overhead   | Slower context switching with more overhead |
| Blocking 1 thread blocks the entire process | Only the blocked thread is affected         |
| Limited use of multiprocessing              | Fully utilizes multiprocessing              |
| Fast and simple creation and management     | Slower and more complex management          |
| Threads share the same address space        | Each thread has its own address space       |
| More portable, works on any OS              | OS-dependent and less portable              |

