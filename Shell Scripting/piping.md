https://medium.com/@bayounm95.eng/understanding-pipes-in-linux-b12256d4720d
https://www.geeksforgeeks.org/c/pipe-system-call/
https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_pipes.htm

In Linux, a pipe is a unidirectional IPC (Inter-Process Communication) channel backed by a ring buffer in kernel memory. It exposes two file descriptors to user space: a **read end** (`pipefd[0]`) and a **write end** (`pipefd[1]`).

## 1. Kernel Memory & Architecture

Pipes do not exist on physical storage. They are backed by `pipefs`, an in-memory pseudo-filesystem initialized during kernel boot.

- **Kernel Data Structure:** A pipe is represented internally by `struct pipe_inode_info`. This structure manages a ring buffer composed of an array of `struct pipe_buffer` objects (default size: 16 pages, or 16×4096=65,536 bytes / 64 KiB).
    
- **Ring Buffer Indexing:** Data write/read pointers are tracked using head and tail indices modulo ring capacity (`head & (ring_size - 1)`).
    
- **Atomic Writes (`PIPE_BUF`):** POSIX guarantees that writes up to `PIPE_BUF` bytes (4096 bytes on Linux) are atomic. Writes larger than 4096 bytes may be split and interleaved with writes from other processes targeting the same pipe.
    

## 2. System Calls Involved

|System Call|Prototype|Role in Piping|
|---|---|---|
|`pipe()` / `pipe2()`|`int pipe(int pipefd[2])`|Allocates a `pipe_inode_info` struct in kernel memory and returns two open file descriptors pointing to it.|
|`fork()`|`pid_t fork(void)`|Clones the parent shell process. The child inherits a copy of the parent's File Descriptor Table (FDT), pointing to the same open file description entries.|
|`dup2()` / `dup3()`|`int dup2(int oldfd, int newfd)`|Atomically duplicates `oldfd` onto `newfd`. Used to overwrite `STDIN_FILENO` (0) or `STDOUT_FILENO` (1).|
|`close()`|`int close(int fd)`|Decrements the kernel reference count for the open file description. Crucial for EOF and `SIGPIPE` propagation.|
|`execve()`|`int execve(const char *path, ...)`|Replaces the current process image with the target executable. Open file descriptors remain open across `execve()` unless `FD_CLOEXEC` is set.|
|`waitpid()`|`pid_t waitpid(pid_t pid, ...)`|Suspends shell execution until child pipeline processes terminate.|

## 3. Execution Flow of `cmd1 | cmd2`

When a shell parses `cmd1 | cmd2`, it executes the following sequence:

1

Create Anonymous Pipe via pipe()

The parent shell invokes `pipe(pipefd)`.

- `pipefd[0]` is opened with `O_RDONLY`.
    
- `pipefd[1]` is opened with `O_WRONLY`.
    

Both descriptors point to a newly allocated inode in `pipefs`.

2

Fork Child 1 (Writer Process)

The shell calls `fork()` to create Child 1 (`cmd1`). Inside Child 1:

1. Executes `dup2(pipefd[1], STDOUT_FILENO)` — maps standard output (FD 1) to the pipe's write end.
    
2. Executes `close(pipefd[0])` and `close(pipefd[1])` to free redundant descriptors.
    
3. Calls `execvp("cmd1", ...)` to replace the process image.
    

3

Fork Child 2 (Reader Process)

The shell calls `fork()` to create Child 2 (`cmd2`). Inside Child 2:

1. Executes `dup2(pipefd[0], STDIN_FILENO)` — maps standard input (FD 0) to the pipe's read end.
    
2. Executes `close(pipefd[0])` and `close(pipefd[1])` to free redundant descriptors.
    
3. Calls `execvp("cmd2", ...)` to replace the process image.
    

4

Parent Cleanup and Reference Decrement

The parent shell executes `close(pipefd[0])` and `close(pipefd[1])`.

**Why this is mandatory:** The kernel open file description maintains a reference counter. If the parent shell leaves `pipefd[1]` open, the write reference count will never reach `0` when Child 1 terminates. As a result, Child 2's `read()` on FD 0 will block indefinitely waiting for data instead of receiving `EOF` (0 bytes read).

5

Process Synchronization via waitpid()

The shell calls `waitpid()` on both child PIDs to reap zombies and collect exit statuses before displaying the next command prompt.

## 4. Reference Implementation in C

The following code demonstrates explicit file descriptor manipulation for `ls -l | grep main`:

C

```c
#include <unistd.h>
#include <sys/wait.h>
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int pipefd[2];
    if (pipe(pipefd) == -1) {
        perror("pipe failed");
        exit(EXIT_FAILURE);
    }

    // --- Child 1: Writer (ls -l) ---
    pid_t pid1 = fork();
    if (pid1 == 0) {
        // Redirect STDOUT (1) to write end of pipe
        dup2(pipefd[1], STDOUT_FILENO);
        
        // Close unused descriptors
        close(pipefd[0]);
        close(pipefd[1]);

        char *args[] = {"ls", "-l", NULL};
        execvp(args[0], args);
        perror("execvp ls failed");
        exit(EXIT_FAILURE);
    }

    // --- Child 2: Reader (grep main) ---
    pid_t pid2 = fork();
    if (pid2 == 0) {
        // Redirect STDIN (0) to read end of pipe
        dup2(pipefd[0], STDIN_FILENO);
        
        // Close unused descriptors
        close(pipefd[0]);
        close(pipefd[1]);

        char *args[] = {"grep", "main", NULL};
        execvp(args[0], args);
        perror("execvp grep failed");
        exit(EXIT_FAILURE);
    }

    // --- Parent (Shell) Cleanup ---
    // Close parent's copies of descriptors so EOF condition can trigger
    close(pipefd[0]);
    close(pipefd[1]);

    // Wait for both processes to complete
    waitpid(pid1, NULL, 0);
    waitpid(pid2, NULL, 0);

    return 0;
}
```

## 5. Kernel Blocking Semantics and Signals

### Reader Semantics (`read()` on `pipefd[0]`)

- **Buffer empty, writers present:** `read()` blocks in `TASK_INTERRUPTIBLE` state until data is written or a signal interrupts it.
    
- **Buffer empty, all writers closed:** `read()` returns `0` immediately (`EOF`).
    

### Writer Semantics (`write()` on `pipefd[1]`)

- **Buffer full ( capacity ≥64 KiB ):** `write()` blocks until the reader process consumes data and clears buffer space.
    
- **All readers closed:** If a process attempts to `write()` to a pipe with no active read descriptors open:
    
    1. The kernel sends `SIGPIPE` (Signal 13) to the writing process.
        
    2. If `SIGPIPE` is ignored or caught, the `write()` system call fails with `errno = EPIPE` (Broken pipe).