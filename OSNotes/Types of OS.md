## 1. Batch operating System

-> A [Batch Operating System](https://www.geeksforgeeks.org/operating-systems/batch-processing-operating-system/) is designed to handle large groups of similar jobs efficiently. 

-> It does not interact with the users directly but instead processes jobs that are grouped by an operator.

-> These jobs are queued and executed one after the other, without user interaction during the process.

-> Executes jobs in groups (batches) without direct user interaction. Users prepare their jobs offline (such as using punch cards) and submit them to an operator. The system groups jobs with similar requirements and processes them together to improve efficiency and reduce setup time.

-> Good for large amount of data. It is widely used in industries like **banking, payroll, and data processing** where large datasets need to be handled regularly.

-> Ex- IBM's z/OS, Unisys MCP

-> Usecases: - Insurance Claim Processing, Library Book Records, Stock Market Reports



![[Pasted image 20260620100327.png]]

#### **Advantages**

**Handling Repetitive Tasks**: Ideal for managing large, repetitive tasks

**Minimal Idle Time**: The system minimizes idle time by processing jobs in a continuous sequence without human intervention.

**Resource Efficiency:** These systems improve the use of computation resources by processing jobs in groups and scheduling them during stages of resource accessibility.

#### Disadvantages

**Limited functionality:** Can solve only simple tasks. Difficult to use for certain tasks, like managing files or software.

**Interruptions** Batch systems can be interrupted frequently, which can lead to missed deadlines or mistakes.

**Security issues:** Batch operating systems are not very secure because they are not typically used for day-to-day tasks, so they are not as secure as OSes.

See more:  https://www.geeksforgeeks.org/operating-systems/batch-processing-operating-system/

## 2. Multi-Programming Operating System

In a [Multi-Programming Operating System](https://www.geeksforgeeks.org/operating-systems/multiprogramming-in-operating-system/), multiple programs run in memory at the same time. The CPU switches between programs, giving an illusion that multiple jobs are running at the same time on the system.

### Features

- Context switch between process.
- Switching happens when current process undergoes waiting state.

### **Advantages**

- **Better CPU Utilization:** CPU stays busy by switching to another job during I/O wait.
- **Improved Throughput:** Multiple jobs run concurrently, so more work gets done in less time.
- **Efficient Resource Use:** CPU, memory, and I/O devices are shared efficiently among processes.

### **Disadvantages**

- **Complex Design:** Requires advanced memory management and CPU scheduling.
- **Security Issues:** More programs in memory increase chances of unauthorized access.
- **High Memory Requirement:** Needs larger RAM to run multiple programs together.

### **How do Multiprogramming Operating Systems Work?**

- The OS selects a ready process. 
- When the chosen process undergoes CPU execution, it might be possible that in between execution, it needs some IO.
- At that time process enters into a wait state, waiting for the event.
- Then, CPU switches to next ready process.
- When the process which undergoes for I/O operation comes again after completing the work, it again becomes ready for execution

There can be preemptive or non-preemptive OS.

Example- **IBM OS/360 (MVT / MFT) (non-preemptive)**, **Linux**, **Windows NT**, **macOS** (all 3 are preemptive)


## 3. Multi-tasking/Time-sharing Operating systems

Logical extension of multiprogramming OS.

It uses **time-slicing** for executing processes. Instead of waiting for a process to voluntarily yield control via an I/O request, the OS assigns each process a tiny window of CPU time, called a **time quantum**.

### Types

#### **1. Preemptive**

- In preemptive multitasking, the [operating system](https://www.geeksforgeeks.org/operating-systems/what-is-an-operating-system/) can interrupt a running process and allocate the CPU to another process.
- Preemption ensures that no single process monopolizes the CPU, improving system responsiveness.
- Examples: Windows 95, WindowsNT, Linux and UNIX-based OS.

**Advantages:**

- Efficient CPU utilization.
- Better system responsiveness and user experience.
- Allows higher-priority processes to take precedence over lower-priority ones.

**Disadvantages:**

- [Context switching](https://www.geeksforgeeks.org/operating-systems/context-switch-in-operating-system/) overhead can reduce performance.
- Complexity in process synchronization and management.
- Can lead to [starvation](https://www.geeksforgeeks.org/operating-systems/starvation-and-aging-in-operating-systems/) if lower-priority processes are never allocated CPU time.

#### 2. **Cooperative/Non-Preemptive Multitasking Operating System**

- OS does not initiate context switching from one process to another.
- A context switch occurs only when processes voluntarily yield control or are blocked.
- Processes cooperate to allow multiple applications to run simultaneously, ensuring the system operates smoothly.
- Examples: older versions of Macintosh OS (8.0-9.2.2) and Windows 3.x.

**Advantages:**

- No overhead of context switching.
- Processes have more control over their execution.

**Disadvantages:**

- Less efficient CPU utilization.
- Risk of system unresponsiveness if a process fails to yield control.


## 4. Multi-Processing Operating System

-> More than one CPU is used for the execution of processes. 

-> Goal of a multiprocessing OS is to ****increase execution speed**** and ****reliability**** through ****parallel processing****

### Working of Multi-Processing Operating System

- The system consists of multiple CPUs connected to a shared main memory.
- Tasks are scheduled and distributed among the processors.
- Each processor executes its assigned task in parallel with others.
- After execution, results are combined (if required) to produce the final output.
- The OS manages CPU scheduling, memory access, and resource allocation.

![[Pasted image 20260620103350.png]]

### Types:

### 1. Symmetric Multiprocessing (SMP)

All processors are equal and execute the same instance of the operating system. Any processor can perform any task, including process scheduling and I/O handling.

- Processes are dynamically assigned to processors using CPU scheduling algorithms.
- All processors share the same physical memory and I/O subsystem.
- Also known as a Shared-Memory Multiprocessing System.

![[Pasted image 20260620103734.png]]

#### Advantages

- Failure of one processor does not affect the functioning of other processors.
- Divides workload equally to the the processors.
- Makes use of available resources efficiently.

#### Disadvantages

- More complex.
- Costly
- Synchronization between multiple processors is difficult.

### 2. Asymmetric Multiprocessing (AMP)

Processors are not equal. One processor acts as the master, while the others act as slave processors.

- The master processor handles scheduling, memory management, and I/O operations.
- Slave processors execute tasks assigned by the master.
- The master maintains the ready queue and dispatches processes to slaves.

![[Pasted image 20260620103725.png]]

#### Advantages

- Less cost
- Easy to design.
- More scalable.

#### Disadvantages

- There can be uneven distribution of workload among the processors.
- The processors do not share same memory.
- Entire system goes down if master fails.


## 5. Distributed Operating System

-> Connects multiple independent computers through a shared communication network. 

-> Each system has its own CPU and memory. 

-> The main benefit is remote access, allowing users to use files and software stored on other connected systems.

![[Pasted image 20260620104102.png]]

Read more: https://www.geeksforgeeks.org/operating-systems/what-is-a-distributed-operating-system/


## 6. Network Operating System

-> Runs on a server and manages data, users, security, applications, and other network functions. 

-> It allows shared access to files, printers, and resources within a small private network. 

-> Users can see the configuration and connections of other users, which is why these systems are considered **tightly coupled**.

![[Pasted image 20260620104639.png]]


## 7. Real-Time Operating System

-> Used in environments where a large number of events, mostly external to the computer system, must be accepted and processed in a short time or within certain deadlines, (industrial control, telephone switching equipment, flight control, and real-time simulations.)

-> Designed to respond to tasks quickly and predictably.

-> Ensures tasks are completed within a **fixed time limit** (deadline).

Examples: Airline traffic control systems, Command Control Systems, airline reservation systems, Heart pacemakers, Network Multimedia Systems, robots, etc.


### Types:

- **Hard RTOS :** Guarantee that critical tasks are completed within a range of time. For example - medical imaging systems, industrial control systems, weapon systems, robots, air traffic control systems

- **Soft RTOS :** Provides some relaxation in the time limit. For example - Multimedia systems, digital audio systems.

- **Firm RTOS :** Example: Multimedia applications.


