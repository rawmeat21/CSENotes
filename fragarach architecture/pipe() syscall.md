The pipe() system call in operating systems facilitates interprocess communication by creating a unidirectional communication channel between two processes. It allows one process to write data into the pipe, while another process can read from it.

When you create an unnamed pipe using `pipe()`, the kernel allocates a memory buffer that temporarily stores data transmitted between processes. This buffer has a system-defined fixed size.

The `pipe()` function fills an array with **two file descriptors**:

- `pipefd[0]` — the **read end of the pipe**
- `pipefd[1]` — the **write end of the pipe**

Some important behaviors to note:

- If the **buffer becomes full**, a write operation will block until space becomes available.
- If the **buffer is empty**, a read operation will block until data is written.
- Once communication is complete, each process must close the file descriptors it no longer needs. When all ends are closed, the kernel deallocates the pipe’s buffer.

![[Pasted image 20260719181629.png]]

![[Pasted image 20260719181712.png]]


pipe uses message passing.