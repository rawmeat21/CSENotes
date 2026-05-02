## WebSockets

### What is a Network Socket — the Foundation

Before WebSockets, you need to understand what a **socket** is, because the name "WebSocket" comes from this.

A socket is a software construct — not hardware. It is an endpoint for network communication, represented in your operating system as a file descriptor (an integer handle). When two programs communicate over a network, each program has a socket. Data flows between the two sockets.

A socket is created by the operating system kernel. Your program asks the OS: "give me a socket." The OS gives back an integer — say `5` — and internally maintains a data structure associated with that number that tracks the connection state, send/receive buffers, remote address, etc. Your program uses that integer to tell the OS "send this data on socket 5" or "read data from socket 5."

This is why sockets are described as software — they are OS abstractions over the underlying network hardware (your NIC — Network Interface Card). Your program never touches hardware directly. It talks to the OS, which talks to the hardware.

```
Your Python program
        │
        │  socket(AF_INET, SOCK_STREAM)  ← system call
        ▼
   OS Kernel
        │
        │  manages TCP state, buffers, retransmission
        ▼
   Network Interface Card (hardware)
        │
        ▼
   Physical network (cables, switches, routers)
```

---

### What is an IP Address

Every device on a network has an **IP address** — a numerical identifier assigned to its network interface. IPv4 addresses are 32-bit integers written as four decimal numbers separated by dots: `192.168.1.10`, `142.250.183.46`.

When you send data over the internet, the IP address tells the network _which machine_ to deliver it to. Routers read the destination IP and forward packets hop-by-hop until they reach the right machine.

---

### What are Ports — Precisely

An IP address identifies a machine. But a machine runs many programs simultaneously — a web server, a database, an SSH server, your Python script. When data arrives at the machine, the OS needs to know _which program_ to deliver it to.

**A port is a 16-bit unsigned integer (0–65535) that identifies a specific process or service on a machine.**

When your program creates a socket and wants to receive incoming connections, it **binds** the socket to a port number. The OS then knows: "any data arriving at this machine destined for port 8080 should go to this program's socket."

A **connection** is identified by a 4-tuple:

```
(source IP, source port, destination IP, destination port)
```

This 4-tuple uniquely identifies every connection on the internet. When a response comes back to your machine, the OS looks at the destination port and routes it to the right socket.

#### Port ranges

|Range|Name|Description|
|---|---|---|
|0–1023|Well-known ports|Reserved for standard services. Binding requires root/admin privileges.|
|1024–49151|Registered ports|Assigned to specific applications by convention|
|49152–65535|Ephemeral ports|Assigned temporarily by the OS to client sockets|

#### Well-known port assignments

|Port|Protocol|
|---|---|
|80|HTTP|
|443|HTTPS|
|22|SSH|
|5432|PostgreSQL|
|6379|Redis|
|8080|Common HTTP alternative|

When you visit `https://codeforces.com`, your browser connects to port 443 on Codeforces's server. When your code hits `https://leetcode.com/graphql`, it connects to port 443. The `:443` is implicit in `https://` URLs — browsers and HTTP libraries know to use it by default.

#### Ephemeral ports — client side

When your Python program makes an outgoing HTTP request, it needs a socket too. The OS automatically assigns it a source port from the ephemeral range — say `54321`. The server's response comes back to `your-ip:54321`, and the OS routes it to your program's socket.

You never choose this port — the OS picks it. This is why you only specify the destination port when making requests.

#### What "listening on a port" means

A server program calls `bind(port)` to claim a port, then `listen()` to tell the OS it's ready to accept incoming connections. The OS will queue incoming connections and hand them to the server one by one via `accept()`. When someone says "the server is listening on port 8000," they mean exactly this — the program has bound that port and is waiting for connections.

python

```python
# This is what happens internally when you run a server
# (Python's socket module — low level, for illustration)
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind(("0.0.0.0", 8000))   # claim port 8000
server_socket.listen(5)                  # ready to accept, queue up to 5

conn, addr = server_socket.accept()      # blocks until a client connects
data = conn.recv(1024)                   # read data from client
conn.send(b"HTTP/1.1 200 OK\r\n\r\n")   # send response
```

FastAPI, Flask, and every other web framework do this internally. You never write this yourself — the framework handles it. But this is what's happening under the hood when you run `uvicorn main:app --port 8000`.

---

### TCP — What Underlies HTTP and WebSockets

Both HTTP and WebSockets run on top of **TCP** (Transmission Control Protocol). You need to understand TCP to understand why WebSockets were invented.

TCP is a **connection-oriented, reliable, ordered** protocol:

- **Connection-oriented** — before any data is exchanged, a connection must be established via a 3-way handshake (SYN → SYN-ACK → ACK). This takes one round trip.
- **Reliable** — every packet sent is acknowledged. If no acknowledgment arrives, the packet is retransmitted. Data always arrives or the connection breaks — it never silently disappears.
- **Ordered** — data arrives in the same order it was sent, even if packets take different network paths.

TCP gives you a **stream** — you write bytes into it, the other end reads bytes out, in order, reliably. HTTP and WebSockets both use this stream to frame their own messages on top.

---

### HTTP Request-Response — The Full Picture

Now you can understand HTTP fully. An HTTP/1.1 interaction over TCP looks like this:

```
Client                          Server
  │                               │
  │──── TCP SYN ──────────────────▶│
  │◀─── TCP SYN-ACK ──────────────│   TCP handshake (one round trip)
  │──── TCP ACK ──────────────────▶│
  │                               │
  │──── HTTP GET /api/data ───────▶│   request sent over TCP stream
  │◀─── HTTP 200 OK + body ────────│   response received
  │                               │
  │──── TCP FIN ──────────────────▶│   connection closed
  │◀─── TCP FIN-ACK ──────────────│
```

Key characteristics:

- **Client initiates.** The server cannot send data unless the client first sends a request.
- **One request, one response.** The server sends exactly one response per request, then waits for the next request.
- **Stateless.** The server does not maintain state between requests (hence cookies for session management).
- **Half-duplex by default.** At any moment, data flows in one direction — client sends request, then server sends response.

This model works perfectly for: fetch a webpage, fetch JSON from an API, submit a form. These are all pull operations — the client asks, the server answers.

It breaks down for: real-time updates, push notifications, live data streams. If your server has new data for the client (a new solve detection, a price update, a chat message), it cannot send it until the client asks. The client has to poll — ask repeatedly: "do you have new data? do you have new data? do you have new data?"

---

### HTTP Polling — What Your Code Currently Does

Your `SolveTracker` works by polling. Every `CHECK_INTERVAL` seconds, the background thread makes fresh HTTP requests to Codeforces/LC/AtCoder and checks if any new problems were solved:

python

```python
# _loop, lines 820-828
while not self._stop.is_set():
    slept = 0
    while slept < CHECK_INTERVAL and not self._stop.is_set():
        self._stop.wait(1)
        slept += 1
    if not self._stop.is_set():
        self._do_check()    # make HTTP requests, check solve status
```

And the main display loop polls `snapshot()` every second to get the latest state:

python

```python
# run(), lines 872-873
while True:
    status, solve_times, last_check = tracker.snapshot()
    ...
    time.sleep(1)
```

This is **polling** — periodically asking "has anything changed?" The problem:

- You're making network requests even when nothing has changed
- There's a lag between when a solve happens and when your code detects it (up to `CHECK_INTERVAL` seconds)
- Every poll costs network I/O, CPU, and API rate limit quota

---

### What WebSockets Are

A **WebSocket** is a protocol that provides a **persistent, full-duplex, bidirectional communication channel** over a single TCP connection.

Let's break each word down:

**Persistent** — the connection stays open. Unlike HTTP where the connection closes after each response, a WebSocket connection stays alive for as long as both sides want. No repeated TCP handshakes. No repeated HTTP overhead.

**Full-duplex** — both sides can send data at the same time, independently, in either direction. The server can push data to the client without the client asking. The client can send data while the server is sending data back.

**Bidirectional** — data flows both ways on the same connection.

**Single TCP connection** — all of this happens over one TCP connection with one port.

Compare:

```
HTTP (request-response):
Client ──── request ────▶ Server
Client ◀─── response ─── Server
[connection closes or waits]
Client ──── request ────▶ Server   ← must ask again to get new data
Client ◀─── response ─── Server

WebSocket (persistent, full-duplex):
Client ──── message ─────▶ Server
Client ◀─── message ────── Server  ← server can push ANY TIME
Client ──── message ─────▶ Server  ← client can send any time too
Client ◀─── message ────── Server
Client ◀─── message ────── Server  ← server pushes without client asking
[connection stays open indefinitely]
```

WebSocket is a software protocol — a specification for how bytes are framed and exchanged over a TCP connection. It is defined in RFC 6455. There is no new hardware involved. It uses the same network infrastructure as HTTP.

---

### The WebSocket Handshake

WebSockets start as HTTP. The client makes a special HTTP request called an **upgrade request**:

```
GET /ws HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

The server responds:

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

Status code `101 Switching Protocols` means: "I agree to upgrade. This TCP connection is no longer HTTP — it's WebSocket from now on."

After this handshake, the TCP connection is handed off from the HTTP handler to the WebSocket handler. HTTP framing is gone. WebSocket framing begins. The connection stays open.

This upgrade mechanism is why WebSockets work on port 80 (HTTP) and port 443 (HTTPS/WSS) — they start as HTTP and upgrade. Firewalls that allow HTTP traffic automatically allow WebSocket traffic because it begins as HTTP.

The URL scheme changes: `http://` → `ws://`, `https://` → `wss://`. `wss://` is WebSocket over TLS (encrypted), same as HTTPS.

---

### WebSocket Frames

After the handshake, data is exchanged as **frames**. A WebSocket frame has a small binary header (2–10 bytes) followed by the payload. The header contains:

- **Opcode** — what kind of frame this is: text (0x1), binary (0x2), close (0x8), ping (0x9), pong (0xA)
- **Payload length** — how many bytes follow
- **Mask bit** — client-to-server frames must be masked (XORed with a random key) per the spec. Server-to-client frames are unmasked.
- **FIN bit** — whether this is the final frame of a message (messages can be split across multiple frames)

You never deal with frames directly when using a library — the library handles framing for you. You just send and receive messages (strings or bytes).

---

### WebSockets in Python — the `websockets` library

```
pip install websockets
```

#### Client

python

```python
import asyncio
import websockets

async def connect():
    uri = "wss://echo.websocket.org"   # public echo server for testing

    async with websockets.connect(uri) as ws:
        # Send a message
        await ws.send("hello server")

        # Receive a message — blocks until one arrives
        message = await ws.recv()
        print(f"Received: {message}")

asyncio.run(connect())
```

`websockets.connect()` is an async context manager. On entry it performs the HTTP upgrade handshake and returns a WebSocket connection object. On exit it sends a close frame and closes the TCP connection.

#### Sending and receiving in a loop

python

```python
async def chat_client():
    async with websockets.connect("wss://example.com/ws") as ws:
        # Send initial message
        await ws.send("subscribe:solve-updates")

        # Listen for messages indefinitely
        async for message in ws:
            print(f"Server says: {message}")
            # async for loop ends when the connection closes
```

`async for message in ws` iterates over incoming messages as they arrive. The coroutine suspends at each iteration waiting for the next message. Other coroutines can run during this wait.

#### Server

python

```python
import asyncio
import websockets

async def handler(websocket):
    # websocket is the connection to one specific client
    async for message in websocket:
        print(f"Got: {message}")
        await websocket.send(f"Echo: {message}")

async def main():
    # Start server on port 8765
    async with websockets.serve(handler, "localhost", 8765):
        await asyncio.Future()   # run forever

asyncio.run(main())
```

`websockets.serve()` listens on the given port. For every new client that connects, it calls `handler(websocket)` as a new coroutine. Multiple clients are handled concurrently by the event loop — no threads needed.

---

### WebSockets in FastAPI

FastAPI has built-in WebSocket support. This is what a WebSocket endpoint looks like:

python

```python
from fastapi import FastAPI, WebSocket
from fastapi.websockets import WebSocketDisconnect

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()    # complete the handshake

    try:
        while True:
            # Receive a message from client
            data = await websocket.receive_text()
            print(f"Client sent: {data}")

            # Send a message to client
            await websocket.send_text(f"Received: {data}")

    except WebSocketDisconnect:
        print("Client disconnected")
```

The `@app.websocket("/ws")` decorator registers this as a WebSocket endpoint at `ws://host/ws`. When a client connects, FastAPI calls this coroutine. `await websocket.accept()` completes the HTTP→WebSocket upgrade. Then you loop, receiving and sending messages until the client disconnects.

`WebSocketDisconnect` is raised when the client closes the connection. You catch it to do cleanup — remove the client from active connections, etc.

#### Broadcasting to multiple clients

python

```python
from fastapi import FastAPI, WebSocket
from fastapi.websockets import WebSocketDisconnect

app = FastAPI()
connected_clients = []

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    connected_clients.append(websocket)

    try:
        while True:
            await websocket.receive_text()   # keep connection alive
    except WebSocketDisconnect:
        connected_clients.remove(websocket)

async def broadcast(message: str):
    for client in connected_clients:
        await client.send_text(message)
```

`broadcast()` can be called from anywhere — an HTTP endpoint, a background task, a solve checker. Every connected client receives the message immediately.

---

### How SolveTracker Would Work with WebSockets

Currently:

```
Background thread ──polls APIs──▶ updates self.status
Main display loop ──polls snapshot()──▶ reads self.status every 1s
```

In a web version, the architecture would change:

```
Background task ──polls APIs──▶ detects new solve ──▶ broadcasts over WebSocket
Browser client ──receives push──▶ updates display immediately
```

The browser never polls. It opens one WebSocket connection when the page loads and listens. The server pushes updates the instant they're detected.

Here is what the full server-side solve tracker would look like:

python

```python
from fastapi import FastAPI, WebSocket
from fastapi.websockets import WebSocketDisconnect
import asyncio
import json

app = FastAPI()
connected_clients = []

# Background task that replaces SolveTracker thread
async def solve_checker_loop(problems):
    last_status = {p["id"]: False for p in problems}

    while True:
        await asyncio.sleep(30)    # wait 30 seconds between checks

        # Run the blocking check_solved in a thread pool
        # so it doesn't block the event loop
        loop = asyncio.get_event_loop()
        fresh = await loop.run_in_executor(None, check_solved, problems)

        for pid, solved in fresh.items():
            if solved and not last_status[pid]:
                # New solve detected — push to all clients immediately
                await broadcast(json.dumps({
                    "type": "solve",
                    "problem_id": pid,
                }))
            last_status[pid] = solved

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    connected_clients.append(websocket)
    try:
        while True:
            await websocket.receive_text()
    except WebSocketDisconnect:
        connected_clients.remove(websocket)

async def broadcast(message: str):
    disconnected = []
    for client in connected_clients:
        try:
            await client.send_text(message)
        except Exception:
            disconnected.append(client)
    for client in disconnected:
        connected_clients.remove(client)

@app.on_event("startup")
async def startup():
    problems = [...]   # loaded from setup
    asyncio.create_task(solve_checker_loop(problems))
```

The browser side would be JavaScript:

javascript

```javascript
const ws = new WebSocket("ws://localhost:8000/ws");

ws.onmessage = function(event) {
    const data = JSON.parse(event.data);
    if (data.type === "solve") {
        markSolved(data.problem_id);   // update the UI immediately
    }
};
```

No polling anywhere. The server pushes. The client reacts.

---

### HTTP Polling vs WebSockets — Precise Comparison

||HTTP Polling|WebSockets|
|---|---|---|
|Connection|New TCP connection per request|One persistent TCP connection|
|Latency|Up to poll interval (30s in your code)|Near-zero — pushed immediately|
|Server can push|No — must wait for client request|Yes — anytime|
|Overhead per update|Full HTTP headers (~500 bytes) each time|2–10 byte WebSocket frame header|
|Wasted requests|Many — most polls find no new data|None — only sends when there's data|
|Implementation complexity|Simple|More complex — connection management, reconnection logic|
|Good for|Infrequent, low-latency-tolerance updates|Real-time, low-latency, high-frequency updates|

### Connection Management — What You Have to Handle

WebSockets introduce state that HTTP doesn't have. You need to handle:

#### Disconnection

Clients disconnect unexpectedly — network drops, browser tab closes, laptop sleeps. Your server must detect this and clean up the client from `connected_clients`. Sending to a dead connection raises an exception — your `broadcast` function handles this with the `disconnected` list pattern above.

#### Reconnection (client side)

If the server restarts or the connection drops, the client needs to reconnect. WebSockets don't auto-reconnect. You have to implement this:

javascript

```javascript
function connect() {
    const ws = new WebSocket("ws://localhost:8000/ws");

    ws.onclose = function() {
        console.log("disconnected, reconnecting in 3s...");
        setTimeout(connect, 3000);   // retry after 3 seconds
    };

    ws.onmessage = function(event) {
        // handle messages
    };
}

connect();
```

#### Ping/Pong — keepalive

TCP connections can be silently dropped by firewalls, NAT devices, and load balancers that kill idle connections. WebSocket has a built-in **ping/pong** mechanism: the server sends a ping frame, the client responds with a pong frame. If no pong arrives, the connection is dead.

`websockets` and FastAPI handle this automatically with a `ping_interval` parameter:

python

```python
async with websockets.serve(handler, "localhost", 8765, ping_interval=20):
    ...
```

Every 20 seconds, the library sends a ping. If no pong arrives within `ping_timeout` seconds, the connection is closed.

---

### Full Picture — Where Everything Fits

You've now covered the full stack from the ground up:

```
Physical network (cables, switches, routers)
        │
        ▼
IP — addresses machines
        │
        ▼
TCP — reliable ordered stream between two ports
        │
        ▼
HTTP — request/response protocol over TCP
    ├── REST APIs (Codeforces, AtCoder, LeetCode)
    └── WebSocket upgrade → persistent full-duplex channel
                │
                ▼
        Push-based real-time updates
        (solve detection → browser instantly)
```

And the software layers in your CP tool:

```
requests / aiohttp     ← HTTP client (Topics 1, 7)
        │
_req / _req_async      ← your network wrapper
        │
cf/ac/lc fetchers      ← REST API consumers (Topic 2)
lc GraphQL queries     ← GraphQL over HTTP POST (Topic 3)
        │
SolveTracker           ← threading, Lock, Event (Topic 6)
        │
Terminal display       ← ANSI escape codes (Topic 5)

Future web version:
FastAPI                ← HTTP + WebSocket server (Topics 8, 9)
        │
WebSocket endpoint     ← pushes solve updates to browser
        │
asyncio event loop     ← runs fetchers + WebSocket handler concurrently (Topic 7)
```