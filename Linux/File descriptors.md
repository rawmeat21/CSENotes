https://en.wikipedia.org/wiki/File_descriptor
https://bottomupcs.com/ch01s03.html
https://medium.com/@tharinduimalka915/linux-file-descriptors-ec945fd36893
https://dev.to/sebastianmarines/understanding-linuxs-file-descriptors-a-deep-dive-into-21-and-redirection-4g5h
https://devopswizard.hashnode.dev/understanding-and-monitoring-file-descriptors-in-linux-a-complete-guide
https://biriukov.dev/docs/fd-pipe-session-terminal/1-file-descriptor-and-open-file-description/
https://en.rattibha.com/thread/1553033011016306689
https://linuxmeerkat.wordpress.com/2011/12/02/file-descriptors-explained/
https://www.youtube.com/watch?v=crd2hgzBVTo&pp=ygUYZmlsZSBkZXNjcmlwdG9yIGluIGxpbnV4
https://www.youtube.com/watch?v=rW_NV6rf0rM&pp=ygUYZmlsZSBkZXNjcmlwdG9yIGluIGxpbnV4

https://chessman7.substack.com/p/fork-and-file-descriptors-the-unix
https://tzimmermann.org/2017/07/28/data-structures-of-unix-file-io/
## The Fork System Call and Process Creation

The `fork()` system call creates a new process by duplicating the calling process. Unlike creating a process from scratch, fork produces an exact copy of the parent's address space, including all variables, heap data, and stack contents. The only immediate difference between parent and child is the return value of fork itself: the parent receives the child's process ID, while the child receives zero.

![[Pasted image 20260918155144.png]]

But process duplication goes beyond just memory. 

The kernel also duplicates the process control block, which contains metadata about the process including its file descriptor table. 

This table maps file descriptor numbers (like 0, 1, 2 for stdin, stdout, stderr) to actual file structures in the kernel. While the file descriptor table is duplicated, the underlying file structures are shared.

## File Descriptors and the Kernel's File Management

![[Pasted image 20260918155355.png]]


**File Descriptor Table**: Each process has its own table mapping small integers (file descriptors) to entries in the system-wide open file table. When you call `open()`, you get back one of these integers.

**Open File Table**: This system-wide table contains entries for each open file, regardless of which process opened it. Each entry tracks the current file offset, access mode (read/write), and status flags.

**Inode Table**: The actual file metadata lives here - permissions, timestamps, disk block locations, and other filesystem-specific information.

When you read from a file using a file descriptor, the kernel follows this chain: your file descriptor points to an open file table entry, which contains the current offset and points to the inode representing the actual file.

## How Fork Handles Open Files

When fork creates a child process, it duplicates the parent's file descriptor table. Each file descriptor in the child points to the same open file table entry as the corresponding descriptor in the parent.

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    int fd = open("test.txt", O_RDWR | O_CREAT, 0644);
    write(fd, "Hello from parent\n", 18);
    
    pid_t pid = fork();
    
    if (pid == 0) {
        // Child process
        write(fd, "Hello from child\n", 17);
    } else {
        // Parent process
        wait(NULL);  // Wait for child to complete
        write(fd, "Parent again\n", 13);
    }
    
    close(fd);
    return 0;
}
```

## Shared File Offsets

When both processes read or write to the same file descriptor, they're manipulating the same offset counter in the kernel's open file table.

Consider what happens when both parent and child try to read from the same file:

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    int fd = open("input.txt", O_RDONLY);
    char buffer[100];
    
    pid_t pid = fork();
    
    if (pid == 0) {
        // Child reads first
        int bytes = read(fd, buffer, 50);
        printf("Child read: %.*s\n", bytes, buffer);
    } else {
        sleep(1);  // Let child read first
        // Parent reads from where child left off
        int bytes = read(fd, buffer, 50);
        printf("Parent read: %.*s\n", bytes, buffer);
    }
    
    close(fd);
    return 0;
}
```

The child reads the first 50 bytes, advancing the shared file offset. When the parent reads, it starts from byte 51, not from the beginning of the file.

## File Descriptor Independence vs. File Description Sharing

While file descriptors are independent between parent and child, the underlying file descriptions are shared. This means:

- **Closing a file descriptor** in one process doesn't affect the other process's ability to use its copy of that descriptor
    
- **File operations** (read, write, lseek) affect the shared file offset visible to both processes
    
- **File status flags** set with fcntl affect both processes since they're stored in the shared open file description

Here's a practical example showing descriptor independence:

```c
int fd = open("test.txt", O_RDWR);
pid_t pid = fork();

if (pid == 0) {
    close(fd);  // Child closes its descriptor
    // fd is now invalid in child, but parent can still use it
    exit(0);
} else {
    wait(NULL);
    write(fd, "Parent can still write\n", 23);  // This works fine
    close(fd);
}
```

The child's close operation only affects its own file descriptor table entry. The parent's descriptor remains valid because the kernel maintains a reference count on open file descriptions. Only when all file descriptors pointing to an open file description are closed does the kernel actually close the file.


## Low-Level File Descriptor Mechanics

When fork duplicates the file descriptor table, it's essentially calling `dup()` on every open file descriptor. 

The `dup()`system call creates a new file descriptor entry that points to the same open file description as the original.

This equivalence means you can simulate fork's file handling behavior manually:

```c
int original_fd = open("test.txt", O_RDWR);
int duplicated_fd = dup(original_fd);

// Now both descriptors share the same file offset
write(original_fd, "First write\n", 12);
write(duplicated_fd, "Second write\n", 13);  // Starts where first ended
```


## Standard File Descriptors and Redirection

Fork's file descriptor inheritance is particularly important for standard streams (stdin, stdout, stderr). 

Shell redirection works by manipulating these descriptors before calling fork and exec:

```c
int fd = open("output.log", O_WRONLY | O_CREAT | O_TRUNC, 0644);
pid_t pid = fork();

if (pid == 0) {
    // Redirect child's stdout to the log file
    dup2(fd, STDOUT_FILENO);
    close(fd);  // Close original descriptor
    
    printf("This goes to the log file\n");
    exit(0);
} else {
    close(fd);  // Parent doesn't need this descriptor
    printf("This goes to the terminal\n");
    wait(NULL);
}
```
The `dup2()` call makes `STDOUT_FILENO` (descriptor 1) point to the same open file description as `fd`. After this, all printf output in the child goes to the log file, while the parent's output remains on the terminal.


## Network Sockets and Fork


