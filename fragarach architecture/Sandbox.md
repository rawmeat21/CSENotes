The Sandbox is responsible for launching a child process from the main process. This child is launched inside a Linux namespace. cgroups are used to control how much hardware resources this child can use. When the child process is launched, it firsts mounts an overlay file system. An overlayfs has 3 parts `lowerdir`, `upperdir` and `merged`. `merged` is a combined view of the other two. The child first pivots its new root to be the `merged` directory through the `pivot_root()` system call. After that, the child unmounts its previous `root` directory using a `umount2()` call. After this point, the child process is rooted at `merged` and is completely isolated from the main system. Also, any mounting done by the child will not be visible to other processes.

`fork()` duplicates the calling process into a parent and a child, and the child shares almost everything conceptually but gets its own copy of memory, file descriptors table, etc. 

`clone()` does the same fundamental thing, but it lets _you_ control exactly which resources are shared between parent and child, and which are given fresh/isolated instances. `fork()` is essentially `clone()` with a fixed set of "share everything the normal way" flags.

Why not use `fork()`? - Because it would be stupid. A normal `fork()`ed child still lives in the same PID namespace (it can see other processes), the same mount table (it sees the real filesystem), the same network namespace (it can talk to the network). 

`clone()` uses Linux namespaces to give the child process its own view of things instead of sharing the parent's view. 

```
CLONE_NEWPID- PID namespace (PID of child is 1), it cannot see other processes
CLONE_NEWNS- mount namespace (gives child its own mount table, system's mount table is untouched)
CLONE_NEWNET- network namespace (child has no connection to the newtork interfaces)
SIGCHILD- signal to the parent for cleanup (not a namespace)
```
`CLONE_NEWUSER` (user namespace). It would let the child _believe_ it's running as root inside its namespace while actually being an unprivileged user on the host

==Note that namespaces== ==_do not_== ==restrict access to physical resources such as CPU, memory and disk. That access is metered and restricted by a kernel feature called ‘cgroups’.==

[Linux namespaces](https://man7.org/linux/man-pages/man7/namespaces.7.html?ref=hackerstack.org) can be used to provide processes a limited view on some of the system components. This is an important piece when we want to isolate processes from each other.

#### 1. The PID Namespace (Process Isolation)

Every program running on Linux is assigned a **PID (Process ID)**. Traditionally, the very first process that boots up (like `systemd` or `init`) gets **PID 1**, and it acts as the parent of everything else.

A PID Namespace creates a brand-new, isolated process tree.

When you spawn a process into a new PID namespace, it thinks it is the very first thing alive on the machine. The kernel assigns it **PID 1** inside its own little bubble.

However, the host operating system still needs to track it. To the host, that same process might be assigned **PID 4829**.

### Why this matters:

- **The Illusion:** If you run `ps aux` or `top` inside that namespace, you will only see PID 1 (yourself) and any child processes you create. The rest of the host system's running programs are completely invisible to you.

- **Security & Stability:** A process inside a custom PID namespace cannot view, signal, or accidentally kill (`kill -9`) any processes running out on the main host system or in other containers.

#### 2. The Mount Namespace

When a process is running inside its own Mount namespace, it has its own private table of mounts (like what disks are hooked up to `/`, `/home`, or `/var`).


### Cgroups

cgroups (control groups) control _how much of the machine's resources a process (or group of processes) is allowed to consume_ — CPU time, memory, number of PIDs, I/O bandwidth, etc. 

Why do we do this? - A number of reasons:

1. The binary may do a fork-bomb (creating too many children)
2. It may also deliberately use a lot of memory or do cpu intensive tasks

We write to files at `/sys/fs/cgroup/`. 

We write to 3 files: `memory.max`, `pids.max`, `cpu.max`. 

Finally we write the PID of the child to `cgroup.procs`. Writing a PID into `cgroup.procs` is the "join" operation: that process (and anything it forks, since children inherit their parent's cgroup) is now subject to every limit set in that directory.


### What the child process does

First the application has to be started in sudo mode to do system level operations. The child is also launched as sudo.

1. First, it mounts an overlay filesystem using the `mount()` syscall.
2. Then it switches to the `merged` directory by using `chdir()` 
3. Then, it uses `pivot_root(".", ".")` to change its root to current directory `merged()`
4. Next, it unmounts its previous root. The child process is now completely isolated.
5. Next, it drops all capabilites of the child process. Capabilities refers to several priviedged operations. This doesn't make the child completely incapable of anything, capabilities refers only to ~40 priviledged operations.
6. Next, it applies seccomp (secure computing mode) filters. These filters control what system calls a process is able to make. 
7. Finally, it uses `execve()` to execute the target binary by replacing itself with the target. `execve()` **replaces** the current process image entirely with the target binary, with the same PID, same namespaces, same cgroup, same capabilities, same seccomp filter. It now runs the actual untrusted code.


### Linux capabilities

Traditional UNIX implementations distinguish two categories of processes:

- _privileged_ processes (whose effective user ID is 0, referred to as superuser or root), and _unprivileged_ processes (whose effective UID is nonzero).  

- Privileged processes bypass all kernel permission checks, while unprivileged processes are subject to full permission checking based on the process's credentials (usually: effective UID, effective GID, and supplementary group list).

In Linux, capabilities are a way to assign specific privileges to a running process. Some capabilites are:

- **CAP_CHOWN** - Make arbitrary changes to file UIDs and GIDs
- **CAP_DAC_OVERRIDE** - Bypass file read, write, and execute permission checks. (DAC is an abbreviation of "discretionary access control".)
- **CAP_NET_ADMIN** - Perform various network-related operations.
- **CAP_SYS_BOOT** - reboot system.
- **CAP_SYS_CHROOT** - use chroot.

Check out more: https://man7.org/linux/man-pages/man7/capabilities.7.html


### Seccomp

**Secure Computing Mode** (seccomp), a crucial [Linux kernel](https://www.armosec.io/glossary/linux-kernel/) component empowers administrators and developers to restrict the system calls available to a process. It provides a secure, controlled environment for applications, limiting their interaction with the kernel to only authorized system calls. By doing so, seccomp significantly reduces the attack surface of containers and pods.

It acts as a gatekeeper between an app and the kernel. 

Two key security strategies are **default seccomp profiles** and **seccomp filters**. 

**Default Seccomp Profiles:** These profiles are pre-defined sets of allowed system calls for a container or process. By employing default profiles, you can ensure that only essential and safe system calls are permitted, mitigating the risk of malicious exploitation.

**Seccomp Filters:** For more advanced and tailored security configurations, seccomp filters come into play. Seccomp filters allow you to specify precisely which system calls should be allowed or denied for a given container or process. This fine-grained control empowers you to adapt seccomp to your application’s needs, minimizing potential reducing security risks.

**Seccomp Modes:** Seccomp operates in two primary modes: ‘strict’ and ‘filter.’

- **Strict Mode:** A minimal set of essential syscalls is allowed in’ strict’ mode. This mode is highly restrictive and suitable for scenarios requiring the absolute minimum of system calls.
- **Filter Mode:** ‘Filter’ mode allows you to define custom policies specifying which syscalls are permitted and which are denied. This mode offers greater flexibility for tailoring policies to specific use cases.

```cpp
	scmp_filter_ctx ctx = seccomp_init(SCMP_ACT_ALLOW);
	
    seccomp_rule_add(ctx, SCMP_ACT_NOTIFY, SCMP_SYS(init_module), 0);
    seccomp_rule_add(ctx, SCMP_ACT_NOTIFY, SCMP_SYS(kexec_load), 0);
    seccomp_rule_add(ctx, SCMP_ACT_NOTIFY, SCMP_SYS(ptrace), 0);
    seccomp_rule_add(ctx, SCMP_ACT_NOTIFY, SCMP_SYS(reboot), 0);
    seccomp_rule_add(ctx, SCMP_ACT_NOTIFY, SCMP_SYS(setuid), 0);
```

What this does:

`SCMP_ACT_NOTIFY` is a Linux seccomp action that halts a process attempting an intercepted system call and delegates the handling to a user-space monitoring agent, rather than directly denying or allowing the call.

calls like `init_module`, `kexec_load`, `ptrace`, `reboot`, `setuid` are intercepted by seccomp and are basically blocked.





