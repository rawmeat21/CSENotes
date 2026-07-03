
**Concurrency vs Parallelism**

- _Concurrent processing_ means that different tasks are progressing at the same time. This does not mean that they are running in parallel. Concurrency works by scheduling different threads to make the maximum utilization of a single CPU.

- _Parallelism_ means that different threads are being processed at the same time. This can be done by distributing the different threads across different CPUs.


> **_Core_**

A core is an individual processing unit within a CPU. Modern CPUs can have multiple cores, allowing them to perform multiple tasks simultaneously.

A quad-core processor has four cores, allowing it to perform four tasks simultaneously. For instance, one core could handle your web browser, another your music player, another a download manager, and another a background system update.


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

## Thread States

![[Pasted image 20260430184932.png]]

![[Pasted image 20260430184953.png]]

## **Thread Lifecycle**

The lifecycle of a thread in Java consists of several states, which a thread can move through during its execution.

- **New:** A thread is in this state when it is created but not yet started.
- **Runnable:** After the start method is called, the thread becomes runnable. It’s ready to run and is waiting for CPU time.
- **Running:** The thread is in this state when it is executing.
- **Blocked/Waiting:** A thread is in this state when it is waiting for a resource or for another thread to perform an action.
- **Terminated:** A thread is in this state when it has finished executing.


## Deadlocks

![[Pasted image 20260430192000.png]]


