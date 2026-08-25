```elixir
docker run -t ubuntu
```
This will open up a tty. But commands don't work. 

Use `-i` (interactive) to pass STDIN to container: 

```tcl
docker run -it ubuntu
```

When to use? Suppose yo want to run a script which takes input:

```bash
docker exec jovial_benz sh -c 'while true; do echo "Input website:"; read website; echo "Searching.."; sleep 1; curl http://$website; done'
```

This doesn't work, you can't enter the website. So use `-i`:

```bash
docker exec -it jovial_benz sh -c 'while true; do echo "Input website:"; read website; echo "Searching.."; sleep 1; curl http://$website; done'
```


Now we can type commands.

```gauss
docker run -d -it --name looper ubuntu sh -c 'while true; do date; sleep 1; done'
```

`-d` - in background.

`--name` - give a name to container.
`sh -c 'shell langauge'` - Used to run shell commands

```armasm
$ docker logs -f looper <-- see what is ubuntu doing
  Sat Mar  1 15:51:29 UTC 2025
  Sat Mar  1 15:51:30 UTC 2025
  Sat Mar  1 15:51:31 UTC 2025
```

```armasm
$ docker attach looper
  Sat Mar  1 15:54:38 UTC 2025
  Sat Mar  1 15:54:39 UTC 2025
  ...
```
This makes this foreground again.

Now you have process logs (STDOUT) running in two terminals. Now press _control+c_ in the attached window. The container is stopped because the process is no longer running.

If we want to attach to a container while making sure we don't close it from the other terminal we can specify to not attach STDIN with `--no-stdin` option.

```gams
$ docker start looper # used to start a stopped container

$ docker attach --no-stdin looper
  Thu Mar  1 15:56:11 UTC 2023
  Thu Mar  1 15:56:12 UTC 2023
```

Now you can do Ctrl+C again. The container will continue running. Control+c now only disconnects you from the STDOUT.


## Running processes inside a container with docker exec

```tap
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

```tap
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

`docker rm --force looper`



Remove automatically after exit: use `--rm`.

```stata
docker run -d --rm -it --name looper-it ubuntu sh -c 'while true; do date; sleep 1; done'
```

Now let's attach to the container and hit _control+p_, _control+q_ to detach us from the STDOUT.

```armasm
$ docker attach looper-it

  Sat Mar 01 19:50:42 UTC 2025
  Sat Mar 01 19:50:43 UTC 2025
  ^P^Qread escape sequence
```

Instead, if we had used _ctrl+c_, it would have sent a kill signal followed by removing the container as we specified `--rm` in `docker run` command.

`docker stop looper` also removes the image directly.


**Question:** Image `devopsdockeruh/simple-web-service:ubuntu` will start a container that outputs logs into a file. Go inside the running container and use `tail -f ./text.log` to follow the logs. Every 10 seconds the clock will send you a "secret message".

What to run:

```
$ docker exec -it jovial_benz bash
root@fc2959c980b6:/usr/src/app# tail -f ./text.log
2026-08-20 04:09:37 +0000 UTC
2026-08-20 04:09:39 +0000 UTC
Secret message is: 'You can find the source code here: https://github.com/docker-hy'
```
To stop:

```
$ docker stop jovial_benz
```








