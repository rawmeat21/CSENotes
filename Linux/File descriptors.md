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
https://jzhao.xyz/thoughts/file-descriptor

![[Pasted image 20260919091151.png]]
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


```c
int fd0 = open("/home/joe_user/my_file.txt", O_RDWR|O_CREAT, S_IRUSR|S_IWUSR|S_IRGRP);
```

If we assume that a process has no files open yet, calling `open()` on `/home/joe_user/my_file.txt` creates the following hierarchy of these data stuctures.

![[Pasted image 20260919015435.png]]

The file buffer is represented by _File buffer no. 0_ in our example. It’s the raw data stored in the regular file.

The kernel creates a new open file description, labeled _Open file description no. 0._ in the example. _The open file description_ stores all state related to the opened file. That is the file-access flags supplied with `open()` and the byte offset where the next read or write operation takes place.

Each process maintains a so-called _file-descriptor table._ As the name suggests, it’s a table of all file descriptors of the process. _A file descriptor_ is a process’ means of refering to an open file description. Each file descriptor refers to an open file description, which in turn refers to a file buffer.

The kernel always allocates the lowest available entry in the file-descriptor table and associates it with the open file description it just created. In the case of our invocation of `open()` this is entry _0._

Open it a second time:

```c
int fd1 = open("/home/joe_user/my_file.txt", O_RDWR|O_CREAT, S_IRUSR|S_IWUSR|S_IRGRP);
```

![[Pasted image 20260919015717.png]]

A new entry in the file-descriptor table is allocated and its index is returned to the user-space application.

Our application now has two file descriptors that refer to the same file buffer, but these file descriptors don’t share any state besides the file buffer’s content. So writing with `fd1` starts at byte offset 0! Doing

```c
write(fd1, "Salut monde!", strlen("Salut monde!"));
```

effectively overwrites the output we already wrote using `fd0`. The file buffer now contains _Salut monde!_ and the byte offset of _Open file description no. 1_ is at position 12.

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


### Duplicating File Descriptors

```c
int fd2 = dup(fd1);
```

![[Pasted image 20260919020007.png]]

Let’s write using both, `fd1` and `fd2`.

```c
write(fd2, " Ca", strlen(" Ca")); // write 3 byte
write(fd1, " va?", strlen(" va?")); // write another 4 byte
```

**Both file descriptors share the same open file description, writing with either therefore modifies the same byte offset.**


### Closing File Descriptors

```c
close(fd1);
```

![[Pasted image 20260919020230.png]]

Also:

```c
close(fd0);
```

![[Pasted image 20260919020310.png]]


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

When you fork a server process, both parent and child inherit the listening socket:

```c
int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
// ... bind and listen setup ...

while (1) {
    int client_fd = accept(listen_fd, NULL, NULL);
    
    pid_t pid = fork();
    if (pid == 0) {
        // Child handles this client
        close(listen_fd);    // Child doesn't need to accept new connections
        handle_client(client_fd);
        close(client_fd);
        exit(0);
    } else {
        // Parent continues accepting
        close(client_fd);    // Parent doesn't need this client connection
    }
}
```

## Race Conditions and Shared File Access

Shared file offsets can lead to race conditions when multiple processes access the same file simultaneously.

```c
void log_message(int fd, const char* msg) {
    lseek(fd, 0, SEEK_END);  // Seek to end
    write(fd, msg, strlen(msg));  // Write message
    write(fd, "\n", 1);
}
```

For more cases: https://chessman7.substack.com/p/fork-and-file-descriptors-the-unix



## IO redirection

https://devconnected.com/input-output-redirection-on-linux-explained/

On Linux **[everything is a file](https://opensource.com/life/15/9/everything-is-a-file)**.

It means that processes, devices, keyboards, hard drives are represented as files living on the filesystem.

The Linux Kernel may differentiate those files by assigning them a **file type** (a file, a directory, [a soft link](https://devconnected.com/understanding-hard-and-soft-links-on-linux/) or a socket for example) but they are stored in the same data structure by the Kernel.

![[Pasted image 20260919093630.png]]

For every process created, a new **task_struct** is created on your Linux host.

This structure holds two references, one for filesystem metadata (called **fs**) where you can find information such as the filesystem mask for example.

The other one is a structure for files holding what we call **file descriptors**.


The Kernel is able to understand that you want to transfer some files between disks, or that you may want to create a new video on your secondary drive for example.

As a consequence, the Linux Kernel is permanently moving data from input devices (a keyboard for example) to output devices (a hard drive for example).

Using this abstraction, **processes are essentially a way to manipulate inputs** (as **read** operations) to render various outputs (as **write** operations).

Processes know where data should be sent to using **file descriptors.**

On Linux, **the file descriptor 0 (or `fd[0]`)** is assigned to the **standard input.**

Similarly **the file descriptor 1 (or `fd[1]`)** is assigned to the **standard output**, and the **file descriptor 2 (or `fd[2]`)** is assigned to **the standard error.**

![[Pasted image 20260919093946.png]]

On a Linux system, for every process, the first three file descriptors are reserved for standard inputs, outputs and errors.

**Those file descriptors are mapped to devices on your Linux system.**

Devices registered when the kernel was instantiated, they can be seen in the **/dev** directory of your host:

![[Pasted image 20260919094123.png]]

If you were to take a look at the file descriptors of a given process, let’s say a bash process for example, you can see that **file descriptors are essentially soft links to real hardware devices on your host:**

![[Pasted image 20260919094150.png]]

In this case, **/dev/pts/0** represents a terminal which is a virtual device (or tty) on my virtual filesystem. In simpler terms, it means that my bash instance (running in a Gnome terminal interface) waits for inputs from my keyboard, prints them to the screen, and executes them when asked to.

### Output redirection

Input and output redirection is a technique used in order to **redirect/change** standard inputs and outputs, essentially changing where data is read from, or where data is written to.

For example, if I execute a command on my Linux shell, the output might be printed directly to my terminal (a cat command for example).

However, with output redirection, I could choose to store the output of my cat command to a file for long-term storage.

**Output redirection is the act of redirecting the output of a process to a chosen place like files, databases, terminals or any devices (or virtual devices) that can be written to.**

![[Pasted image 20260919094445.png]]

As you know, the `>` operator must be used for this.

IMP note: What happens if you do this?

```bash
echo 'This a cool butterfly' > file
sed 's/butterfly/parrot/g' file > file
```

`file` is empty. 

By default, when parsing your command, the kernel will not execute the commands sequentially.

It means that it won’t wait for the end of the sed command to open your new file and to write the content to it.

**Instead, the kernel is going to open your file, erase all the content inside it, and wait for the result of your sed operation to be processed.**

As you sed operation is seeing an empty file (because all the content was erased by the output redirection operation), the content is empty.

In order to redirect the output to the same file, you may want to use **pipes** or more [advanced commands](https://unix.stackexchange.com/questions/159513/what-are-the-shells-control-and-redirection-operators#186126) such as

```bash
command … input_file > temp_file  &&  mv temp_file input_file
```

### Input redirection

**Input redirection is the act of redirecting the input of a process to a given device (or virtual device) so that it starts reading from this device and not from the default one assigned by the Kernel.**


![[Pasted image 20260919094822.png]]

As an example, when you are opening a terminal, you are interacting with it with your keyboard.

However, there are some cases where you might want to work with the content of a file, because you want to programmatically send the content of the file to your command.

**To redirect the standard input on Linux, you have to use the “<” operator.**

### Error redirection

**Error redirection is redirecting errors returned by processes to a defined device on your host.**

![[Pasted image 20260919095022.png]]

To redirect error output on Linux, use the “**2>**” operator

```
$ command 2> file
```

### Pipes

**With pipelines, you are not overwriting inputs or outputs, but you are connecting them together.**

Pipelines are used on Linux systems to connect processes together, linking standard outputs from one program to the standard input of another.

Multiple processes can be linked together with **pipelines** (or **pipes**)

![[Pasted image 20260919095143.png]]

```bash
$ grep '.com' file | wc -l
```

![[Pasted image 20260919095212.png]]

