https://medium.com/@onix_react/what-are-sockets-and-what-are-sockets-for-8eef56436b7b

linux way:
https://medium.com/@ahnafzamil/networking-101-what-are-sockets-7e2d274e153

how it works:

## 1. Operating System Implementation & Kernel Primitives

When you create a socket, the kernel allocates data structures to track the state, address bindings, routing information, and packet queues for that communication endpoint:

```
User Space               Kernel Space
+------------------+     +-------------------------------------------------+
| Application Code |     | File Descriptor Table                           |
|                  |     |   FD [3] ---------> struct file                 |
|  sendto() / recv()| <==>|                       |                         |
+------------------+     |                       v                         |
                         |                 struct socket                   |
                         |                       |                         |
                         |                 struct sock (inet_sock / udp_sock)|
                         |                       |                         |
                         |                 sk_receive_queue / sk_write_queue|
                         +-------------------------------------------------+
                                                 |
                                                 v
                                        Network Device (NIC)
```

**File Descriptor Table Mapping:** A socket returns an integer File Descriptor (e.g., `FD = 3`). The process file table points this FD to an internal `struct file`, where the file operations (`f_ops`) redirect standard I/O syscalls (`read`, `write`, `close`) to socket-specific handlers (`socket_file_ops`).

**`struct socket` & `struct sock`:**

- `struct socket` provides the BSD socket API interface abstraction.
    
- `struct sock` (specifically protocol-dependent implementations like `struct udp_sock` or `struct tcp_sock`) manages lower-level network state, memory allocation, and incoming/outgoing packet queues (`sk_buff` queues).


## 2. Core Concepts: Domains, Types, and Addressing

To establish a socket, you specify three primary properties:

### A. Address Family (Domain)

Defines the protocol family and network address format used to locate endpoints.

- **`AF_INET`:** IPv4 internet protocols (32-bit addresses + 16-bit ports).
    
- **`AF_INET6`:** IPv6 internet protocols (128-bit addresses + 16-bit ports).
    
- **`AF_UNIX` / `AF_LOCAL`:** Local Inter-Process Communication (IPC) via Unix domain sockets using filesystem pathnames (bypasses the TCP/IP stack).
    
- **`AF_PACKET`:** Low-level packet interface bypasses transport and network layers, allowing direct raw framing over the Data Link Layer (Ethernet frames).
    

### B. Socket Type

Defines the semantics of data transfer at the Transport Layer (L4).

- **`SOCK_DGRAM` (Datagram Sockets / UDP):** Connectionless, unreliable, unordered messages of fixed maximum length. Preserves message boundaries.
    
- **`SOCK_STREAM` (Stream Sockets / TCP):** Connection-oriented, sequenced, reliable, full-duplex byte stream. Does not preserve message boundaries (requires application-layer framing).
    
- **`SOCK_RAW`:** Direct access to IP layer or network protocols, bypassing protocol-specific handling (used for ICMP/ping or custom protocols).


### C. Network Byte Order

Network protocols strictly enforce **Big-Endian** byte ordering (Most Significant Byte first). Host architectures (such as x86_64) typically use **Little-Endian**.

- `htons()` / `htonl()`: Host-To-Network Short (16-bit) / Long (32-bit). Converts integer fields (like port numbers) to Big-Endian.
    
- `ntohs()` / `ntohl()`: Network-To-Host Short / Long.
    
- `inet_pton()` / `inet_ntop`: Converts IP addresses between human-readable ASCII presentation strings (`"192.168.1.1"`) and numeric binary network format.

## 3. Syscall Execution Flow Comparison

### UDP (Connectionless Architecture)

Used in your code example. No handshake is performed.

```
          SENDER (Client)                              RECEIVER (Server)
      +----------------------+                      +--------------------+
      |  socket(AF_INET,     |                      |  socket(AF_INET,   |
      |         SOCK_DGRAM)  |                      |         SOCK_DGRAM)|
      +----------+-----------+                      +---------+----------+
                 |                                            |
                 |                                  +---------v----------+
                 |                                  |  bind(local_addr)  |
                 |                                  +---------+----------+
                 |                                            |
      +----------v-----------+                      +---------v----------+
      |  sendto(dest_addr)   | ===== Datagram ====> |  recvfrom()        |
      +----------+-----------+                      +---------+----------+
                 |                                            |
      +----------v-----------+                      +---------v----------+
      |  close()             |                      |  close()           |
      +----------------------+                      +--------------------+
```

### TCP (Connection-Oriented Architecture)

Requires a 3-Way Handshake (`SYN`, `SYN-ACK`, `ACK`) before data transmission.

```
          CLIENT                                       SERVER
      +----------------------+                      +--------------------+
      | socket(SOCK_STREAM)  |                      | socket(SOCK_STREAM)|
      +----------+-----------+                      +---------+----------+
                 |                                            |
                 |                                  +---------v----------+
                 |                                  |  bind(local_addr)  |
                 |                                  +---------+----------+
                 |                                            |
                 |                                  +---------v----------+
                 |                                  |  listen(backlog)   |
                 |                                  +---------+----------+
                 |                                            |
      +----------v-----------+                      +---------v----------+
      | connect(remote_addr) | ===== 3-Way =======> | accept()           |
      |                      | <==== Handshake ===> | (blocks until conn)|
      +----------+-----------+                      +---------+----------+
                 |                                            |
      +----------v-----------+                      +---------v----------+
      | send() / write()     | ===================> | recv() / read()    |
      +----------+-----------+                      +---------+----------+
```


From Assignment 1:


