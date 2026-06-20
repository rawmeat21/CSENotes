
**Concurrency vs Parallelism**

- _Concurrent processing_ means that different tasks are progressing at the same time. This does not mean that they are running in parallel. Concurrency works by scheduling different threads to make the maximum utilization of a single CPU.

- _Parallelism_ means that different threads are being processed at the same time. This can be done by distributing the different threads across different CPUs.

## What is a Thread?

![[Pasted image 20260430143057.png]]

A thread is an independent path of a process. It exists within the address space of a process. It represents a flow of control within a process.

**Types of Threads**

1. User-Level Threads: Managed by user libraries, faster to switch, but kernel unaware.
2. Kernel-Level Threads: Managed by OS kernel, slower but more powerful.
3. Hybrid Model: Combines benefits of both.

**Advantages of Threads:**

- Threads minimize the context switching time.
- Use of threads provides concurrency (parallelism) within a process.

## Multithreading

**Multithreading** is the ability of a process to create multiple threads that run concurrently.

**_Web Browser:_** Each tab can be a separate thread.
**_MS Word:_** MS Word uses multiple threads, one thread to format the text, other thread to keyboard inputs.

**Types of Multithreading**

### 1. Many-to-Many Model: 

Many user-level threads are mapped to many kernel threads.

#### **Advantages:**  

**Concurrency:** Multiple kernel threads can run on multiple processors, providing true concurrency.  

**Flexibility:** The number of kernel threads can be optimized based on the workload, providing a balance between overhead and performance.  

**Efficient Blocking:** If one user-level thread blocks, other threads can continue to execute because they can be mapped to other kernel threads.

#### **Disadvantages:**  

**Complexity:** The model is more complex to implement and manage. The runtime system must effectively map and schedule threads between user space and kernel space.

**Example:** **Solaris Threads:** Solaris implemented a many-to-many threading model, allowing flexible mapping of user threads to kernel threads.


### 2. Many-to-One Model: 

Many user-level threads are mapped to a single kernel thread.

Common in systems where user-space libraries manage threads without kernel support.

#### **Advantages**: 

**Low Overhead:** Since the kernel only manages one thread per process, there is less overhead compared to the one-to-one model.  

**Fast Context Switches:** Switching between user threads can be faster since it is done in user space without involving the kernel.

#### **Disadvantages:**  

**No True Concurrency:** Because all user threads are mapped to a single kernel thread, only one thread can be executing at a time on a multi-core processor.  

**Blocking Calls:** If a user-level thread makes a blocking system call, the entire process (including all threads) can be blocked.

**Example:** **Green Threads**: Early Java implementations used green threads, where all Java threads were mapped to a single native thread.

### 3. One-to-One Model: 

Each user-level thread is mapped to a single kernel thread.
Used by modern operating systems like Linux and Windows.
#### **Advantages:**  

**True Concurrency:** Since each user thread is a separate kernel thread, multiple threads can run truly concurrently on multiple processors.  

**Kernel-Supported:** Since the kernel is aware of each thread, it can manage thread scheduling, blocking, etc more effectively.  

**Preemptive Scheduling:** The kernel can preempt threads, improving responsiveness.

#### **Disadvantages:**  

**Overhead:** Creating and managing kernel threads is more resource-intensive (in terms of memory and processing time) compared to managing user-level threads.  

**Scalability:** Because each user thread has a corresponding kernel thread, the overhead can be significant if there are a large number of threads.

**Example:**  
**Pthreads in Linux**: Each pthread (POSIX thread) in a Linux application is usually mapped to a unique kernel thread.

![[Pasted image 20260620110314.png]]


**Benefits of Multithreading**

- Improved responsiveness
- Efficient resource utilization
- Faster execution through parallelism

**Issues in Multithreading**

- Race Conditions: Multiple threads accessing shared data simultaneously → unpredictable results.
- Deadlocks
- Starvation: A thread never gets CPU time due to scheduling priorities.

**_Solutions_**

- Synchronization primitives: Mutexes, semaphores, condition variables.
- Careful design to avoid circular waits.
### Components of a Thread

Each thread maintains its own:

- **Program Counter (PC):** Keeps track of the next instruction to execute.
    
- **Stack:** Stores local variables, function parameters, and return addresses.
    
- **Register Set:** Holds the current working state of the processor for that thread.
    
- **Thread ID:** A unique identifier within the process.

There is a parent child relation between Threads. If a thread x is created from a thread y, then y is the *parent* of x. Naturally, main thread is the mother of all threads.

![[Pasted image 20260430143254.png]]

![[Pasted image 20260620110124.png]]

**Context Switching: Process vs Thread**

When the CPU switches from one process to another, it must:

- Save/restore registers and program counter
- Change memory mapping (page tables)
- Flush caches/TLB

Switching between threads of the same process is lighter because memory mapping remains the same, only registers and stack pointers need to be saved/restored.


![[Pasted image 20260430143514.png]]



![[Pasted image 20260430143646.png]]

In a multi-threaded environment, threads within the same process operate within a shared context while maintaining individual execution states.

Threads share the address space of a process, so they work with the same data and code!

List of what threads actually share:

- **Address Space:** All threads see the same virtual memory. A pointer to a memory address in one thread points to the same data in another.
    
- **Data Section (Global Variables):** Static and global variables are accessible and modifiable by all threads.
    
- **Code Section:** The actual compiled instructions of the program are shared; multiple threads can execute the same function simultaneously.
    
- **Heap Memory:** Dynamically allocated memory (via `malloc` or `new`) is shared. Any thread can access a block of memory on the heap if it has the address.
    
- **Operating System Resources:** This includes open file descriptors, sockets, signals, and child processes. If one thread opens a file, any other thread in that process can read from or write to it.


**On a multi-core processor, threads can achieve true parallelism, where different threads 
of the same process execute on different physical cores at the exact same time.


![[Pasted image 20260430143901.png]]
![[Pasted image 20260430143921.png]]

### User Threads vs Daemon Threads

If there are no user threads running, the program will terminate. Otherwise, there may be daemon threads running.

![[Pasted image 20260430144335.png]]










(Java part)
## Let's create threads

![[Pasted image 20260430145612.png]]


![[Pasted image 20260430150836.png]]
(A small example that shows the main thread/method can finish before its child thread)

if we mark the thread as daemon by doing `th.setDaemon(true)` then it may stop after main is done. Daemon threads live to serve user threads

![[Pasted image 20260430152604.png]]

The Thread class is not abstract, nor an interface. So why not use that only?

Thread implements Runnable interface. So it has a run() method which needs to be implemented as you can see. What is target? see below.

So we create a Thread by either
1. Extending Thread itself after which we rewrite the run() method
2. Implementing Runnable (which is what Thread does anyway) and writing the run() method. Then, we pass our created class object to the Thread constructor, this is **target**. **target** is a reference to a Runnable object. So, if we set an object to target, target is not null and hence it runs the run() method of out class object.

Which method is better?- If you extend Thread, then you **cannot extend any other classes!**
Multiple inheritance doesn't exist in Java. But you can implement multiple classes, right? So, there's no problem there. 

![[Pasted image 20260430154123.png]]
When you implement Runnable

![[Pasted image 20260430154203.png]]
When you extend Thread


## Synchronization


What happens when 2 threads simultaneously read a value?
![[Pasted image 20260430154451.png]]
![[Pasted image 20260430154611.png]]

An example Stack 
![[Pasted image 20260430155250.png]]

2 Threads have access to one stack object
![[Pasted image 20260430155300.png]]

Example situation: 
thread1 runs, tries to push 100 into empty stack
stackTop++, so it becomes 0, then it goes to sleep (so its taken out of context)

Now thread2 comes, does stackTop-- (=-1 again!) and then returns element at index 0.
thread1 again runs and sets array[-1]=element, and then all hell breaks loose.

	Fix for this?

prevent more than 1 thread from having access to some object at a particular time!

A Thread can acquire a 'lock', when it has the lock to the room (=object/function) ONLY it can do what it wants. Other threads have to wait.

![[Pasted image 20260430160514.png]]

You can see an object being used as a lock. You can basically use any object as a lock!
When a thread has access to a lock, only this thread can make any changes.

You can see push() and pop() are using the same lock object. Hence, these methods are bounded by this lock object. When a thread gains access to this lock object, other Threads CANNOT access any function which is using the SAME lock!!

However if the locks were different, then other threads could work with them parallely!

![[Pasted image 20260430161201.png]]

You can also use the `synchronized` keyword if you want to make the whole function synchronized. 
But then where's the lock? 
![[Pasted image 20260430161317.png]]

The lock used is the instance of the object itself! This also means that ALL `synchronized` methods are usable by only 1 Thread.

Synchronisation for static methods?

![[Pasted image 20260430161646.png]]

It uses ClassName.class (dk what is that)

![[Pasted image 20260430161802.png]]
![[Pasted image 20260430161847.png]]
![[Pasted image 20260430161918.png]]


![[Pasted image 20260430180124.png]]


![[Pasted image 20260430180158.png]]


![[Pasted image 20260430180241.png]]


![[Pasted image 20260430180310.png]]

![[Pasted image 20260430180336.png]]


## Volatile

![[Pasted image 20260430180911.png]]

![[Pasted image 20260430180949.png]]

The threads read from a cache (Each thread runs on a core, and each core has its own cache)

![[Pasted image 20260430181603.png]]

Th2 makes an update to Flag. This is Not reflected in Thread 1's cache.

![[Pasted image 20260430181700.png]]

Even after Flag is changed in RAM, Th1 still doesn't have the updated value of Flag

![[Pasted image 20260430181808.png]]

With the volatile keyword, Threads read the variable from RAM directly!

## Thread States

![[Pasted image 20260430184932.png]]

![[Pasted image 20260430184953.png]]

![[Pasted image 20260430185156.png]]
yield() tells JVM to put the current running thread back into ready to run state. But there's no guarantee that JVM would actually listen

![[Pasted image 20260430185323.png]]
![[Pasted image 20260430185442.png]]

Difference between sleep and wait: When a thread sleeps, it will NOT relinquish any lock but when it waits it WILL relinquish the lock

**Also, wait only relinquishes the lock on the object it was called from.

![[Pasted image 20260430185829.png]]

![[Pasted image 20260430185925.png]]



![[Pasted image 20260430190120.png]]
![[Pasted image 20260430190143.png]]


Concept of thread join()- When called on a parent thread, it pauses the parent thread until its child threads finish
![[Pasted image 20260430190325.png]]

![[Pasted image 20260430190534.png]]


## Priorities

![[Pasted image 20260430190624.png]]

![[Pasted image 20260430190637.png]]
![[Pasted image 20260430190729.png]]

## Thread Scheduling

![[Pasted image 20260430190906.png]]

## Deadlocks

![[Pasted image 20260430192000.png]]


