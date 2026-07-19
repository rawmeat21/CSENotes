eBPF (Extended Berkeley Packet Filter) is a Linux technology that allows developers to run sandboxed, custom programs directly inside the operating system kernel. 

It is used to safely and efficiently extend the capabilities of the kernel at [runtime](https://en.wikipedia.org/wiki/Runtime_\(program_lifecycle_phase\) "Runtime (program lifecycle phase)") without requiring changes to kernel [source code](https://en.wikipedia.org/wiki/Source_code "Source code") or loading [kernel modules](https://en.wikipedia.org/wiki/Loadable_kernel_module "Loadable kernel module"). 

It allows you to write small programs that run in the Linux kernel. These programs can be used to monitor, control, and modify the behavior of the kernel.

**eBPF programs are written in a simple C-like language, and they are compiled into bytecode that is then loaded into the kernel.

Some of the things that eBPF programs can be used for include:

- **Monitoring kernel events.** eBPF programs can be used to monitor kernel events, such as network packets or system calls. This can be used to collect data for troubleshooting, debugging, or security analysis.  
- **Controlling kernel behavior.** eBPF programs can be used to control kernel behavior. This can be used to optimize performance, improve security, or implement new features.  
- **Modifying kernel data.** eBPF programs can be used to modify kernel data. This can be used to change the behavior of the kernel, or to collect data that is not otherwise accessible.

Traditionally there were only two ways to get kernel-level behavior: 

(1) write a real kernel module, load it with `insmod` — powerful but dangerous (a bug can crash or corrupt the whole machine) and fragile (breaks across kernel versions);

(2) don't run in the kernel at all, and instead rely on things like `ptrace()` (slow) or reading `/proc` (doesn't give you syscall-level events at all).

eBPF is a third option: you write a small program, the kernel **verifies it can't crash or hang the system** (a "verifier" statically analyzes it — no unbounded loops, no invalid memory access, no crashing the kernel), then it gets **JIT-compiled** (Just in time) and run directly inside kernel space, attached to a specific hook point (a syscall entry, a network packet arriving, a function being called, etc.) — all without needing a kernel module, without needing to recompile the kernel, and safely, because the verifier's rules make it structurally impossible for the program to do most of the dangerous things a kernel module could do.

#### A little more about eBPF

eBPF programs can't just call arbitrary kernel functions or use normal data structures the way kernel module code can. Instead:

- They can only call a fixed, kernel-exposed set of **BPF helper functions** (`bpf_get_current_pid_tgid()`, `bpf_ktime_get_ns()`, `bpf_ringbuf_reserve()`, etc.) — these are the sanctioned APIs for talking to the kernel from inside an eBPF program.

- They communicate with the rest of the world (other eBPF programs, or userspace) through special kernel objects called **maps** — persistent, kernel-managed data structures (arrays, hash maps, ring buffers) that both the eBPF program _and_ regular userspace C++ code can read/write. This is the _only_ way data crosses the kernel/userspace boundary here.

What's a ring buffer? It's basically a circular queue. We can push to it or pop. It's needed because: The eBPF program (running in kernel context, on possibly any CPU, whenever a syscall fires) is the **producer**, it only ever writes at `head` and advances it. The Tracer object is the **consumer**, it only ever reads at `tail` and advances it. This is the **single-producer/single-consumer (SPSC)** pattern from concurrent data structure design.

```c
// will be sent through the ring buffer by the eBPF program everytime a syscall is made
struct event {
    __uint32_t pid;// pid
    __uint32_t syscall_nr;// syscall number
    __uint64_t timestamp;// when was syscall made 
};

```

```cpp
// This is what is Tracer object reads. It will set the blocked field.
struct event {
    __uint32_t pid;// pid
    __uint32_t syscall_nr;// syscall number
    __uint64_t timestamp;// when was syscall made
    __uint8_t blocked;    
};
```

#### The `handle_syscall()` function

```c
SEC("tracepoint/raw_syscalls/sys_enter")
```

What the fuck is this? Let's talk about tracepoints. Refer this: https://blog.devgenius.io/instrumenting-linux-kernel-with-tracepoints-524a7849a9a3

[**_Instrumentation_**](https://en.wikipedia.org/wiki/Instrumentation_\(computer_programming\)) is a mechanism by which you can measure the runtime behaviour of a system. Programs require instrumentation to observe it’s internal working state, it’s analogous to how someone uses a thermometer to measure (instrument) body temperature and take correct health measures based on its readings. 

#### Static Instrumentation : Linux Kernel Tracepoints

[**Tracepoints**](https://github.com/torvalds/linux/blob/master/Documentation/trace/tracepoints.rst#using-the-linux-kernel-%20tracepoints) are function calls that kernel developers have inserted into the kernel code at logical places, these are directly compiled into the kernel binary.

Better definition:

Tracepoints are predefined hook points in the Linux kernel, and eBPF programs can be attached to these tracepoints to execute custom logic whenever the kernel reaches those points.

our eBPF program attaches to `tracepoint/raw_syscalls/sys_enter`. This tracepoint is a global hook designed to **intercept the initialization phase of every single system call executed by any thread across the entire system.**

**`tracepoint/raw_syscalls/sys_enter`** is a built-in Linux kernel tracepoint that fires every time **any system call is initiated** by a user-space application. 

### Why I used eBPF in fragarach

The eBPF program has 1 job: whenever a syscall is made in the system, it intercepts it, checks if the PID matches my target's PID, if yes, it adds it to the ring buffer. The Tracer will read this 'event' struct.

