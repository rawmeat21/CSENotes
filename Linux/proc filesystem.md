
The Linux versions of `ps` and `top` read their process status information from the
`/proc` directory, **a pseudo-filesystem in which the kernel exposes a variety of interesting information about the system’s state.**

The information is NOT limited to process information.

Note: Because the kernel creates the contents of `/proc` files on the fly (as they are read),
most appear to be empty, 0-byte files when listed with `ls -l`. You’ll have to `cat` or `less`
the contents to see what they actually contain.

![Pasted image 20260911233227](../assets/Pasted%20image%2020260911233227.png)

**The individual components contained within the cmdline and environ files are
separated by null characters rather than newlines. You can filter their contents
through `tr "\000" "\n"` to make them more readable.


```bash
$ echo $$ # get terminal emulator's PID

$ ls /proc/$(echo $$)
cmdline  cwd  environ  exe  fd  fdinfo  maps  ns  root  stat  statm  status  ...

```

**`cmdline`** — the exact invocation, null-separated:

```bash
❯ cat /proc/$(echo $$)/cmdline
/usr/bin/zsh% 
```

**`environ`** — every environment variable the process was started with, also null-separated:

```
$ cat /proc/54321/environ | tr '\0' '\n' | head -5
SHELL=/bin/bash
PWD=/home/qing
LOGNAME=qing
```

**`exe`** — a symlink to the actual binary being executed

**`fd`** — every open file descriptor, as symlinks. This is _the_ tool for answering "what files does this process have open right now":

bash

```bash
$ ls -l /proc/54321/fd
lrwx------ 1 qing qing 64 ... 0 -> /dev/pts/3
lrwx------ 1 qing qing 64 ... 1 -> /dev/pts/3
lrwx------ 1 qing qing 64 ... 2 -> /dev/pts/3
```

**`maps`** — what libraries a process is linked against, live, without needing `ldd`:

bash

```bash
$ cat /proc/54321/maps | grep '\.so' | awk '{print $6}' | sort -u | head
/usr/lib/libc.so.6
/usr/lib/libreadline.so.8
/usr/lib/libtinfo.so.6
```


**`cgroup`** — systemd puts _every_ process into a cgroup (control group) as part of normal operation, not just containers:

bash

```bash
$ cat /proc/54321/cgroup
0::/user.slice/user-1000.slice/session-2.scope
```




