## Multithreading in Python

---

### The GIL — Global Interpreter Lock

Before anything else, you need to understand this because it defines what threading in Python actually means.

CPython (the standard Python interpreter) has a **Global Interpreter Lock** — a mutex that allows only **one thread to execute Python bytecode at a time**. Even on a multi-core machine, Python threads do not run Python code in parallel.

This means:

- **CPU-bound work** — computation, number crunching, tight loops — gets **no speedup** from threading. You have multiple threads but only one runs at a time.
- **IO-bound work** — network requests, file IO, database queries, waiting — **does benefit** from threading. When a thread is waiting for IO, it releases the GIL, allowing another thread to run.

```
Thread 1: ──run──|wait for IO|──────run──
Thread 2:        |───────run──|wait|─────
                  ↑
          Thread 1 releases GIL during IO wait
          Thread 2 acquires it and runs
```

For CPU-bound parallelism you use `multiprocessing` — separate processes, each with their own GIL. Threading in Python is specifically for IO-bound concurrency.

---

### `threading` Module

python

```python
import threading
```

---

### Creating and Starting Threads

python

```python
import threading

def task(name, count):
    for i in range(count):
        print(f"{name}: {i}")

t = threading.Thread(target=task, args=("Thread-1", 5))
t.start()    # starts the thread — task() runs in a new thread
t.join()     # blocks the calling thread until t finishes
```

- `target` — the callable to run in the thread
- `args` — tuple of positional arguments
- `kwargs` — dict of keyword arguments

`t.start()` returns immediately — the thread runs concurrently. `t.join()` blocks the calling thread (usually the main thread) until `t` completes.

#### Multiple threads

python

```python
threads = []

for i in range(5):
    t = threading.Thread(target=task, args=(f"Thread-{i}", 3))
    threads.append(t)
    t.start()

for t in threads:
    t.join()    # wait for all to finish
```

Start all first, then join all — otherwise you'd be waiting for each thread to finish before starting the next one.

---

### Subclassing `Thread`

Alternative to `target` — define a class that extends `Thread` and override `run()`:

python

```python
class MyThread(threading.Thread):
    def __init__(self, name, count):
        super().__init__()
        self.name = name
        self.count = count

    def run(self):              # called when t.start() is invoked
        for i in range(self.count):
            print(f"{self.name}: {i}")

t = MyThread("Worker", 5)
t.start()
t.join()
```

Override `run()`, not `start()`. `start()` sets up the thread and calls `run()` internally.

---

### Thread Properties

python

```python
t.name          # thread name — settable, used in logs
t.ident         # thread ID assigned by OS
t.daemon        # bool — daemon thread or not (below)
t.is_alive()    # True if thread is running
```

#### Daemon threads

A **daemon thread** is a background thread that is automatically killed when all non-daemon threads (including the main thread) have finished. The program does not wait for daemon threads to complete.

python

```python
t = threading.Thread(target=background_task)
t.daemon = True    # must be set before t.start()
t.start()
# when main thread ends, t is killed automatically
```

Non-daemon threads (the default) keep the program alive until they finish. The main thread is always non-daemon.

Use daemon threads for background services — log flushing, heartbeats, monitoring — that should not block program exit.

---

### Race Conditions

When multiple threads access and modify **shared mutable state**, you get a race condition — the result depends on the timing of thread execution, which is non-deterministic.

python

```python
import threading

counter = 0

def increment():
    global counter
    for _ in range(100000):
        counter += 1    # NOT atomic — read, add, write are three separate steps

threads = [threading.Thread(target=increment) for _ in range(5)]
for t in threads: t.start()
for t in threads: t.join()

print(counter)   # should be 500000, but will be less — race condition
```

`counter += 1` compiles to multiple bytecode instructions:

1. Load `counter`
2. Load `1`
3. Add
4. Store back to `counter`

The GIL can switch between threads between any of these steps. Two threads can both read the same value, both add 1, and both write back the same result — one increment is lost.

---

### `Lock` — Mutex

A `Lock` is a mutual exclusion object. Only one thread can hold it at a time. Other threads that try to acquire it block until it is released.

python

```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(100000):
        lock.acquire()
        counter += 1
        lock.release()

threads = [threading.Thread(target=increment) for _ in range(5)]
for t in threads: t.start()
for t in threads: t.join()

print(counter)   # 500000 — correct
```

Always release the lock — even if an exception occurs. Use `with` for safety:

python

```python
def increment():
    global counter
    for _ in range(100000):
        with lock:            # acquires on enter, releases on exit — always
            counter += 1
```

`with lock:` is the standard pattern. It is equivalent to `try/finally` with `acquire/release`.

---

### `RLock` — Reentrant Lock

A regular `Lock` cannot be acquired twice by the same thread — it deadlocks itself. An `RLock` (reentrant lock) can be acquired multiple times by the **same thread**. It must be released the same number of times it was acquired.

python

```python
lock = threading.RLock()

def outer():
    with lock:
        inner()       # same thread acquires lock again — fine with RLock

def inner():
    with lock:        # would deadlock with regular Lock
        print("inner")
```

Use `RLock` when a function that holds a lock needs to call another function that also acquires the same lock.

---

### `Event` — Thread Signaling

An `Event` is a simple flag that threads can wait on or signal. One thread sets the event, others waiting on it unblock.

python

```python
import threading

event = threading.Event()

def waiter():
    print("waiting for event")
    event.wait()               # blocks until event is set
    print("event received, continuing")

def setter():
    import time
    time.sleep(2)
    print("setting event")
    event.set()                # unblocks all threads waiting on this event

t1 = threading.Thread(target=waiter)
t2 = threading.Thread(target=setter)

t1.start()
t2.start()
t1.join()
t2.join()
```

```
event.set()      # set the flag — all waiters unblock
event.clear()    # reset the flag
event.is_set()   # True if set
event.wait(timeout=5)   # block for up to 5 seconds
```

---

### `Condition` — Fine-Grained Signaling

A `Condition` is a lock with the ability to wait for a specific condition and be notified when it changes. Used for producer-consumer type coordination.

python

```python
import threading
import time
from collections import deque

queue = deque()
condition = threading.Condition()

def producer():
    for i in range(5):
        time.sleep(1)
        with condition:
            queue.append(i)
            print(f"produced {i}")
            condition.notify()       # wake up one waiting thread

def consumer():
    for _ in range(5):
        with condition:
            while not queue:         # always check condition in a loop
                condition.wait()     # releases lock and waits — reacquires on wake
            item = queue.popleft()
            print(f"consumed {item}")

t1 = threading.Thread(target=producer)
t2 = threading.Thread(target=consumer)
t1.start()
t2.start()
t1.join()
t2.join()
```

Key points:

- `condition.wait()` releases the lock and blocks. When notified, it reacquires the lock before returning.
- Always check the actual condition in a `while` loop — not `if`. A thread can wake up spuriously.
- `condition.notify()` wakes one waiting thread. `condition.notify_all()` wakes all.

---

### `Semaphore` — Limiting Concurrency

A `Semaphore` maintains an internal counter. `acquire()` decrements it — blocks if zero. `release()` increments it. Used to limit how many threads access a resource simultaneously.

python

```python
import threading
import time

semaphore = threading.Semaphore(3)   # at most 3 threads at once

def worker(n):
    with semaphore:
        print(f"thread {n} working")
        time.sleep(2)
        print(f"thread {n} done")

threads = [threading.Thread(target=worker, args=(i,)) for i in range(10)]
for t in threads: t.start()
for t in threads: t.join()
```

At most 3 threads run `worker` simultaneously. The other 7 wait at `with semaphore:`.

---

### `Barrier` — Synchronization Point

A `Barrier` makes a set of threads wait until all of them have reached a certain point, then releases them all simultaneously:

python

```python
import threading

barrier = threading.Barrier(3)   # wait for 3 threads

def worker(n):
    print(f"thread {n} doing phase 1")
    barrier.wait()                # all 3 must reach here before any continues
    print(f"thread {n} doing phase 2")

threads = [threading.Thread(target=worker, args=(i,)) for i in range(3)]
for t in threads: t.start()
for t in threads: t.join()
```

All threads block at `barrier.wait()` until all 3 have called it. Then they all proceed.

---

### `Queue` — Thread-Safe Data Exchange

`queue.Queue` is a thread-safe FIFO queue. The preferred way to pass data between threads — no manual locking needed:

python

```python
import threading
import queue
import time

q = queue.Queue(maxsize=5)    # maxsize=0 means unlimited

def producer():
    for i in range(10):
        q.put(i)               # blocks if queue is full
        print(f"put {i}")
        time.sleep(0.1)

def consumer():
    while True:
        item = q.get()         # blocks if queue is empty
        if item is None:       # sentinel value to stop
            break
        print(f"got {item}")
        q.task_done()          # signals that item processing is complete

t1 = threading.Thread(target=producer)
t2 = threading.Thread(target=consumer)

t1.start()
t2.start()
t1.join()

q.put(None)    # send sentinel to stop consumer
t2.join()
```

python

```python
q.put(item)             # add item — blocks if full
q.put_nowait(item)      # add item — raises queue.Full if full
q.get()                 # remove item — blocks if empty
q.get_nowait()          # remove item — raises queue.Empty if empty
q.task_done()           # signal processing complete
q.join()                # block until all items have been task_done'd
q.empty()               # True if empty
q.qsize()               # approximate size
```

`queue.LifoQueue` is a stack. `queue.PriorityQueue` is a min-heap priority queue — items are tuples `(priority, data)`.

---

### `ThreadPoolExecutor` — High Level API

From `concurrent.futures`. Manages a pool of worker threads. The highest-level and most convenient threading interface:

python

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

def fetch(url):
    import urllib.request
    with urllib.request.urlopen(url) as r:
        return len(r.read())

urls = [
    "http://example.com",
    "http://example.org",
    "http://example.net",
]

with ThreadPoolExecutor(max_workers=4) as executor:
    futures = [executor.submit(fetch, url) for url in urls]

    for future in as_completed(futures):
        result = future.result()
        print(f"page size: {result}")
```

- `executor.submit(fn, *args)` — schedules `fn(*args)` in the pool, returns a `Future`
- `future.result()` — blocks until done and returns the value (or raises if the function raised)
- `as_completed(futures)` — yields futures as they complete (not in submission order)
- The `with` block waits for all submitted tasks before exiting

#### `executor.map`

Simpler when you have one function and many inputs:

python

```python
with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(fetch, urls))
    # results are in the same order as urls
```

`executor.map` returns an iterator of results in input order. Exceptions are re-raised when you iterate.

---

### `local()` — Thread-Local Storage

Data that is unique to each thread — each thread sees its own copy:

python

```python
import threading

local_data = threading.local()

def worker(value):
    local_data.value = value          # each thread sets its own copy
    import time
    time.sleep(0.1)
    print(f"{threading.current_thread().name}: {local_data.value}")

threads = [threading.Thread(target=worker, args=(i,)) for i in range(5)]
for t in threads: t.start()
for t in threads: t.join()
```

Each thread's `local_data.value` is independent. Setting it in one thread does not affect others. Used for per-thread state like database connections or request context in web frameworks.

---

### Deadlock

A deadlock occurs when two or more threads are each waiting for a lock held by the other — circular wait, no progress:

python

```python
lock_a = threading.Lock()
lock_b = threading.Lock()

def thread1():
    with lock_a:
        time.sleep(0.1)
        with lock_b:        # waits for lock_b
            print("thread1")

def thread2():
    with lock_b:
        time.sleep(0.1)
        with lock_a:        # waits for lock_a
            print("thread2")
```

```
Thread 1 holds lock_a, waits for lock_b
Thread 2 holds lock_b, waits for lock_a
→ both wait forever
```

Prevention strategies:

- **Lock ordering** — always acquire locks in the same order across all threads
- **Lock timeout** — use `lock.acquire(timeout=5)` and handle failure
- **Avoid nested locks** — restructure to not hold multiple locks simultaneously

---
