# Bash Tutorial Part 3: Systems-Level Bash (for systems internship prep)

This part assumes Parts 1-2. It focuses on what actually comes up in systems work:
file descriptors, process/job control, IPC internals, performance/anti-patterns,
resource limits, networking without external tools, and debugging.

---

## 1. File descriptors (the real model behind all redirection)

Every process starts with 3 open file descriptors: `0`=stdin, `1`=stdout, `2`=stderr.
`>`, `<`, `2>` etc. are just syntax sugar over manipulating these FD numbers.

```bash
exec 3> output.txt        # open FD 3 for writing, stays open for rest of script
echo "line1" >&3            # write to FD 3
echo "line2" >&3
exec 3>&-                     # close FD 3

exec 4< input.txt           # open FD 4 for reading
read -u 4 line                 # read a line from FD 4 (like reading from a file handle)
exec 4<&-                        # close FD 4

# duplicate a descriptor
exec 5>&1        # save current stdout into FD 5
exec 1> log.txt     # redirect stdout to a file
echo "goes to log.txt"
exec 1>&5              # restore stdout from FD 5
exec 5>&-                 # close the backup

# swap stdout and stderr (classic trick)
command 3>&1 1>&2 2>&3 3>&-
```
Understanding this matters because it's exactly the abstraction the OS gives you
(`open()`, `dup2()`, `close()` in C) — bash redirection is just a thin wrapper over
the same syscalls.

```bash
ls /nonexistent 2>/dev/null       # discard stderr
command > /dev/null 2>&1            # discard both stdout and stderr entirely
command &> /dev/null                  # bash shorthand for the same thing
command > out.log 2> err.log            # separate files per stream
```

---

## 2. Process and job control

```bash
command &         # run in background, returns immediately
jobs                 # list background jobs of current shell
jobs -l                # include PIDs
fg %1                    # bring job 1 to foreground
bg %1                       # resume a stopped job in background
kill %1                        # kill job 1 by job spec
kill -9 1234                      # SIGKILL a process by PID (hard kill)
kill -15 1234                       # SIGTERM (graceful, default signal)
disown %1                              # detach job from shell (survives shell exit)
nohup ./long_task.sh &                    # immune to SIGHUP (survives terminal close)
```

`wait` — synchronize on background jobs (mirrors `waitpid()` in C):
```bash
./task1.sh &
pid1=$!
./task2.sh &
pid2=$!
wait "$pid1" "$pid2"       # block until BOTH finish
echo "both done"

wait -n     # wait for the NEXT job to finish (whichever completes first)
```

Parallel execution pattern (map-reduce style):
```bash
for url in "${urls[@]}"; do
    curl -s "$url" > "out_${url//\//_}.txt" &
done
wait                  # barrier — wait for all fetches before continuing
```

`Ctrl+Z` suspends the foreground job (SIGTSTP) — resume with `fg` or `bg`.

### Process introspection
```bash
ps aux                    # all processes, full listing
ps -ef | grep myprogram      # find PID of a running program
pgrep -f myprogram              # cleaner: get PID(s) directly by name/pattern
pkill -f myprogram                 # kill by name pattern instead of PID
top / htop                            # live resource usage
nice -n 10 ./cpu_heavy.sh                # launch with lower scheduling priority
renice -n 5 -p 1234                        # change priority of a running process
taskset -c 0,1 ./program                     # pin process to specific CPU cores
```

---

## 3. Subshells vs current shell (deep dive — this trips people up constantly)

Anything that forks a new process runs in a **subshell**: `( )`, pipelines `|`, command
substitution `$( )`, background jobs `&`. Variable changes inside these do NOT propagate
back to the parent shell.

```bash
count=0
echo "a b c" | while read -r word; do
    ((count++))          # this runs in a SUBSHELL because of the pipe!
done
echo "$count"              # still 0 — the increments were lost!

# Fix: avoid the pipe subshell using process substitution instead
count=0
while read -r word; do
    ((count++))
done < <(echo "a b c")
echo "$count"    # 3 — correct, no subshell this time
```
This exact gotcha (pipeline = subshell = lost state) is one of the most common real-world
bash bugs and worth knowing cold.

`shopt -s lastpipe` makes the LAST command of a pipeline run in the current shell instead
(only works when job control is off, i.e. inside scripts, not interactive shells):
```bash
shopt -s lastpipe
count=0
echo "a b c" | while read -r word; do ((count++)); done
echo "$count"   # 3, now works
```

---

## 4. Coprocesses (bidirectional pipe to a subprocess)

```bash
coproc MYPROC { bc -l; }        # start `bc` as a coprocess with I/O pipes
echo "5 + 3" >&"${MYPROC[1]}"     # write to its stdin
read -r result <&"${MYPROC[0]}"     # read its stdout
echo "$result"                        # 8
```
This is the bash equivalent of opening a bidirectional pipe to a child process — same
idea as `popen()` in C but with separate read/write file descriptors exposed to you.

---

## 5. Networking without external tools (`/dev/tcp`, `/dev/udp`)

Bash (when built with this feature, true on most Linux distros) can open raw TCP sockets
via a special pseudo-device path — no `curl`/`nc` required:

```bash
exec 3<>/dev/tcp/example.com/80          # open a TCP connection, FD 3 = both read+write
echo -e "GET / HTTP/1.1\r\nHost: example.com\r\nConnection: close\r\n\r\n" >&3
cat <&3                                    # read the raw HTTP response
exec 3<&-                                     # close it

# quick port check (no netcat needed)
timeout 1 bash -c "echo > /dev/tcp/127.0.0.1/22" && echo "port 22 open" || echo "closed"
```
Useful for lightweight health checks / port scanning in scripts where you can't assume
`curl` or `nc` are installed.

`ss` / `netstat` for inspecting sockets:
```bash
ss -tulpn          # all listening TCP/UDP sockets with process info
ss -tan state established     # currently established TCP connections
```

---

## 6. Resource limits (`ulimit`) — directly systems-relevant

```bash
ulimit -a               # show all current limits
ulimit -n                  # max open file descriptors (default often 1024)
ulimit -n 4096                # raise it (soft limit, within hard limit ceiling)
ulimit -u                        # max user processes
ulimit -v                           # max virtual memory (KB)
ulimit -c unlimited                    # allow core dumps of unlimited size
ulimit -Hn                                # show the HARD limit ceiling (soft can't exceed it)
```
Real use case: a server process hitting "too many open files" is a `ulimit -n` problem,
not a bug in the program — classic systems-interview gotcha.

---

## 7. Signals deep dive

```bash
kill -l                    # list all signal names/numbers
kill -SIGTERM 1234            # graceful shutdown request
kill -SIGKILL 1234               # cannot be caught/ignored, immediate termination
kill -SIGSTOP 1234                  # pause process (uncatchable, unlike SIGTSTP)
kill -SIGCONT 1234                     # resume a stopped process
```

Common signals to actually know:
| Signal | Number | Meaning |
|---|---|---|
| SIGHUP | 1 | terminal closed / "reload config" convention |
| SIGINT | 2 | Ctrl+C |
| SIGKILL | 9 | force kill, cannot be caught |
| SIGTERM | 15 | polite "please exit" request (default for `kill`) |
| SIGSTOP | 19 | pause, cannot be caught |
| SIGCONT | 18 | resume after SIGSTOP |
| SIGCHLD | 17 | sent to parent when a child process exits |

`trap` on `DEBUG` — runs before every single command, useful for tracing:
```bash
trap 'echo "about to run: $BASH_COMMAND"' DEBUG
```

`trap` on `SIGCHLD` — react when a background child finishes:
```bash
trap 'echo "a child process exited"' SIGCHLD
sleep 2 &
wait
```

---

## 8. Performance and anti-patterns

### Builtins vs external commands (fork cost matters at scale)
```bash
# SLOW: forks a new process for every iteration
for i in $(seq 1 100000); do
    echo "$i" >> file.txt      # also reopens the file every time!
done

# FAST: no forks, single file open
{
    for ((i = 1; i <= 100000; i++)); do
        echo "$i"
    done
} > file.txt
```
Prefer `(( ))`/`[[ ]]`/parameter expansion (all builtins, no fork) over spawning `expr`,
`sed`, `awk`, `basename`, `cut` in a hot loop — each is a full `fork()+exec()`.

### "Useless Use of Cat" (UUOC) and similar redundant forks
```bash
cat file.txt | grep "pattern"     # forks cat AND grep unnecessarily
grep "pattern" file.txt              # one process, same result

x=$(cat file.txt)      # forks cat
x=$(< file.txt)           # bash builtin, no fork
```

### String building in a loop
```bash
# SLOW-ish for huge N: quadratic string copies
result=""
for s in "${arr[@]}"; do result+="$s,"; done

# for huge datasets, prefer printf + IFS join, or just accept +=  for moderate sizes
result=$(IFS=,; echo "${arr[*]}")
```

### Avoid re-reading files / re-forking subshells inside tight loops
Cache command output once outside the loop rather than calling `$(cmd)` per iteration.

---

## 9. Defensive / production-grade scripting

```bash
#!/usr/bin/env bash
set -euo pipefail    # e: exit on error, u: error on unset var, pipefail: catch pipe failures
IFS=$'\n\t'             # safer default word-splitting (newline/tab only, not space)

trap 'echo "FAILED at line $LINENO: $BASH_COMMAND" >&2' ERR

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"   # robust self-path

main() {
    local input="${1:?usage: $0 <input>}"    # fail fast with usage message if missing
    ...
}
main "$@"
```

`shellcheck script.sh` — the standard static analyzer, catches quoting bugs, unused
vars, unsafe globbing, etc. Run it on everything before shipping; systems teams expect this.

---

## 10. Quoting rules (precise, since this is a frequent interview gotcha)

```bash
'single quotes'      # LITERAL, no expansion of anything, not even $var or \n
"double quotes"         # expands $var, $(...), $((...)), backslash-escapes; blocks globbing
$'...'                     # ANSI-C quoting: interprets \n \t \xNN etc. as real escapes
```
```bash
echo 'price: $5'          # price: $5   (literal)
echo "price: $5"             # price: 5  (wrong! $5 = positional param, likely empty)
echo "price: \$5"               # price: $5   (escaped)
echo $'tab:\there'                 # tab: <actual tab> here
```
Always double-quote variable expansions (`"$var"`, `"${arr[@]}"`) unless you specifically
want word-splitting/globbing to occur — that single habit prevents most bash bugs.

---

## 11. Debugging toolchain around bash (systems interview territory)

```bash
strace -f -e trace=open,read,write ./script.sh   # trace every syscall a process makes
ltrace ./compiled_program                            # trace library calls
lsof -p 1234                                            # list open file descriptors of a PID
lsof -i :8080                                              # what process is using port 8080
/usr/bin/time -v ./program                                    # detailed resource usage report
perf stat ./program                                              # CPU perf counters
gdb -p 1234                                                          # attach debugger to running process
```
These aren't bash features per se, but scripting around them (parsing `strace` output,
automating `gdb` batch commands, wrapping `perf` runs) is exactly the kind of "systems
scripting" a DE Shaw-style internship tests.

---

## 12. Environment vs shell variables, and PATH mechanics

```bash
x=5                  # shell variable, visible only in THIS shell
export y=10              # environment variable, inherited by child processes

env                          # show all environment variables
printenv PATH                   # show one specific env var
echo $PATH                         # colon-separated list of directories searched for commands

PATH="/custom/bin:$PATH"              # prepend a dir (higher priority)
PATH="$PATH:/custom/bin"                 # append a dir (lower priority, fallback)

command -v gcc         # find full path bash WOULD use for `gcc` (like `which`, more POSIX)
hash -r                   # clear bash's cached PATH lookup table (after installing new binaries)
```
Child processes only inherit `export`ed variables — a plain `x=5` is invisible to anything
you `fork()`/`exec()` afterward, exactly mirroring how `environ[]` works in C.

---

## 13. Here-documents and here-strings (feeding structured input to programs)

```bash
cat <<EOF > config.txt
name: Qing
role: intern
EOF

# variable expansion still happens by default inside <<EOF
name="Qing"
cat <<EOF
Hello, $name
EOF

# quote the delimiter to DISABLE expansion (treat body as fully literal)
cat <<'EOF'
$name will not expand here
EOF

# indented heredoc (strips leading tabs, needs actual TAB chars not spaces)
cat <<-EOF
	indented line, tab stripped
EOF

# here-string: feed a single value as stdin without a temp file
grep "pattern" <<< "$myvar"
```

---

## 14. Arithmetic edge cases worth knowing

```bash
echo $((7 / 2))         # 3, integer division truncates toward zero
echo $((-7 / 2))           # -3, NOT -4 (differs from Python's floor division!)
echo $((7 % -2))              # 1, sign follows dividend, like C, unlike Python
echo $((2**62))                  # bash uses 64-bit signed integers internally
echo $((2**63))                     # overflow wraps around to negative — no auto-promotion
```
Bash arithmetic is fixed-width signed 64-bit — same overflow behavior C++ `long long`
has, no arbitrary precision. Use `bc` for big integers if you need them.

---

## 15. Testing bash scripts

```bash
bash -n script.sh          # syntax check only
shellcheck script.sh         # static analysis / lint

# bats (Bash Automated Testing System) — for actual unit tests
@test "addition works" {
    result=$(add 2 3)
    [ "$result" -eq 5 ]
}
```

---

## Summary: what a systems interview is actually likely to probe

- Redirection/FD manipulation (`exec N<>`, dup, closing descriptors)
- Subshell vs current-shell state (the pipeline gotcha in Section 3)
- Process/job control and `wait`/`$!`/`SIGCHLD`
- Signal handling and the difference between SIGTERM/SIGKILL/SIGSTOP
- `ulimit` and "too many open files" style resource-limit reasoning
- Quoting correctness and why unquoted `$var` breaks
- Fork cost / builtin-vs-external tradeoffs at scale
- Reading `strace`/`lsof`/`ps` output to diagnose a stuck or leaking process

Everything above, combined with Parts 1-2, should cover the full spread from
"replace C++ logic with bash" to "reason about bash the way an OS/systems engineer does."
