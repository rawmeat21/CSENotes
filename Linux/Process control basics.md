### Components of a process

```
 ┌─────────────────────────────┐
 │        USER SPACE            │   <- what the program "sees"
 │  ┌───────────────────────┐   │
 │  │   stack (grows down)  │   │
 │  │          ↓             │   │
 │  │                        │   │
 │  │          ↑             │   │
 │  │   heap (grows up)      │   │
 │  ├───────────────────────┤   │
 │  │   BSS  (uninit. data)  │   │
 │  │   Data (init. data)    │   │
 │  │   Text (code, r/o)     │   │
 │  └───────────────────────┘   │
 └─────────────────────────────┘
              ▲
              │  syscalls / traps
              ▼
 ┌─────────────────────────────┐
 │        KERNEL SPACE          │
 │  task_struct for this PID:   │
 │   - PID, PPID                │
 │   - UID/EUID/GID/EGID        │
 │   - open file descriptor tbl │
 │   - signal mask               │
 │   - scheduling info (nice)   │
 │   - memory page tables       │
 └─────────────────────────────┘
```

A process consists of an address space and a set of data structures within the kernel. 

The address space is a set of memory pages that the kernel has marked for the process’s use. These pages contain the code and libraries that the process is executing, the process’s variables, its stacks, and various extra information needed by the kernel while the process is running. 

The process’s virtual address space is laid out randomly in physical memory and tracked by the kernel’s page tables.

![Pasted image 20260911222125](../assets/Pasted%20image%2020260911222125.png)

On Linux it's literally called `task_struct`, and you can eyeball a live one indirectly via `/proc/<pid>`.

**Thread**- A “thread” is an execution context within a process. Every process has at least 1 thread, but some processes have many. Each thread has its **own stack** and CPU context but operates within the address space of its enclosing process.

### Some parameters of processes

**PID: process ID number**

The kernel assigns a unique ID number to every process. Most commands and system calls that manipulate processes require you to specify a PID to identify the target of the operation. PIDs are assigned in order as processes are created.

**PPID: parent PID**

Neither UNIX nor Linux has a system call that **initiates a new process running a particular program**. 

Instead, it’s done in two separate steps:

1. First, an existing process must clone itself to create a new process. 
2. The clone can then exchange the program it’s running for a different one.

When a process is cloned, the original process is referred to as the parent, and the
copy is called the child. The PPID attribute of a process is the PID of the parent
from which it was cloned.

**IMP:** The parent PID is a useful piece of information when you’re confronted with an unrecognized (and possibly misbehaving) process. Tracing the process back to its origin (whether that is a shell or some other program) may give you a better idea of its purpose and significance.

```
systemd (PID 1)
 ├─ systemd-journald
 ├─ systemd-logind
 ├─ sshd
 │   └─ sshd (per-connection)
 │       └─ bash
 │           └─ vim
 └─ getty (tty1)
     └─ bash
```

**Why `fork`+`exec` and not one syscall?** Because it's flexible. Between the fork and the exec, the child can do setup work — redirect its stdin/stdout, drop privileges, change its working directory, close file descriptors — _before_ the new program even starts running. This is exactly what your shell does every time you run a command with `command > out.txt`: fork, redirect stdout in the child, then exec.


**UID and EUID: real and effective user ID**

A process’s UID is the user identification number of the person who created it, or more accurately, it is a copy of the UID value of the parent process.

The EUID is the “effective” user ID, an extra UID that determines what resources and files a process has permission to access at any given moment. For most processes, the UID and EUID are the same, the usual exception being programs that are setuid.

UID MEANS IDENTITY
EUID MEANS PERMISSION

- **UID** = who _owns_ the process (inherited from the parent).
- **EUID** = what permissions the process is currently _using_.

Most systems also keep track of a “saved UID,” which is a copy of the process’s EUID at the point at which the process first begins to execute. Unless the process takes steps to obliterate this saved UID, it remains available for use as the real or effective UID. A conservatively written setuid program can therefore renounce its special privileges for the majority of its execution and access them only at the points where extra privileges are needed.

**Linux also defines a nonstandard FSUID process parameter that controls the determination of filesystem permissions.**


**GID and EGID: real and effective group ID**

GID is the group identification number of a process.

EGID is related to the GID in the same way that the EUID is related to the UID in that it can be “upgraded” by the execution of a setgid program. As with the saved UID, the kernel maintains a saved GID for each process.

![Pasted image 20260911223826](../assets/Pasted%20image%2020260911223826.png)

The only time at which the GID is actually significant is when a process creates new files. Depending on how the filesystem permissions have been set, new files might default to adopting the GID of the creating process.


**Niceness**

Priority - how much CPU time a process recieves.

More niceness means lower priority.

In Linux the range is -20 to +19, and in FreeBSD it’s -20 to +20.

Unless the user takes special action, a newly created process inherits the niceness of its parent process.

The owner of the process can increase its niceness but cannot lower it, even to return the process to the default niceness. This restriction prevents processes running at low priority from bearing high-priority children. **However, the superuser can set nice values arbitrarily.**

Use `nice` and `renice` to change priority.

```bash
$ nice -n 5 ~/bin/longtask // Lowers priority (raise nice) by 5
$ sudo renice -5 8829 		 // Sets niceness to -5
$ sudo renice 5 -u boggs		 // Sets niceness of boggs’s procs to 5
```


**Control terminal**

Most nondaemon processes have an associated control terminal. 

The control terminal determines the default linkages for the standard input, standard output, and standard error channels. It also distributes signals to processes in response to keyboard events such as `<Control-C>`

Check yours:
bash

```bash
$ tty
/dev/pts/3        # a pseudo-terminal, e.g. inside a terminal emulator
```

### Life cycle

![Pasted image 20260911224737](../assets/Pasted%20image%2020260911224737.png)
![Pasted image 20260911224746](../assets/Pasted%20image%2020260911224746.png)

**A zombie is not "using resources" in the sense of CPU or RAM** — it's a corpse waiting for its death certificate to be signed. You cannot `kill -9` a zombie away — it's already dead; there's nothing left to kill. The fix is always to find and deal with the _parent_.

### Signals

**Signals are process-level interrupt requests**

• They can be sent among processes as a means of communication.
• They can be sent by the terminal driver to kill, interrupt, or suspend processes when keys such as `<Control-C>` and `<Control-Z>` are pressed.
• They can be sent by an administrator (with kill) to achieve various ends.
• They can be sent by the kernel when a process commits an infraction such as division by zero.
• They can be sent by the kernel to notify a process of an “interesting” condition such as the death of a child process or the availability of data of an I/O channel.

What happens when a process recieves a signal?

-> If the receiving process has designated a handler routine for that particular signal, the handler is called with information about the context in which the signal was delivered. This is called **catching the signal**. When the handler completes, execution restarts from the point at which the signal was received.

-> Else, the kernel takes some default action on behalf of the process. The default action varies from signal to signal. Many signals terminate the process; some also generate core
dumps (if core dumps have not been disabled).


**To prevent signals from arriving, programs can request that they be either ignored or blocked.**

-> A signal that is ignored is simply discarded and has no effect on the process

-> A blocked signal is queued for delivery, but the kernel doesn’t require the process to act on it until the signal has been explicitly unblocked. The handler for a newly unblocked signal is called only once, even if the signal was received several times while reception was blocked.

![Pasted image 20260911225345](../assets/Pasted%20image%2020260911225345.png)

(The 1st column is signal number)

**The signals named KILL and STOP cannot be caught, blocked, or ignored.**

-> KILL signal destroys the receiving process
-> STOP suspends its execution until a CONT signal is received. CONT can be caught or ignored, but not blocked.

TSTP is a “soft” version of STOP that might be best described as a request to stop. It’s the signal generated by the terminal driver when `<Control-Z>` is typed on the keyboard. Programs that catch this signal usually clean up their state, then send themselves a STOP signal to complete the stop operation. Alternatively, programs can ignore TSTP to prevent themselves from being stopped from the keyboard.

![Pasted image 20260911225947](../assets/Pasted%20image%2020260911225947.png)


|Signal|#|What it really means in practice|
|---|---|---|
|`SIGHUP`|1|"reload config" convention for daemons, OR literally "your terminal hung up"|
|`SIGINT`|2|what `<Ctrl-C>` sends — "please stop, politely"|
|`SIGQUIT`|3|`<Ctrl-\>` — like INT but expects a core dump|
|`SIGKILL`|9|unblockable, uncatchable, instant death — last resort|
|`SIGTERM`|15|the _default_ kill signal — "please exit cleanly"|
|`SIGSTOP`|19|unblockable pause (like `<Ctrl-Z>` but uncatchable)|
|`SIGTSTP`|20|what `<Ctrl-Z>` actually sends — a _catchable_ "please stop"|
|`SIGCONT`|18|resume after STOP/TSTP|

**The SIGTERM vs SIGKILL distinction is the single most important practical thing in this section.** `kill pid` sends SIGTERM by default — a _request_. A well-behaved program catches it, flushes its buffers, closes its files, deletes its lockfile, and exits. `kill -9 pid` sends SIGKILL, which the kernel enforces at the scheduler level — the process gets **zero chance to clean up**. This is why you always try plain `kill` first and only escalate to `-9` if the process is genuinely wedged.

#### How to kill processes??

Use `kill` command. You can only use this on your created processes or if you are root.

```bash
$ kill [-signal_number] pid
```

By default, a `TERM` signal is sent. But just sending a `TERM` doesn't mean a process will die.

```
$ kill -9 pid
```

sends a `KILL` signal. 

**Processes can on occasion become so wedged that even `KILL` does not affect them, usually because of some degenerate I/O vapor lock such as waiting for a volume that has disappeared. Rebooting is usually the only way to get rid of these processes.**

![Pasted image 20260911230540](../assets/Pasted%20image%2020260911230540.png)


### Sleeping processes and threads

When runnable, threads must often wait for the kernel to complete some background work for them before they can continue execution. 

For example, when a thread reads data from a file, the kernel must request the appropriate disk blocks and then arrange for their contents to be delivered into the requesting process’s address space. During this time, the requesting thread enters a short-term sleep state in which it is ineligible to execute. 

**A process is generally reported as “sleeping” when all its threads are asleep**.

Interactive shells and system daemons spend most of their time sleeping, waiting for terminal input or network connections. 

#### Uniterruptible sleep

Some operations can cause processes or threads to enter an uninterruptible sleep state. This state is usually transient and is not observed in `ps` output (denoted by a
D in the STAT column). 

However, a few degenerate situations can cause it to persist. The most common cause involves server problems on an NFS filesystem mounted with the hard option. 

Since processes in the uninterruptible sleep state cannot be roused even to service a signal, they cannot be killed.

To get rid of them, you must fix the underlying problem or reboot.

#### Zombie process?

If you see zombies hanging around, check their PPIDs with `ps` to find out where they’re coming from.











