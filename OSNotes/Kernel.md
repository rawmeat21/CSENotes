### What is bootloader?

A boot loader is a type of program that loads and starts the boot time tasks and processes of an operating system or the computer system. It enables loading the operating system within the computer memory when a computer is started or booted up. A boot loader is also known as a boot manager or bootstrap loader.

![[Pasted image 20260616210410.png]]

### What is a kernel?

The kernel is at the core of an operating system, loaded into memory during the boot process. 

A kernel exists in a privileged mode and communicates directly with hardware components such as CPU, memory, and peripheral devices. From the level of providing basic services, the kernel allows applications to function without being concerned with the intricacies of hardware.

Kernel is the first part of the OS loaded into memory during boot, and it stays resident while the system is running.

![[Pasted image 20260616210427.png]]

![[Pasted image 20260616210454.png]]

### Whats a system call?

_A_ **_system call_** _is a way for programs to interact with the operating system. A computer program makes a system call when it makes a request to the operating system’s kernel. System call provides the services of the operating system to the user programs via Application Program Interface(API)._

### What is kernel space?

In Linux system memory can be divided into two distinct regions: _kernel space_ and user space. Kernel space is where the kernel _executes_ and provides its _services_.”

### Functions of kernel?

- **Process Management :** Scheduling and execution of processes.
- **Memory Management :** Allocation and deallocation of memory space, managing virtual memory, handling memory protection and sharing.
- **Device Management :** Managing input/output devices, providing a unified interface for hardware devices and handling device driver communication.
- **File System Management :** Managing file operations and providing a file system interface to applications.
- **Resource Management :** Managing system resources (CPU time, disk space, network bandwidth). It allocating and deallocating resources as needed.
- **Security and Access Control :** Enforcing access control policies like user permissions and authentication.
- **Inter-Process Communication :** Facilitating communication between processes by providing mechanisms like message passing and shared memory.


### Types of kernel?

#### Monolithic Kernel (Linux)

This is a gigantic, all-inclusive program that manages everything - such as process handling, memory management, file systems, and device drivers-in a single address space of the kernel. 

Effective communication can be established between components within this category since every component has direct access to kernel space, resulting in high performance due to minimum overhead.

**Characteristics:** Run all kernel services in the kernel space, that is, memory sharing. Includes all the drivers and services necessary. Bulky yet efficient.

**• Advantages:**

· Speeds up execution as direct function calls are used.
· Simplifies development as all components are tightly integrated.
· High performance for resource-intensive tasks.

**• Disadvantages:**

· It becomes risky to bugs and vulnerabilities as more individuals increased their sizes.
· A single error can cause the collapse of the total system due to lack of isolation. Its complexity makes maintenance or modification very difficult.

#### Microkernel

It will run only the most basic services — like IPC, some scheduling, and memory management — in kernel mode. 

Other services, like the file system or a device driver, will run as separate modules in user mode.

**Characteristics:** The microkernel is small and modular wherein services communicate via message passing. Such separation improves reliability and security.

**Advantages:**

· It preserves stability, in that one module can fail without crashing the entire system.
· It preserves modularity, which facilitates maintenance and upgrades.
· It enhances security because isolated components can be investigated for threats.

**Disadvantages:**

· Performance is somewhat slowed due to the overhead of **passing messages**.

**Examples:** Systems that use microkernel architecture include QNX, MINIX

### Hybrid Kernel

Combination of the monolithic kernel and the microkernel architecture.

Core services such as process and memory management run fully in kernel space while some services, like drivers, may run either in user space or kernel space depending on how they are implemented. 

The hybrids have typical characteristics: very fast processing speed as it is not as modular as micro-kernel, but still has some modularity.

**Advantages :** performance-wise better than micro-kernels since there will be less communication overhead, comparatively more flexible to be used in a modular style compared to monolithic kernels, and a lot will cover, from desktop to servers.

**Disdvantages:** greater complexity compared to monolithic ones increases tasks of development, and potentially extends concerns of stability in comparison to microkernels.

**Examples:** Microsoft Windows (NT kernel) and macOS (XNU kernel) are hybrid kernels.

### Nanokernel

Extremely minimal kernel which delegates almost most of the functionality, including basic services, to higher layers.

It provides nothing more than interrupt management and the minimal hardware management.

**Characteristics:** Nanokernels are light in weight. Very few services are implemented as part of the kernel in terms of actual operating system services.

**Advantages:**

· tiny footprint, perfect for resource-constrained environments.
· High flexibility, because developers can customize higher-level services.

**Disadvantages:**

· It describes limited functionality and requires extensive external development.
· It’s possible that the full system requires complex implementation.

**Examples:** L4 and seL4.

They are significant for specialized applications such as real-time systems or secure environments due to extreme minimalism.


### Exokernel

Experimental kernel design that provides minimal abstractions; 

It allows applications to have direct access to hardware resources. Resource protection is separated from its management, so that applications may customize the handling of the resources.

**Characteristics:** Exokernels allocate resources such as CPU time and memory directly to applications, with the kernel ensuring security and isolation.

**Advantages:**

· Flexibility is maximized and allows extreme application optimization.
· Enhanced system performance with reduced kernel overhead.

**Disadvantages:**

· Complexity of application development realized through direct access to hardware.
· It’s possible of limitation since it is experimental.

**Examples:** The Exokernel of MIT and Aegis are investigation projects of this model.

Since exokernels are mainly theoretical and focus on niche systems with extreme performance optimizations, they concentrate more on academic research.