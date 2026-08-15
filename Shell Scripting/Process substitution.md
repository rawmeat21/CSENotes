Process substitution (`<(cmd)` and `>(cmd)`) allows the standard input or output of a process to be exposed as a virtual file path (typically `/dev/fd/N`), enabling commands that expect file paths to read from or write to asynchronous command pipelines.

## Internal Mechanism

When Bash parses a process substitution construct:

1. **Pipe Creation & Subshell Fork:** Bash invokes the `pipe()` system call to create an anonymous pipe, then executes `fork()` to run the inner command in a child subshell.
    
2. **File Descriptor Binding:**
    
    - For **Input substitution** `<(cmd)`: The child subshell's `stdout` (FD 1) is bound to the write end of the pipe.
        
    - For **Output substitution** `>(cmd)`: The child subshell's `stdin` (FD 0) is bound to the read end of the pipe.
        
3. **Argument Substitution:** Bash replaces the `<(cmd)` syntax on the command line with a path string pointing to the corresponding open file descriptor—typically `/dev/fd/<N>` on Linux (resolved via `/proc/self/fd/<N>`).
    
4. **FIFO Fallback:** On systems lacking dynamic `/dev/fd/` entries, Bash creates a named pipe in `/tmp` via `mkfifo()`, passes the FIFO path as the argument, and unlinks the file entry after opening.
    
5. **Execution & Cleanup:** The main process opens and processes `/dev/fd/<N>` using standard file I/O operations (`open()`, `read()`, `write()`, `close()`).
    

## Key Usage Patterns

### 1. Commands Expecting Multiple File Arguments

Commands such as `diff`, `comm`, or `paste` accept file path arguments rather than multiple standard input streams.

Bash

```bash
# Compare the sorted outputs of two commands without writing to disk
diff -u <(sort serverA.log) <(sort serverB.log)
```

Think that `<(sort serverB.log)` and `<(sort serverA.log)` output **2 files**, which are passed as input to `diff`.

### 2. Preserving Variable Scope in Loops

Piping data to a loop (`cmd | while ...`) executes the loop inside a subshell, causing any state changes to variables inside the loop to be lost upon exit. Feeding the loop via process substitution executes the loop in the current shell environment.

Bash

```bash
count=0

# Loop runs in the current shell context
while read -r line; do
    ((count++))
done < <(grep "ERROR" system.log) <--- Think that <(grep "ERROR" system.log) produces a file which is passed to the while loop using < (redirection)

echo "Total errors: $count" # Variable value is preserved
```

### 3. Multiplexing Output Streams

Process substitution can be combined with `tee` to redirect a single stream into multiple child processes concurrently.

Bash

```bash
# Process a single log stream into multiple filtered output targets
cat app.log | tee >(grep "ERROR" > errors.log) >(grep "WARN" > warnings.log) > /dev/null
```

`tee` is designed to take file paths directly as CLI arguments:

```bash
tee [OPTION]... [FILE]... (note: multiple file arguments are allowed)
```

So technically we are passing in a file when we write `>(grep "ERROR" > errors.log)`.
Also, `tee` passes the output to `/dev/null`, which is basically what `tee` is used for.

## Process Substitution vs Piping

| Feature               | Standard Piping (`cmd1 \| cmd2`)                 | Process Substitution (`cmd2 <(cmd1)`)          |
| --------------------- | ------------------------------------------------ | ---------------------------------------------- |
| **Data Interface**    | Connects directly to process `stdin` (FD 0)      | Passed as a file path argument (`/dev/fd/N`)   |
| **Stream Capacity**   | 1 input stream                                   | Arbitrary number of concurrent inputs          |
| **Execution Context** | Receiver executes in subshell (in standard Bash) | Receiver executes in current shell environment |
| **Seekability**       | Stream-only (unseekable)                         | Stream-only (fails `lseek()` syscalls)         |

Read more: https://oneuptime.com/blog/post/2026-01-24-bash-process-substitution/view
