### 1. Parsing & interpretation (it's a programming language)

Bash isn't just splitting text on spaces and exec'ing it. It's parsing a real grammar:

- Variables, assignment, expansion: `x=5; echo $x`, `${x:-default}`
- Control flow: `if`/`while`/`for`/`case`, functions
- Arithmetic: `$((x + 1))`
- Quoting rules (single vs double vs backtick — each behaves differently for expansion)
- Arrays, associative arrays (bash-specific, not POSIX)

This is why you can write actual `.sh` scripts that are full programs, not just command lists.

### 2. Expansion (rewriting your input before running anything)

Before bash executes a line, it runs it through several expansion passes, in a specific order:

- **Brace expansion**: `file{1,2,3}.txt` → `file1.txt file2.txt file3.txt`
- **Tilde expansion**: `~` → `/home/qing`
- **Parameter/variable expansion**: `$HOME`, `${arr[2]}`
- **Command substitution**: `$(date)` — runs `date`, substitutes its output
- **Arithmetic expansion**: `$((2+2))`
- **Word splitting**: unquoted expansion results get split on `$IFS`
- **Pathname expansion (globbing)**: `*.txt` → actual matching filenames

None of this is "running a command" — it's text transformation the shell does on your behalf before anything is exec'd. This is also the source of most shell footguns (unquoted `$var` splitting unexpectedly, etc.).

### 3. Process management (this is the "OS-facing" job)

When it does run something, there's real work involved:

- `fork()`s a child process
- Sets up file descriptors for that child (this is where redirection happens: `>`, `<`, `2>&1`)
- Builds pipelines: `cmd1 | cmd2 | cmd3` — bash creates pipes, forks 3 children, wires their stdin/stdout together
- `exec()`s the actual program in the child, replacing the child's memory image
- Waits for it, collects its exit status (`$?`)

```
you type:  cat file.txt | grep foo | wc -l

bash:
  1. fork() → child A → exec("cat", ["file.txt"])
  2. fork() → child B → exec("grep", ["foo"])
  3. fork() → child C → exec("wc", ["-l"])
  4. wires: A's stdout --(pipe)--> B's stdin
            B's stdout --(pipe)--> C's stdin
  5. bash itself doesn't run cat/grep/wc — it just
     sets up the plumbing and gets out of the way
```

- **Built-in commands** are the exception — `cd`, `export`, `echo`, `alias`, `source`, `jobs` don't fork a new process at all; bash runs them _in itself_. (`cd` _has_ to be a builtin — a child process changing its own working directory would have zero effect on the parent shell.)

### 4. Job control

Tracking which processes are in the foreground vs background, process groups, and translating terminal signals into the right target:

- `&` → background a job
- `fg`/`bg`/`jobs` → manage them
- Ctrl+Z → suspend foreground job (`SIGTSTP`)
- Ctrl+C → kill foreground job (`SIGINT`)
- Deciding which process group currently "owns" the terminal (`tcsetpgrp`)

### 5. Environment & inheritance

- Maintains the environment variable table, exports it to children (`export FOO=bar` — children inherit `FOO`, un-exported vars don't)
- `PATH` resolution — searching directories to find `ls` when you type `ls`
- Shell functions and aliases — user-defined shortcuts that live only in that shell session

### 6. Scripting features beyond one-liners

- Functions, local variables, return values via exit codes
- Traps: `trap 'cleanup' EXIT` — run code on signals or shell exit
- Here-documents (`<<EOF`) and here-strings (`<<<`)
- Subshells `( ... )` vs current-shell grouping `{ ... }`
- Exit status chaining: `&&`, `||`

### 7. The interactive UX layer (via readline)

History, tab completion, line editing, `PS1` prompt customization/escapes, completion scripts (`complete -F _git_completion git`).