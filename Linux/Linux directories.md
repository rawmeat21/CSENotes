![Pasted image 20260624233123](../assets/Pasted%20image%2020260624233123.png)
![Pasted image 20260624233141](../assets/Pasted%20image%2020260624233141.png)
![Pasted image 20260624233203](../assets/Pasted%20image%2020260624233203.png)
![Pasted image 20260624233214](../assets/Pasted%20image%2020260624233214.png)
![Pasted image 20260624233223](../assets/Pasted%20image%2020260624233223.png)

![](../assets/Pasted%20image%2020260925115759.png)

Let's start. In Linux, everything is a file. Like:

- Processes
- Audio devices
- Kernel data structures and tuning parameters
- Interprocess communication channels

Yep.

What is a filesystem?

The filesystem can be thought of as comprising four main components:

• A namespace – a way to name things and organize them in a hierarchy
• An API – a set of system calls for navigating and manipulating objects
• Security models – schemes for protecting, hiding, and sharing things
• An implementation – software to tie the logical model to the hardware


Some filesystems live on disk partitions or on logical volumes backed by physical disks, but filesystems can be anything that obeys the proper API: a network file server, a kernel component, a memory-based disk emulator, etc.

## `mount`

**Filesystems are attached to the tree with the `mount` command.**

`mount` maps a directory within the existing file tree, **called the mount point**, to the
root of the newly attached filesystem. 

The previous contents of the mount point become temporarily inaccessible as long as another filesystem is mounted there.

Mount points are usually empty directories, however.


```
sudo mount /dev/sda4 /users
```

installs the filesystem stored on the disk partition represented by `/dev/sda4` under
the path `/users`. You could then use `ls /users` to see that filesystem’s contents.

You can run the mount command without any arguments to see all the filesystems
that are currently mounted.

The `/etc/fstab` file contains information about the filesystems mounted on the system. This is a very useful file!

The information in this file allows filesystems to be automatically checked (with `fsck`) and mounted (with `mount`) at boot time, with options you specify.


To detach filesystems, use `umount`.

-> `umount` complains if you try to unmount a filesystem that’s in use. The filesystem to be detached must not have open files or processes whose current directories are located there, and if the filesystem contains executable programs, none of them can be running.

-> There is `umount -f` (force unmount). However, it’s almost always a bad idea to use it on non-NFS mounts, and it may not work on certain types of filesystems (e.g., those that keep journals, such as XFS or ext4). 

Instead of using this, use `fuser`.

`fuser -c mountpoint` prints the PID of every process that’s using a file or directory on that filesystem, plus a series of letter codes that show the nature of the activity. 

For example,

```
freebsd$ fuser -c /usr/home
/usr/home: 15897c 87787c 67124x 11201x 11199x 11198x 972x
```

In this example from a FreeBSD system, 

-> c indicates that a process has its current working directory on the filesystem.

-> x indicates a program being executed. However, the details are usually unimportant, the PIDs are what you want.

A more elaborate alternative to `fuser` is the `lsof` utility.


Under Linux, scripts in search of specific information about processes’ use of filesystems can also read the files in `/proc` directly. However, `lsof -F`, which formats lsof’s output for easy parsing, is an easier and more portable solution. Use additional command-line flags to request just the information you need.


## File types

• Regular files
• Directories
• Character device files
• Block device files
• Local domain sockets
• Named pipes (FIFOs)
• Symbolic links

![](../assets/Pasted%20image%2020260925115542.png)

You can determine the type of an existing file with the `file` command.

```bash
❯ file /dev/stdin 
/dev/stdin: symbolic link to /proc/self/fd/0
```

**Regular files**

Regular files consist of a series of bytes; filesystems impose no structure on their contents. Text files, data files, **executable programs**, and shared libraries are all stored as regular files. Both sequential access and random access are allowed.


**Directories**

A directory contains named references to other files.

**A file’s name is stored within its parent directory, not with the file itself.**


**Hard links**

This is something that's already been discussed under `Symbolic links and Hard links.md`.


**Character and block device files**

Device files let programs communicate with the system’s hardware and peripherals.

-> The kernel includes (or loads) driver software for each of the system’s devices. This software takes care of the messy details of managing each device so that the kernel itself can remain relatively abstract and hardware-independent.

**Device drivers** present a standard communication interface that looks like a regular file. 

When the filesystem is given a request that refers to a character or block device
file, it simply passes the request to the appropriate device driver. 

It’s important to distinguish device files from device drivers, however. The files are just rendezvous points that communicate with drivers. **They are not drivers themselves**.


![](../assets/Pasted%20image%2020260925120456.png)

**Block devices** — data is accessed in fixed-size chunks ("blocks," historically 512 bytes, now often 4096), and — critically — **randomly addressable**. You can jump straight to block #50000 without reading everything before it. The kernel also **buffers/caches** block device I/O aggressively (this is the page cache you'd have seen mentioned if we'd covered memory management yet). Disks, SSDs, USB drives, loopback devices — anything where "seek to an arbitrary position" is a meaningful, efficient operation.

**Character devices** — data is a **stream**. You read it byte-by-byte (or in whatever chunk size makes sense), sequentially, and there's no general notion of "seek to position N" that makes sense for most of them. Keyboards, serial ports, `/dev/null`, `/dev/random`, your terminal — none of these have a concept of "the 500th byte" independent of when you read it.


```
BLOCK device (e.g. /dev/sda, a disk):
   [blk0][blk1][blk2][blk3][blk4][blk5]...
              ▲                    ▲
        you can seek()       or seek() here
        directly here         directly, no need
                               to read blk0-4 first

CHARACTER device (e.g. /dev/ttyS0, a serial port):
   byte -> byte -> byte -> byte -> byte -> ...
   (a stream; "seeking" to a specific byte has no meaning --
    there's no fixed underlying storage to index into)
```

A genuinely intuitive way to think about it: **block = a disk you can randomly poke at; character = a pipe of bytes flowing past you.**


A device file has **no data of its own** sitting on disk the way a regular file does. It's a special entry that says "when someone opens/reads/writes me, forward that operation to _this specific kernel driver_, addressing _this specific unit_."

```
   your program:  read("/dev/sda", buf, 512)
                          │
                          ▼
              kernel sees this is a device file,
              NOT a regular file -- doesn't touch
              a filesystem's data blocks at all
                          │
                          ▼
              looks up major number -> which DRIVER
                          │
                          ▼
              hands the read request to that driver,
              which talks to actual hardware
                          │
                          ▼
                     returns data
```
	
Device files are characterized by two numbers, called the major and minor device numbers.

The major device number tells the kernel which driver the file refers to, and the minor device number typically tells the driver which physical unit to address. 

For example, major device number 4 on a Linux system denotes the serial driver. The first serial port (/dev/tty0) would have major device number 4 and minor device number 0.

*Drivers can interpret the minor device numbers that are passed to them in whatever
way they please. For example, tape drivers use the minor device number to deter-
mine whether the tape should be rewound when the device file is closed.

An example:
```bash
$ ls -l /dev/sda /dev/tty
brw-rw---- 1 root disk 8, 0 Sep 25 10:00 /dev/sda
crw-rw-rw- 1 root tty  5, 0 Sep 25 10:00 /dev/tty
```

The two numbers after the group name (`8, 0` above) are exactly what the book describes: **major** = which driver, **minor** = which unit/instance that driver should address.

```bash
$ ls -l /dev/sd*
brw-rw---- 1 root disk 8,  0 ... /dev/sda
brw-rw---- 1 root disk 8,  1 ... /dev/sda1
brw-rw---- 1 root disk 8,  2 ... /dev/sda2
brw-rw---- 1 root disk 8, 16 ... /dev/sdb
```

Same major number (8) across all of these: they're all handled by the same underlying SCSI-disk driver. The minor number distinguishes _which_ disk and _which partition_: `sda` gets minor 0, its first partition `sda1` gets minor 1, and a second physical disk `sdb` jumps to minor 16 (the SCSI disk driver reserves a block of 16 minor numbers per physical disk, to leave room for up to 15 partitions on each - a real, if slightly dated, design decision baked into the major/minor scheme).


`/proc/devices` lists all device drivers currently registered with the kernel, split into two sections: **Character devices** and **Block devices**. Each line is:

```
<major_number> <name>
```

```
❯ cat /proc/devices
Character devices:
  1 mem
  4 /dev/vc/0
  4 tty
  4 ttyS
  5 /dev/tty
  5 /dev/console
  5 /dev/ptmx
  7 vcs
 10 misc
 13 input

Block devices:
  8 sd
 65 sd
 66 sd
 67 sd
```



Some devices examples:

```bash
$ ls -l /dev/null /dev/zero /dev/random /dev/urandom /dev/tty
crw-rw-rw- 1 root root 1, 3 ... /dev/null
crw-rw-rw- 1 root root 1, 5 ... /dev/zero
crw-rw-rw- 1 root root 1, 8 ... /dev/random
crw-rw-rw- 1 root root 1, 9 ... /dev/urandom
crw-rw-rw- 1 root root 5, 0 ... /dev/tty
```

All major number 1: that's the "memory driver," a virtual driver that implements a handful of small, fake devices entirely in software (no actual hardware behind them). The **minor** number tells that one driver which fake behavior to produce:

- `/dev/null` (minor 3): discards everything written to it, returns EOF instantly on read. This is why `command > /dev/null` is "throw output away."
- `/dev/zero` (minor 5): returns infinite `0x00` bytes on read. Classic use: `dd if=/dev/zero of=file bs=1M count=100` to create a 100MB file of zeros.
- `/dev/random` and `/dev/urandom` (minors 8, 9): return random bytes, sourced from the kernel's entropy pool.

#### `/dev/tty`

`/dev/tty` is the character device that literally _is_ that mechanism at the filesystem level: a process's stdin/stdout being "connected to its control terminal" concretely means those file descriptors are open against a character device file like `/dev/pts/3`. Try:

```bash
$ echo hello > /dev/tty
hello
```

The kernel subsystem that handles "a device that's a two-way character stream representing a human sitting at a terminal, with support for things like line editing, `<Ctrl-C>` handling, echo" is called the **TTY subsystem** today, decades after actual teletypewriters vanished.

The **line discipline** layer - the part of the kernel that turns raw incoming bytes into "oh, that's a backspace, delete the previous character," or "that's `<Ctrl-C>`, send SIGINT to the foreground process group", is the actual reusable _concept_ of "TTY-ness." 

It's not tied to any specific hardware; it's a kernel service that can sit on top of **any** two-way byte stream.

**`/dev/ttyN`** - These are called virtual consoles, and can be opened using `<Ctrl-Alt-F1>` through `<Ctrl-Alt-F6>`. 

**Virtual console**: the kernel _emulating_ what a physical serial terminal would have been, using your keyboard and framebuffer/screen instead of an actual serial cable. There's no real teletypewriter behind `tty1`, but the kernel-level line discipline (echo, `<Ctrl-C>`, backspace handling) is real and running.

```bash
$ ls -l /dev/tty1 /dev/tty2 /dev/tty3
crw--w---- 1 root tty 4, 1 ... /dev/tty1
crw--w---- 1 root tty 4, 2 ... /dev/tty2
crw--w---- 1 root tty 4, 3 ... /dev/tty3
```

`4` - Handled by the same driver (major number).

**Pseudo terminal devices**

A `ttyN` device is wired straight into a **single physical input path**: the actual keyboard and screen attached to the machine.

**shells (and every other program that cares about terminals like `vim`, `less`, `top`) are written expecting to talk to a real TTY-like device with all that line-discipline behavior**. You can't just pipe bytes through a plain `pipe()` as pipes don't have any of that terminal semantics (no concept of window size, no signal generation on `<Ctrl-C>`, no line editing).

So the kernel needed a way to say: **"give me a TTY-like device, with full line-discipline behavior, that isn't wired to real hardware, instead, wire the _other end_ to another process (my terminal emulator) instead of a serial cable."** That's a **pseudo-terminal (PTY)**.

A pseudo-terminal is always created as a **pair** of connected device endpoints:

- **the master**: held by the terminal emulator (or `sshd`, or `tmux`, or `script`, or anything else that's "driving" a terminal from software instead of hardware)
- **the slave**: presented to the shell (and everything the shell runs) as if it were a completely normal TTY device: it supports the exact same `ioctl()`s, the exact same line discipline, the exact same everything, as `/dev/tty1` does

```
  ┌────────────────────┐                          ┌──────────────────┐
  │  Terminal emulator   │                          │   bash (shell)     │
  │  (Alacritty, kitty,   │                          │   and everything   │
  │   or sshd, or tmux)   │                          │   it runs (vim...) │
  │                       │                          │                    │
  │   writes keystrokes   │                          │  reads from,       │
  │   into the MASTER  ───┼──── kernel PTY driver ───┼─► writes to the   │
  │   reads program       │      (line discipline     │  SLAVE, exactly   │
  │   output from the     │       lives here, same     │  as if it were a  │
  │   MASTER            ◄─┼──── as tty1's driver) ────┼─  real /dev/ttyN  │
  └────────────────────┘                          └──────────────────┘
        /dev/ptmx                                     /dev/pts/N
        (or /dev/pty*                                  (the SLAVE
         historically)                                   device node)
```

**The shell has no idea it's not talking to real hardware.** It calls the exact same `ioctl(TIOCGWINSZ)` to ask "how big is my window" that it would on a real console, the terminal emulator (as master) is the one that answers that query, based on the actual pixel size of its window divided by font metrics. 

`<Ctrl-C>` you press in your terminal window travels: keyboard → terminal emulator → written into the master → kernel's line discipline (running on the slave side) sees the interrupt character → generates SIGINT → delivered to the shell's foreground process group. Same mechanism as a real console, same kernel code path, just with a software process instead of a serial cable on the other end.


The modern (Unix98) PTY allocation scheme works like this:

bash

```bash
$ ls -l /dev/ptmx
crw-rw-rw- 1 root root 5, 2 ... /dev/ptmx
```

`/dev/ptmx` is the **PTY multiplexer**: a single, **special device file**. When your terminal emulator wants to open a _new_ pseudo-terminal, it opens `/dev/ptmx`. 

Each `open()` call on `/dev/ptmx` hands back a fresh master file descriptor, and as a side effect, the kernel automatically creates a brand-new** slave device** node under `/dev/pts/`, numbered sequentially:

bash

```bash
$ ls /dev/pts/
0  1  2  3  ptmx
```

Something like `/dev/pts/3` isn't a fixed, pre-existing device the way `/dev/tty3` is, it's **dynamically created the moment something opens `/dev/ptmx` for the fourth time this boot** (0-indexed), and it's the _slave_ end of that specific master/slave pair. 

It's a real device node, you can `ls -l` it, `stat` it, but its whole lifetime is tied to "as long as some master has it open." Close the terminal emulator window, and `/dev/pts/3` disappears.

What is `/dev/pts`? - It's just a folder:

```
❯ ls -l /dev | grep pts
drwxr-xr-x   2 root    root               0 Sep 24 20:19 pts
```

When a process opens `/dev/tty`, the kernel doesn't connect it to some fixed device, it redirects the open to **whatever that specific process's controlling terminal currently is**. If your shell's controlling terminal is `/dev/pts/3`, then for _that shell and its children_, opening `/dev/tty` is equivalent to opening `/dev/pts/3`. For a process whose controlling terminal is `/dev/tty2`, opening `/dev/tty` redirects there instead. Same device file, different actual target, depending entirely on who's asking.

bash

```bash
$ tty                     # tells you what YOUR controlling terminal resolves to
/dev/pts/3
$ echo hi > /dev/tty      # writes to /dev/pts/3, because that's what
                           # /dev/tty resolves to for THIS shell
```
**Ok, but what is a controlling terminal?**

A controlling terminal is a terminal device (virtual console, PTY slave, serial line or any TTY-driver device) that has been designated as the terminal **for an entire session** of processes, not for one specific process. It plays three distinct roles:

1. **Signal dispatch**: when you press `<Ctrl-C>` or `<Ctrl-Z>` at that terminal, the kernel's line discipline generates SIGINT or SIGTSTP and delivers it to the **foreground process group** attached to that terminal, not to one specific PID, to a _group_ of them at once. 
2. **The `/dev/tty` redirect target**: as we covered, opening `/dev/tty` resolves to whatever the calling process's controlling terminal currently is.
3. **Hangup notification**: if the terminal itself goes away (you close the terminal emulator window, or an actual serial line drops), the kernel sends SIGHUP to the session.

Its job is to be the _addressable target_ for keyboard-generated signals and I/O for an entire **session** (a group of related processes), not an individual process.

```
                     SESSION
        (created by setsid, roughly = "one login")
                        │
         ┌──────────────┼──────────────────┐
         │                                   │
   process group A                    process group B
   (foreground)                       (background)
         │                                   │
     bash ── vim                        sleep 100 &

  The controlling terminal belongs to the SESSION.
  Only ONE process group within that session is the
  "foreground" group at any moment -- that's the one
  that receives Ctrl-C / Ctrl-Z generated signals.
```

This is exactly what's happening when you background a job:

bash

```bash
$ sleep 100 &
[1] 5821
$ fg
```

`sleep 100` still belongs to the _same session_ and has the _same controlling terminal_ as your shell, but it's not in the **foreground process group**, so pressing `<Ctrl-C>` while it's backgrounded doesn't touch it; the signal only goes to whichever process group the terminal currently considers foreground (your interactive shell, at that moment). `fg` moves it back into the foreground group, and now `<Ctrl-C>` would reach it.

You can see these processes have the same session ID and same TT:

```bash
$ ps -o pid,pgid,sid,tty,stat,comm
   PID  PGID   SID TT       STAT COMMAND
  4821  4821  4821 pts/3    Ss   bash
  5821  5821  4821 pts/3    S    sleep
```

**Does every process need a controlling terminal?**

**No.**

```bash
$ ps -eo pid,tty,comm | head -8
    PID TT   COMMAND
      1 ?    systemd
    512 ?    systemd-journald
   4821 pts/3 bash
   5821 pts/3 sleep
```

Usually, daemons won't have a controlling terminal. 

If `systemd-journald` _had_ a controlling terminal, then closing that terminal (you log out, or your SSH session drops) would send SIGHUP to it and potentially kill it, which is extremely undesirable for something that's supposed to run forever, independent of any particular login session. So daemons deliberately detach.


The way you _become_ session-leader-with-no-controlling-terminal is the `setsid()` system call: it makes the calling process the leader of a **brand new session**, with no process group members yet and, crucially, **no controlling terminal**. A fresh session starts with none by default. 

This is exactly what proper daemonizing code does, and it's also a command you can run yourself:

```bash
$ setsid ./my-daemon-script.sh &
```

```bash
$ setsid sleep 300 &
$ ps -eo pid,tty,comm | grep sleep
   5900 ?    sleep
```
Now, nothing you type in that window can send it a keyboard-generated signal anymore; it's fully detached. This is genuinely the mechanism `systemd` itself uses (among other steps) when starting your `.service` units.

**What can act as a controlliing terminal?**

**Any TTY-driver device can serve as a controlling terminal**, because "being a controlling terminal" isn't a property of the specific hardware or software behind the device, it's a _role_ the session mechanism assigns to whatever TTY-like device a session's leader happens to open. Concretely:

- Virtual consoles (`/dev/tty1`, `/dev/tty2`, ...): your login shell at a raw `<Ctrl-Alt-F2>` console has that `ttyN` as its controlling terminal.
- PTY slaves (`/dev/pts/N`): your terminal-emulator shell, or an SSH session's shell, has the allocated `pts/N` as its controlling terminal.
- Real serial lines (`/dev/ttySN`): genuinely rare today, but historically, a shell logged in over an actual serial cable had that as its controlling terminal.


Sorry, I got carried away. 


**Local domain sockets**

Sockets are connections between processes that allow them to communicate hygienically. UNIX defines several kinds of sockets, most of which involve the network.

Local domain sockets are accessible only from the local host and are referred to through a filesystem object rather than a network port. They are sometimes known as “UNIX domain sockets.” Syslog and the X Window System are examples of standard facilities that use local domain sockets, but there are many more, including many databases and app servers.

Local domain sockets are created with the `socket` system call and removed with the
`rm` command or the `unlink` system call once they have no more users.


**Named pipes**

Like local domain sockets, named pipes allow communication between two processes running on the same host. They’re also known as “FIFO files”.


**Symbolic links**

Already covered on `Symbolic links and Hard links.md`.


