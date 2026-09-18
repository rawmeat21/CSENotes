The following article has been created using:

Claude
https://medium.com/@M4verick/how-i-went-from-confused-to-confident-understanding-ttys-ptys-ssh-and-tmux-d34252c0e452
https://www.warp.dev/blog/what-happens-when-you-open-a-terminal-and-enter-ls


- **Shell** — a program that reads commands and executes them (bash, zsh, fish). It's just a regular process, like any other. Its job is to parse input and fork/exec other processes.
- **Terminal** — historically, a physical device (a screen + keyboard, or even older, a teletype/printer) that let a human talk to a computer. In modern Linux, this role is played by a **TTY device** (a kernel abstraction — more below). A terminal emulator is a GUI application. 
- **Terminal emulator** — a GUI application (xterm, GNOME Terminal, Konsole, Alacritty, kitty) that _pretends to be_ an old physical terminal. It draws characters on screen and provides input, but internally it talks to the kernel through the TTY layer. 

![[Pasted image 20260918144951.png]]
An IBM 2741 teletype and the IBM System/360 Mo. 40 mainframe computer. These were released in the late 60s, and were prevalent until the 70s. The purchase of one of these mainframes (>$200k at the time) included a teletype.


![[Pasted image 20260918145317.png]]
A VT100 (VT = video terminal), released in 1978 by DEC.[5](https://www.warp.dev/blog/what-happens-when-you-open-a-terminal-and-enter-ls#footnote-five) This model implemented and popularized the ANSI escape codes which are still used today

### What is a shell?

A shell is a program that reads commands and executes them. It doesn't draw pixels, manage windows, or talk to the display server/GPU directly. It reads text from stdin, interprets it (parsing, expansion, running programs, handling pipes/redirects), and writes text to stdout/stderr. That's it.

The work of running other programs as processes and interpreting the commands you write is done by a shell. There are several choices of shell. Popular ones include [Bash](https://en.wikipedia.org/wiki/Bash_\(Unix_shell)), [Zsh](https://en.wikipedia.org/wiki/Z_shell), and [fish](https://en.wikipedia.org/wiki/Fish_\(Unix_shell)).


## More about the terminal emulator

To get things started, the terminal needs to spawn a process for the shell the user wants, as well as a method for communicating with the shell and with any processes the shell starts. This communication is done through PTY.

#### Creating the PTY

Before the terminal spawns a child process for the shell, it establishes a way to communicate with it. Much like their teletype ancestors, terminal emulators work by streaming characters as the user types them.

Streaming into _what_, though? There is no longer a wire to another computer because everything is happening on one system. 

Instead, the wires of the traditional TTY are replaced with pairs of **file descriptors** known as the [PTY](https://www.baeldung.com/linux/pty-vs-tty#bd-what-is-a-pty), short for pseudo-TTY. These files are like the two ends of that wire that transmits user-entered input to programs and sends back the output.

The terminal asks the kernel to create these files. 

There is still a TTY driver in the kernel with a line discipline responsible for mediating the data between the two ends of the PTY. One end is the **leader [6](https://www.warp.dev/blog/what-happens-when-you-open-a-terminal-and-enter-ls#footnote-six)**, intended for the terminal to interface with, write user input to, and read output from for display to the user. The other end is the **follower**, which will be used by the shell and all other processes created in the session.

PTY _leader_ is the end of the PTY that interfaces with the _terminal emulator_, while the PTY _follower_ interfaces with the _shell_.

![[Pasted image 20260918150058.png]]


These file descriptors (fd) are not “normal files,” but virtual [character devices](https://unix.stackexchange.com/questions/37829/how-do-character-device-or-character-special-files-work). 

The fd for the leader just points to a buffer in memory, while the follower is a character device file with an actual path on disk. 

If you want to see what that path is, run the `tty` command from the terminal. You can write to this path from a different process and see the data you write appear in the other session!

![[9_Gn_M8g_0d3b281ed5.gif]]


#### Spawning the Shell

Before receiving the user's first command, the terminal has one remaining task: spawn the shell process.

The shell is mainly responsible for:

- conditional statements, loops, parallelism
- creating the child processes for each command that the user wants to run

The shell is the first child process of the terminal session.

The terminal will spawn it and set it to read and write from the PTY **follower**. It does this by setting the shell’s stdin, stderr and stdout (fd 0 through 2) to the PTY follower.

![[Pasted image 20260918150805.png]]


#### Shell initialisation

When the shell initializes, it runs some startup scripts to enable users to customize the experience. This may involve setting environment variables, aliases, functions, or printing any information the user may want to see when the session starts. The exact paths of these scripts depend on a couple of things: which shell you’re using and whether or not the session is a login shell.

-> Login shells are shell sessions wherein this shell process is the one the user is logging into the system with; the process is the first process under this user ID. For example, if you log into a server running the Linux distribution Ubuntu Server, which doesn’t have a GUI environment, you’ll be logging in with a shell and that is a login shell. 

-> On the other hand, if you are already logged in and you start a subshell in a session, it will be a non-login shell.


```
┌─────────────────────────────────────┐
│  Terminal Emulator (e.g. GNOME      │
│  Terminal, iTerm2, Alacritty)       │
│  - Opens a GUI window               │
│  - Renders glyphs/fonts to pixels   │
│  - Handles scrollback, colors,      │
│    resizing, copy-paste             │
│  - Creates a pseudo-terminal (PTY)  │
└───────────────┬───────────────────---┘
                │ PTY (text stream)
                ▼
┌───────────────────────────────────────┐
│  Shell (e.g. bash, zsh, fish)         │
│  - Reads text commands from the PTY   │
│  - Parses/interprets them             │
│  - Forks & execs programs             │
│  - Writes text output back to the PTY │
└────────────────────────────────────────┘
```

```
┌─────────────────────────────────────┐
│         Terminal Emulator            │
│                                       │
│   keyboard events                     │
│        │                              │
│        ▼                              │
│   write(master_fd, bytes) ───────────┼──► INPUT: user → PTY master
│                                       │
│   read(master_fd, buf) ◄─────────────┼──► OUTPUT: PTY master → user
│        │                              │
│        ▼                              │
│   interpret bytes/escape sequences    │
│   → rasterize glyphs onto the window  │
└─────────────────────────────────────┘
```

1. **Input path**: you type a key → terminal emulator writes those bytes into the PTY master fd.
2. **Output path**: the shell writes output to the PTY slave → kernel shuttles it to the master → terminal emulator **reads** from the master fd → interprets that byte stream (plain text _and_ embedded escape/control sequences for cursor movement, color, clearing lines, etc.) → draws the corresponding glyphs/colors/cursor position onto its GUI window.

**A terminal emulator is a GUI program that turns keystrokes into bytes written to the PTY master, and turns bytes read back from the PTY master into pixels on screen**.


#### Typing a command into the emulator

When you type in the terminal, the keystrokes are first translated to ASCII characters (e.g. the backspace key is translated to the ASCII character 0x08). These characters are then written to the PTY leader by the terminal.

The TTY driver then reads the characters from the PTY leader and stores them in its line discipline, which acts as an intermediate buffer between the PTY ends. The line discipline’s job is to interpret the characters from the PTY leader its own character set and then process them. How a character is processed by the line discipline is solely dependent on the character itself.

Let’s consider two categories of line discipline characters

1. special characters[7](https://www.warp.dev/blog/what-happens-when-you-open-a-terminal-and-enter-ls#footnote-seven) (e.g. ERASE, INTR)
2. everything else (e.g. characters like “l” and “s”)

Depending on the special character, the line discipline will decide whether it needs to write back to the PTY leader, write through to the PTY follower, or both. 

For example, when the line discipline receives a BS character (ASCII 0x08) which is entered by the `backspace` key, it interprets it as an `ERASE` character. To process it, the line discipline will edit its internal buffer by removing the last character and then writing the delete intent back to the PTY leader. The terminal emulator can then read the change from the PTY leader and reflect it in the terminal display. Notice that the PTY follower was never written to in this case.

On the other hand, when the line discipline receives an ETX character (ASCII 0x03) which is entered by the keys `CTRL-C` (and displayed as ^C), the line discipline will interpret it as an `INTR` (short for **INT**E**R**RUPT) character: it’ll send a SIGINT to the PTY follower in order to interrupt any processes running in the foreground (i.e. programs reading input from and writing output to the terminal). Note that background processes are unaffected.[8](https://www.warp.dev/blog/what-happens-when-you-open-a-terminal-and-enter-ls#footnote-eight) The line discipline will also write the ETX + linefeed characters back to the PTY leader, which is why the terminal emulator then displays `^C` and moves to the next line.

For all non-special characters, like the friendly “l” and “s”, the line discipline will just write the character back to the PTY leader. Since they are written back to the leader, the terminal program reads them back out and into the display which creates the “echo” effect of characters as they’re typed. Otherwise, you wouldn’t actually be able to see characters while you type!

**Note:** While the line discipline is a useful construct, most modern shells actually disable the line discipline’s editor and “echoing” feature. Instead, characters are buffered by the shell process itself so that the shell can implement features like tab completions and [autosuggestions](https://fishshell.com/docs/current/interactive.html#autosuggestions) which _need_ to see characters as they’re typed. That being said, parts of the line discipline are still used. For example, the handling of some special characters, like ^C, is usually still delegated to the line discipline. That way, if the shell process is too busy to handle user input (e.g. it’s in an infinite loop), the foreground process can still be sent an interrupt signal. A shell can control these terminal settings via the [termios](https://man7.org/linux/man-pages/man3/termios.3.html) interface.

#### Pressing enter

Once the `Enter` key is pressed, the terminal will send the CR ASCII character to the line discipline which will interpret it the same as an NL character. 

To process this character, the line discipline will forward its internal buffer along with the line feed to the program listening on the PTY follower (i.e. the shell). From this point, the shell takes over.

#### Parsing the command

Once the shell receives the user input and linefeed, it begins to parse the command to figure out what it means.

First, the command is tokenized and syntactically/semantically analyzed.

---> When the shell locates the executable, it will fork a process and run the executable in the child process, passing along any command arguments.

To visualize these different processes, you can think of the terminal emulator as the root of a process tree where one if its children is the shell itself and any programs you run are descendants thereof. 

In fact, you can visualize this tree with the `pstree` command. All you have to do is provide the process ID of the terminal:

![[Pasted image 20260918152517.png]]

In this example, the terminal emulator process (Warp) has PID 84860 and the tabs/shell processes have PIDs 84890, 85525 and 86041. In one of the tabs (PID 86041), we’re running `tmux` and hence, it’s a direct child of that shell process itself, as expected!

#### Returning output

Say we ran `ls`. Let’s assume `ls` resolves to `/bin/ls`. As mentioned before, the shell will fork a child process and have it run `/bin/ls` within.

Since the child process inherits its parents’ file descriptors, the output produced by the child process will get written to the PTY follower, which is shuffled along to the line discipline. 

Instead of processing these bytes, the line discipline will just forward them to the PTY leader. The terminal emulator app will then read the characters from the PTY leader and display them on the screen.

![[Pasted image 20260918152757.png]]


Oftentimes, output will have text decorations like colors and bolding. For example, in the following command/output pair, the output is intentionally colored (directories are a different color from simple files, which are a different color from executables):

![[Pasted image 20260918152903.png]]

So how did `ls` emit these colors? And how did the terminal emulator know what to do with them? The answer is escape sequences!

#### Escape Sequences

The shell (and other programs) can emit more than just plain-old-characters for the terminal to print. They can emit _escape sequences_ to have _control_ over the terminal, including text decorations, moving the cursor, scrolling, etc.

In the above case, the directories are printed a different color because there is an escape sequence to first change the foreground color before the characters for the directory name are emitted. So by the time the characters for the directory name reach the terminal, the terminal will know to print these characters with a certain color.

It’s the terminal's job to actually render the characters on the screen with an appropriate color (an incompatible terminal would just ignore the escape sequences).


Uff, that was a lot. Happy?

### How a process gets launched

Remember that any process or application that gets launched, does so through a `fork() + exec()` mechanism. Any binary can launch any other binary, as long it has this mechanism in it's code. 

As Claude says: Your shell, Hyprland, rofi, GNOME Shell, `systemd` — they're all doing the identical fork+execve dance underneath.

A few refinements worth having, though:

**Fork isn't always required.** If a process wants to _become_ another program rather than spawn a child, it can call `execve()` alone, with no `fork()` first — this replaces the calling process's own memory image, same PID. This is what the `exec` builtin does in bash: `exec firefox` doesn't create a new process, it turns your current shell process _into_ Firefox (the shell is gone, no child left behind to `wait()` on). Login shells, and things like `exec-once`-style tools, sometimes exploit this to avoid leaving an extra parent process hanging around.

**Not everything that starts a program does it directly.** Some things request a launch _indirectly_, without calling fork/exec themselves at all:

- **D-Bus activation**: a process sends a message like "please start `org.mozilla.firefox`" over D-Bus, and some _other_ process (`dbus-daemon`, or `systemd` via D-Bus activation) is the one that actually forks+execs it.
- **systemd** (`systemctl start foo.service`, or `systemd --user` launching your desktop session's apps): you ask systemd, systemd does the fork+exec.


### What is a TTY, really?

"TTY" = **teletypewriter**, a holdover name from actual hardware teleprinters used with early Unix systems in the 1970s.

Teletypes were basic text clients to allow users to interact with a computer. It’s short for “teletypewriter” because they descend from typewriters, and are partly mechanical devices. They communicated with the computer via a physical wire connecting the two devices. The communication worked like this:

1. ASCII text would be transmitted character-by-character over the wire as the user typed.
2. The kernel of the mainframe would receive the input and decode it.
3. The text gets sent to a driver called the **TTY driver**. This kernel module was responsible for sending this input to user programs and collecting the output.
4. Finally, the kernel sends that output back to the teletype for display to the user.

One thing to mention is the **line discipline**, which buffered the characters in kernel memory. The program wouldn’t receive the input until “Enter” was pressed. The line discipline allowed this buffer to be editable and provided some program-independent shortcuts, e.g. ctrl-w. It was also an important performance optimization at the time because asking the program to react to every individual character was highly inefficient.

In modern operating systems: Each `/dev/ttyX` device represents an entire **keyboard and screen session**.

A TTY is an interactive communication channel between **you** and the **system**. When you open a terminal directly on your computer (without using remote access), you are interfacing with one of these TTYs.

![[Pasted image 20260918140042.png]]


There are two kinds you'll encounter:

1. **Virtual consoles / virtual terminals (VTs)** — these are what `Ctrl+Alt+F1`–`F7` switches between. They're driven directly by the kernel (via the `vt` subsystem), and each one is a device node like `/dev/tty1`, `/dev/tty2`, etc. No terminal emulator program is involved — the kernel itself is drawing text to the screen and reading your keyboard.
2. **Pseudo-terminals (PTYs)** — these are what terminal emulator apps (and SSH sessions) use. A PTY is a _pair_ of virtual devices: a "master" side (held by the terminal emulator/SSH server) and a "slave" side (`/dev/pts/0`, `/dev/pts/1`, etc. — which is what your shell thinks is its terminal). The kernel just shuttles bytes between master and slave; the terminal emulator does the actual rendering to a window.

```
Virtual Console (Ctrl+Alt+F1):
  keyboard -> kernel (vt subsystem) -> /dev/tty1 -> shell -> screen (kernel draws text directly)

Terminal emulator (e.g. GNOME Terminal) in your GUI:
  keyboard -> GUI window -> PTY master (held by terminal emulator)
                                |
                                v
                          PTY slave (/dev/pts/3) -> shell -> back up to GUI window for rendering
```

#### What Happens When I Open a Local Terminal?

When you open a terminal window on a graphical desktop (Linux, macOS), something slightly different happens:

Instead of `/dev/ttyX`, your session attaches to **pseudo-terminal devices** like `/dev/ttys000`, `/dev/ttys001`, and so on. Each terminal window or tab is assigned a new **pseudo-terminal** (sometimes called a **PTY**). 

#### What a PTY is

A **PTY (pseudo-terminal)** is a kernel-provided _fake_ serial terminal device, existing purely in software, with two ends:

- **Master** — a regular file descriptor, held by a userspace program (the terminal emulator)
- **Slave** — `/dev/pts/N`, which _behaves exactly like_ a real hardware terminal device, held by the shell (or vim, ssh, python, whatever)

Anything written to one end can be read from the other, and vice versa — but critically, it's not a dumb pipe. The kernel's PTY subsystem in between applies **the same TTY line-discipline logic that used to run for physical serial terminals**: cooked-mode editing (Backspace, Ctrl+U), turning Ctrl+C into `SIGINT` sent to the foreground process group, tracking terminal size (`ioctl TIOCGWINSZ`), etc.


##### Why does the terminal emulator talk to the PTY to talk to the shell?

**Because bash is a program written for a kernel-managed hardware terminal that no longer physically exists, and the PTY is the kernel's software illusion of that hardware — giving bash the terminal _services_ it needs (signals, line discipline, raw mode) while giving the GUI terminal emulator, on the other end, just a plain byte stream it can read and write like any normal file.**


