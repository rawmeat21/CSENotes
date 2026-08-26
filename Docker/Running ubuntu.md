```bash
docker run -t ubuntu
```
This will open up a tty. But commands don't work. 

Use `-i` (interactive) to pass STDIN to container: 

```bash
docker run -it ubuntu
```

When to use? Suppose you want to run a script which takes input:

```bash
docker exec jovial_benz sh -c 'while true; do echo "Input website:"; read website; echo "Searching.."; sleep 1; curl http://$website; done'
```

This doesn't work, you can't enter the website. So use `-i`:

```bash
docker exec -it jovial_benz sh -c 'while true; do echo "Input website:"; read website; echo "Searching.."; sleep 1; curl http://$website; done'
```


Now we can type commands.

```bash
docker run -d -it --name looper ubuntu sh -c 'while true; do date; sleep 1; done'
```

`-d` - in background.

`--name` - give a name to container.
`sh -c 'shell langauge'` - Used to run shell commands

```bash
$ docker logs -f looper <-- see what is ubuntu doing
  Sat Mar  1 15:51:29 UTC 2025
  Sat Mar  1 15:51:30 UTC 2025
  Sat Mar  1 15:51:31 UTC 2025
```

To makes this foreground:
```bash
$ docker attach looper
  Sat Mar  1 15:54:38 UTC 2025
  Sat Mar  1 15:54:39 UTC 2025
  ...
```

Now you have process logs (STDOUT) running in two terminals. Now press _control+c_ in the attached window. The container is stopped because the process is no longer running.

If we want to attach to a container while making sure we don't close it from the other terminal we can specify to not attach STDIN with `--no-stdin` option.

```bash
$ docker start looper # used to start a stopped container

$ docker attach --no-stdin looper
  Thu Mar  1 15:56:11 UTC 2023
  Thu Mar  1 15:56:12 UTC 2023
```

Now you can do Ctrl+C again. The container will continue running. Control+c now only disconnects you from the STDOUT.


## Running processes inside a container with docker exec

Note: ONLY works on a running container

```bash
$ docker exec looper ls -la
total 56
drwxr-xr-x   1 root root 4096 Mar  6 10:24 .
drwxr-xr-x   1 root root 4096 Mar  6 10:24 ..
-rwxr-xr-x   1 root root    0 Mar  6 10:24 .dockerenv
lrwxrwxrwx   1 root root    7 Feb 27 16:01 bin -> usr/bin
drwxr-xr-x   2 root root 4096 Apr 18  2022 boot
drwxr-xr-x   5 root root  360 Mar  6 10:24 dev
drwxr-xr-x   1 root root 4096 Mar  6 10:24 etc
drwxr-xr-x   2 root root 4096 Apr 18  2022 home
lrwxrwxrwx   1 root root    7 Feb 27 16:01 lib -> usr/lib
drwxr-xr-x   2 root root 4096 Feb 27 16:01 media
drwxr-xr-x   2 root root 4096 Feb 27 16:01 mnt
drwxr-xr-x   2 root root 4096 Feb 27 16:01 opt
dr-xr-xr-x 293 root root    0 Mar  6 10:24 proc
drwx------   2 root root 4096 Feb 27 16:08 root
drwxr-xr-x   5 root root 4096 Feb 27 16:08 run
lrwxrwxrwx   1 root root    8 Feb 27 16:01 sbin -> usr/sbin
drwxr-xr-x   2 root root 4096 Feb 27 16:01 srv
dr-xr-xr-x  13 root root    0 Mar  6 10:24 sys
drwxrwxrwt   2 root root 4096 Feb 27 16:08 tmp
drwxr-xr-x  11 root root 4096 Feb 27 16:01 usr
drwxr-xr-x  11 root root 4096 Feb 27 16:08 var
```

We can execute the Bash shell in the container in interactive mode and then run any commands within that Bash session:

```bash
$ docker exec -it looper bash

  root@2a49df3ba735:/# ps aux <-- we can enter commands easily

  USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
  root         1  0.2  0.0   2612  1512 pts/0    Ss+  12:36   0:00 sh -c while true; do date; sleep 1; done
  root        64  1.5  0.0   4112  3460 pts/1    Ss   12:36   0:00 bash
  root        79  0.0  0.0   2512   584 pts/0    S+   12:36   0:00 sleep 1
  root        80  0.0  0.0   5900  2844 pts/1    R+   12:36   0:00 ps aux
```

How to kill this? If you try `docker stop looper` it won't work:

![[Pasted image 20260820092743.png]]

Its frozen.

```bash
docker kill looper && docker rm looper
```

Use this instead, or: 

`docker rm --force looper` <--- use this when `docker stop` doesn't work.



### Remove automatically after exit: use `--rm`.

```bash
docker run -d --rm -it --name looper-it ubuntu sh -c 'while true; do date; sleep 1; done'
```
#### What is the point of running an interactive terminal and putting it on background?

##### Technical Reasons to Combine `-d` and `-it`

- **Prevents Interactive Shells from Immediately Exiting:** If you start a container running an interactive shell (`docker run -d ubuntu bash`), `bash` attempts to read from `STDIN`. Without `-i`, `STDIN` receives an immediate `EOF`, causing PID 1 to terminate and the container to exit. Adding `-it` keeps `STDIN` open and attached to a PTY in the background, allowing the container to remain running continuously until explicitly stopped or attached to.
    
- **Preserves PTY Capabilities for `docker attach`:** If you later run `docker attach <container_name>`, the attached session inherits the stream properties set at container creation. If `-it` was passed on creation, `docker attach` provides a full, interactive terminal session complete with `readline` support, tab completion, and proper signal processing. Without `-t` allocated at creation, attaching only pipes raw, unformatted `STDOUT`/`STDERR` streams.
    
- **Changes Stream Buffering & Terminal Detection (`isatty`):** Allocating a TTY with `-t` causes the `isatty(STDOUT_FILENO)` system call to evaluate to `true` inside the process. Many logging frameworks, CLI tools, and runtimes (like Python or Node.js) check `isatty()` to decide buffering strategies:
    
    - **With `-t` (TTY present):** `STDOUT` operates in **line-buffered** mode, flushing output immediately after every newline (`\n`), and applications enable ANSI color escape codes.
        
    - **Without `-t` (No TTY):** `STDOUT` switches to **block-buffered** mode (typically 4KB/8KB buffers), delaying output until the buffer fills or the process exits.
        
- **Signal Trapping via PTY Line Discipline:** A process running inside a PTY receives standard POSIX terminal signals (such as `SIGINT` from `Ctrl+C` or `SIGTSTP` from `Ctrl+Z`) through the kernel's TTY line discipline driver when attached.
    

In this specific command, the `-it` flags interact as follows:

1. **`-i` is redundant here:** The `sh -c` loop does not read from `STDIN`, so keeping `STDIN` open does not alter the execution loop.
    
2. **`-t` alters stdout buffering:** It forces the `date` command and shell to run with a PTY, ensuring output is line-buffered in `docker logs -f` rather than block-buffered.
    
3. **`-it` enables interactive control upon attachment:** If you execute `docker attach looper-it`, the pre-allocated PTY allows you to send `Ctrl+C` (`SIGINT`) directly to the `sh` process loop to terminate it. Without `-t`, `Ctrl+C` over `docker attach` would not pass through the PTY line discipline to stop the loop.


Now let's attach to the container and hit _control+p_, _control+q_ to detach us from the STDOUT.

```bash
$ docker attach looper-it

  Sat Mar 01 19:50:42 UTC 2025
  Sat Mar 01 19:50:43 UTC 2025
  ^P^Qread escape sequence
```

Instead, if we had used _ctrl+c_, it would have sent a kill signal followed by removing the container as we specified `--rm` in `docker run` command.

`docker stop looper` also removes the image directly.


**Question:** Image `devopsdockeruh/simple-web-service:ubuntu` will start a container that outputs logs into a file. Go inside the running container and use `tail -f ./text.log` to follow the logs. Every 10 seconds the clock will send you a "secret message".

What to run:

```bash
$ docker exec -it jovial_benz bash
root@fc2959c980b6:/usr/src/app# tail -f ./text.log
2026-08-20 04:09:37 +0000 UTC
2026-08-20 04:09:39 +0000 UTC
Secret message is: 'You can find the source code here: https://github.com/docker-hy'
```

To stop:

```bash
$ docker stop jovial_benz
```










