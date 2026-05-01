## Socket Programming in Python

---

### What is a Socket

A socket is an endpoint for communication between two machines (or processes on the same machine) over a network. You get a socket object, bind it to an address and port, and send/receive data through it.

Python's module is `socket` — a thin wrapper over the OS socket API.

python

```python
import socket
```

---

### Two Types of Sockets

|Type|Protocol|Nature|
|---|---|---|
|`SOCK_STREAM`|TCP|Connection-oriented — reliable, ordered|
|`SOCK_DGRAM`|UDP|Connectionless — no guarantee, no order|

The exam question asks for a **connectionless server** — that means **UDP**.

---

### TCP — Connection-Oriented

#### How TCP works

```
Server                          Client
  |                               |
bind(addr)                        |
listen()                          |
accept() ←——— connect() ←————————|
  |                               |
recv() ←————— send() ←———————————|
send() ————————————→ recv()       |
  |                               |
close()                         close()
```

#### TCP Server

python

```python
import socket

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

server.bind(("localhost", 9999))   # bind to address and port

server.listen(5)                   # start listening, max 5 queued connections
print("server listening...")

conn, addr = server.accept()       # blocks until a client connects
                                   # conn = new socket for THIS client
                                   # addr = (ip, port) of client
print(f"connected by {addr}")

data = conn.recv(1024)             # receive up to 1024 bytes
print(f"received: {data.decode()}")

conn.send("hello from server".encode())

conn.close()
server.close()
```

#### TCP Client

python

```python
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

client.connect(("localhost", 9999))   # connect to server

client.send("hello from client".encode())

data = client.recv(1024)
print(f"received: {data.decode()}")

client.close()
```

#### Key points

- `AF_INET` — IPv4 address family. Use `AF_INET6` for IPv6.
- `send()` and `recv()` work with **bytes**, not strings. Use `.encode()` to convert str → bytes and `.decode()` to convert back.
- `accept()` returns a **new socket** for each client — the original server socket keeps listening. The new `conn` socket is what you communicate through.
- `listen(5)` — the `5` is the backlog — how many connections can queue up waiting for `accept()`.

---

### Handling Multiple Clients — Threaded TCP Server

The basic server above handles one client and exits. For multiple clients, spawn a thread per connection:

python

```python
import socket
import threading

def handle_client(conn, addr):
    print(f"handling {addr}")
    while True:
        data = conn.recv(1024)
        if not data:
            break                        # client disconnected
        conn.send(data)                  # echo back
    conn.close()

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)  # reuse port immediately
server.bind(("localhost", 9999))
server.listen(5)

print("server listening...")

while True:
    conn, addr = server.accept()
    t = threading.Thread(target=handle_client, args=(conn, addr))
    t.daemon = True
    t.start()
```

`SO_REUSEADDR` lets you restart the server immediately without waiting for the OS to release the port.

---

### UDP — Connectionless

#### How UDP works

There is no connection. No handshake. You just send a datagram to an address. The other side may or may not receive it.

```
Server                        Client
  |                             |
bind(addr)                      |
recvfrom() ←—— sendto() ←——————|
sendto() ————————→ recvfrom()   |
```

No `connect()`, no `accept()`, no `listen()`. Just `bind`, `sendto`, `recvfrom`.

#### UDP Server — Connectionless Server (exam question)

python

```python
import socket

server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)   # SOCK_DGRAM for UDP

server.bind(("localhost", 9999))
print("UDP server ready...")

while True:
    data, addr = server.recvfrom(1024)    # blocks until datagram arrives
                                          # data = bytes received
                                          # addr = (ip, port) of sender
    print(f"received '{data.decode()}' from {addr}")

    server.sendto(f"echo: {data.decode()}".encode(), addr)  # send reply back to sender
```

#### UDP Client

python

```python
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

message = "hello server"
client.sendto(message.encode(), ("localhost", 9999))   # send directly to address

data, addr = client.recvfrom(1024)
print(f"received: {data.decode()}")

client.close()
```

#### What makes it connectionless

- No `connect()` on server side
- No `accept()` — no persistent connection established
- Each `recvfrom()` can receive from a **different client** — the server doesn't care who sends
- Each datagram is independent — no state between messages
- `recvfrom` returns the sender's address with every message — you use that to reply

---

### `recvfrom` vs `recv`

||`recv(bufsize)`|`recvfrom(bufsize)`|
|---|---|---|
|Returns|`bytes`|`(bytes, address)`|
|Used in|TCP|UDP|
|Knows sender|already connected|gets address per message|

### Socket Options and Utilities

python

```python
socket.gethostname()                  # machine's hostname
socket.gethostbyname("google.com")    # DNS lookup — returns IP string

server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)  # reuse address
server.settimeout(5.0)   # operations timeout after 5 seconds instead of blocking forever
```

---

### `with` Statement for Sockets

Sockets support context manager protocol — closes automatically:

python

```python
with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as server:
    server.bind(("localhost", 9999))
    data, addr = server.recvfrom(1024)
    server.sendto(data, addr)
# socket closed here automatically
```