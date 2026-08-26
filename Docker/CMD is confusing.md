### The mental model

Think of it like a function with a default argument:

```
docker run <image> [optional-command-override]
```

- No override given → Docker uses whatever is in `CMD`.
- Override given → Docker uses **only** what you typed, completely replacing `CMD`.

### `docker run -it yt-dlp ps`

**Anything you type after the image name in `docker run` overrides CMD entirely.**

So if the `yt-dlp` image has something like:

```dockerfile
CMD ["yt-dlp", "--help"]
```

then normally `docker run yt-dlp` would run `yt-dlp --help`. But when you run:

```bash
docker run -it yt-dlp ps
```

Docker throws away the whole CMD array and replaces it with `ps`. It does **not** append `ps` to the existing CMD, it substitutes it. So the container tries to run `ps` as its main process instead of whatever CMD said.

### `CMD ["./hello.sh"]`

This isn't "running a command inside the container" as some special container-only thing,  it's just: when someone runs `docker run <image>` with **no extra arguments**, Docker executes `./hello.sh` as the process.

### CMD vs ENTRYPOINT

This is usually where the _real_ confusion comes from, because CMD behaves differently depending on whether `ENTRYPOINT` is also set:

- **CMD alone** → CMD _is_ the whole command. `docker run image foo` replaces the entire CMD with `foo`.
- **ENTRYPOINT + CMD** → ENTRYPOINT is the fixed command that always runs; CMD supplies _default arguments_ to it. `docker run image foo` replaces just the CMD part, so it runs `entrypoint foo` instead of `entrypoint cmd-args`.

If the `yt-dlp` image uses ENTRYPOINT, then `docker run yt-dlp ps` actually runs `<entrypoint> ps` — i.e., it's passing `ps` as an argument to whatever the entrypoint binary is, not literally running the `ps` process command. Worth checking that image's Dockerfile if you want to confirm which case you're in.

### By default, the entrypoint in Docker is set as `/bin/sh -c` in properly set up images

Every container needs a process to actually run — some program has to be PID 1. When you _don't_ specify `ENTRYPOINT` in a Dockerfile, Docker doesn't leave it empty; it silently uses:

```bash
ENTRYPOINT ["/bin/sh", "-c"]
```

as the default. This matters because of how `CMD` and `ENTRYPOINT` combine — Docker always runs:

```bash
<ENTRYPOINT array> + <CMD array>
```

concatenated together as the final command.


Go back to your `hello.sh` Dockerfile:

dockerfile

```dockerfile
CMD ["./hello.sh"]
```

There's no `ENTRYPOINT` here, so Docker falls back to the default `["/bin/sh", "-c"]`. The actual command that runs is the concatenation:

```bash
/bin/sh -c ./hello.sh
```

That's literally shell running your script as an argument. This is what "you're giving the file as a parameter to `/bin/sh -c`" means — `CMD` wasn't magically "running a script inside the container" as some special feature; it was just supplying the argument to the hidden default entrypoint, `sh -c`.


### Exec form vs shell form

This is a separate, related distinction, about _how you write_ CMD/ENTRYPOINT, not what they do.

**Exec form** (what you've been using — JSON array syntax):

dockerfile

```dockerfile
CMD ["./hello.sh"]
ENTRYPOINT ["/usr/local/bin/yt-dlp"]
```

This runs the program **directly** as PID 1, with no shell involved. Docker just executes that binary with those exact arguments.

**Shell form** (plain string, no brackets):

```dockerfile
CMD ./hello.sh
ENTRYPOINT /usr/local/bin/yt-dlp
```

Docker silently wraps this in `/bin/sh -c "..."` for you. So it becomes:

```bash
/bin/sh -c "./hello.sh"
```

### Why exec form is preferred, but shell form has a use case

With **exec form**, there's no shell wrapping — which means:

- Signals (like Ctrl+C / `SIGTERM` from `docker stop`) go straight to your program, so it can shut down cleanly.
- No accidental shell parsing/escaping surprises.

With **shell form**, since there _is_ a real shell running your command, you get shell features — importantly, **environment variable expansion**:

dockerfile

```dockerfile
CMD echo "Password is $MYSQL_PASSWORD"
```

This works in shell form because `/bin/sh` is the one interpreting `$MYSQL_PASSWORD`. If you wrote this in exec form:

dockerfile

```dockerfile
CMD ["echo", "Password is $MYSQL_PASSWORD"]
```

it would print the literal string `$MYSQL_PASSWORD` — nothing to expand it, since no shell is involved.

![[Pasted image 20260826123059.png]]


 `sh -c` syntax is: `sh -c COMMAND_STRING [name] [arg1] [arg2] ...`

- `COMMAND_STRING` = `/bin/ping -c 3` — this is the **only** thing that gets executed.
- Everything after that becomes positional parameters **inside** that command string: `$0`, `$1`, `$2`...

#### Let's look at an example:

```bash
$ docker pull python:3.11
...

$ docker run -it python:3.11
Python 3.11.8 (main, Feb 13 2025, 09:03:56) [GCC 12.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> print("Hello, World!")
Hello, World!
>>> exit()

NOTICE HOW PYTHON STARTED AUTOMATICALLY? THIS MEANS ITS EITHER THE ENTRYPOINT OR CMD. 

$ docker run -it python:3.11 --version
 docker: Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: exec: "--version": executable file not found in $PATH

BUT THIS FAILS! IF PYTHON WAS THE ENTRYPOINT, THEN THIS WOULD HAVE WORKED. SO IT MUST BE THE CMD.

$ docker run -it python:3.11 bash
  root@1b7b99ae2f40:/# python --version
  Python 3.11.14
  root@1b7b99ae2f40:/# exit
```

We can create our own image for personal use with a new Dockerfile:

```dockerfile
FROM python:3.11
ENTRYPOINT ["python3"]
CMD ["--help"]
```


### How can I handle a script which requires arguments?

Simple, say you have: 

```bash
#!/bin/bash

echo "Searching..";
sleep 1;
curl http://$1;
```

```dockerfile
FROM ubuntu:24.04

WORKDIR /usr/src/app


RUN apt-get update
RUN apt-get -y install curl

COPY script.sh .

RUN chmod +x script.sh

ENTRYPOINT ["./script.sh"] <--- NOTICE!

```

Now you can do:

```bash
$ docker build -t curler-v2 .
$ docker run curler-v2 helsinki.fi <--- directly pass argument 1 (it replaces the CMD)

  Searching..
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                   Dload  Upload   Total   Spent    Left  Speed
  100   232  100   232    0     0  13647      0 --:--:-- --:--:-- --:--:-- 13647
  <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
  <html><head>
  <title>301 Moved Permanently</title>
  </head><body>
  <h1>Moved Permanently</h1>
  <p>The document has moved <a href="https://www.helsinki.fi/">here</a>.</p>
  </body></html>
```

