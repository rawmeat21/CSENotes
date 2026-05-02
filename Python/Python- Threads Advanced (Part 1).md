## Threading — Synchronization Primitives

We'll skip the basics you already know and go straight into the meat: **Lock, RLock, Event, Condition, Semaphore, and BoundedSemaphore** — what each one is, how it works internally, when to use it, and lots of examples. Then we'll tie everything back to your `SolveTracker`.

---

### Why Synchronization Primitives Exist

When two or more threads access **shared mutable state** — a variable, a dict, a list, anything — without coordination, you get a **race condition**. The outcome depends on the exact timing of thread scheduling, which is non-deterministic. Your program produces different (wrong) results on different runs.

Before getting into the primitives, you need to understand what a race condition actually looks like at the instruction level.

python

```python
# Two threads both run this:
counter = 0

def increment():
    global counter
    counter += 1  # looks atomic, is NOT
```

`counter += 1` compiles to three operations:

1. READ `counter` from memory into a register
2. ADD 1 to the register
3. WRITE the register back to `counter`

If two threads interleave:

```
Thread A: READ  counter → gets 0
Thread B: READ  counter → gets 0
Thread A: ADD 1         → register = 1
Thread B: ADD 1         → register = 1
Thread A: WRITE 1       → counter = 1
Thread B: WRITE 1       → counter = 1
```

Both threads incremented, but `counter` is `1` instead of `2`. One increment was lost. This is a race condition.

Synchronization primitives exist to prevent this — they give threads a way to coordinate access to shared state.

---

### The GIL — What It Actually Means

Python has a **Global Interpreter Lock (GIL)** — a mutex internal to CPython that ensures only one thread executes Python bytecode at a time.

This means:

- Pure Python threads don't run in true parallel on multiple CPU cores
- The GIL makes individual bytecode operations atomic — a single bytecode instruction won't be interrupted mid-execution
- BUT the GIL releases between bytecodes, and `counter += 1` is multiple bytecodes — so the race condition above is still real even with the GIL

The GIL does NOT make your code thread-safe. It only prevents two bytecodes from executing simultaneously. A logical operation like `counter += 1` is multiple bytecodes, so it's still interruptible.

The GIL does release during I/O operations (network calls, file reads, `time.sleep`). This is why threading is still useful in Python for I/O-bound work — while one thread is blocked waiting for an HTTP response, other threads can run. Your `SolveTracker` is I/O-bound (it makes network requests), so threading gives you real concurrency benefit.

---

### `threading.Lock`

A `Lock` is the most fundamental synchronization primitive. It has two states: **locked** and **unlocked**. It has two operations:

- `.acquire()` — if the lock is unlocked, lock it and return immediately. If it's already locked, **block** (wait) until it becomes unlocked, then lock it and return.
- `.release()` — unlock the lock, allowing one waiting thread to proceed.

Only one thread can hold the lock at a time. This guarantees **mutual exclusion** — the region of code between `acquire()` and `release()` (called the **critical section**) can only be executed by one thread at a time.

#### Basic usage

python

```python
import threading

lock = threading.Lock()
counter = 0

def increment():
    global counter
    lock.acquire()
    counter += 1      # critical section — only one thread here at a time
    lock.release()
```

#### Context manager — always use this

The problem with `acquire()`/`release()` directly: if an exception occurs inside the critical section, `release()` never gets called. The lock stays locked forever. Every other thread blocks indefinitely. Deadlock.

Use the `with` statement instead — it calls `acquire()` on entry and `release()` on exit, even if an exception is raised:

python

```python
def increment():
    global counter
    with lock:
        counter += 1   # lock released automatically, even on exception
```

Always use `with lock:`. Never use bare `acquire()`/`release()` unless you have a specific reason.

#### Full example — fixing the race condition

python

```python
import threading

lock = threading.Lock()
counter = 0

def increment_many():
    global counter
    for _ in range(100000):
        with lock:
            counter += 1

t1 = threading.Thread(target=increment_many)
t2 = threading.Thread(target=increment_many)
t1.start()
t2.start()
t1.join()
t2.join()

print(counter)  # always 200000 — no race condition
```

Without the lock, this would print a different value every run, always less than 200000.

#### Where this appears in your code

Your `SolveTracker` uses a lock to protect shared state that's written by the background thread and read by the main thread:

python

```python
# line 807
self._lock = threading.Lock()
```

The background thread (`_do_check`) writes to `self.status` and `self.solve_times`:

python

```python
# lines 838-842
with self._lock:
    for pid, solved in fresh.items():
        if solved and not self.status[pid]:
            self.solve_times[pid] = elapsed
        self.status[pid] = solved
```

The main thread reads this state via `snapshot()`:

python

```python
# lines 846-848
def snapshot(self):
    with self._lock:
        return dict(self.status), dict(self.solve_times), self.last_check
```

Without the lock, the main thread could read `self.status` while the background thread is halfway through updating it — some entries updated, some not. The display would show an inconsistent state. The lock ensures `snapshot()` always sees a fully-updated state.

Also notice `dict(self.status)` — this creates a **copy** of the dict rather than returning a reference. If the main thread held a reference to `self.status` directly, the background thread could modify it while the main thread is iterating it, causing a `RuntimeError: dictionary changed size during iteration`. The copy is taken inside the lock, so it's a consistent snapshot.

#### Why `last_check` is set at the START of `_do_check`

python

```python
def _do_check(self):
    check_started = datetime.now()      # record NOW, before the network call
    with self._lock:
        self.last_check = check_started  # store it
    try:
        fresh = check_solved(self.problems)  # this takes several seconds
        ...
```

If `last_check` were set _after_ `check_solved()`, it would show the time the check _finished_. The display would say "last checked 0 seconds ago" right after the check completes, then immediately jump to "15 seconds ago" as time passes during the next interval. Setting it at the _start_ means it shows when the check began, which is what you actually want to display — "I started checking N seconds ago."

---

### `threading.RLock` — Reentrant Lock

A regular `Lock` has one problem: if the **same thread** tries to acquire it twice, it deadlocks with itself.

python

```python
lock = threading.Lock()

def outer():
    with lock:
        inner()   # outer holds lock, inner tries to acquire it → deadlock

def inner():
    with lock:    # blocks forever — same thread, lock already held
        ...
```

An `RLock` (Reentrant Lock) allows the **same thread** to acquire it multiple times. It maintains a count — each `acquire()` increments it, each `release()` decrements it. The lock is fully released only when the count reaches zero.

python

```python
rlock = threading.RLock()

def outer():
    with rlock:     # count → 1
        inner()
    # count → 0, released

def inner():
    with rlock:     # same thread — count → 2, doesn't block
        ...
    # count → 1
```

**When to use RLock:** when you have recursive functions or methods that call each other, and both need to hold the same lock. In your code, `Lock` is sufficient because no method calls another method that also acquires the same lock.

---

### `threading.Event`

An `Event` is a simple signaling mechanism between threads. It wraps a boolean flag with blocking wait capability.

It has four methods:

- `.set()` — set the flag to `True`. All threads waiting on `.wait()` are woken up.
- `.clear()` — reset the flag to `False`.
- `.is_set()` — returns `True` if the flag is set.
- `.wait(timeout=None)` — block until the flag is set. If `timeout` is given, block for at most that many seconds, then return regardless. Returns `True` if the flag was set, `False` if it timed out.

#### Basic example — one thread signals another

python

```python
import threading
import time

event = threading.Event()

def waiter():
    print("Waiter: waiting for signal...")
    event.wait()             # blocks here until event is set
    print("Waiter: got the signal, proceeding")

def signaler():
    time.sleep(2)
    print("Signaler: setting event")
    event.set()

t1 = threading.Thread(target=waiter)
t2 = threading.Thread(target=signaler)
t1.start()
t2.start()
t1.join()
t2.join()
```

Output:

```
Waiter: waiting for signal...
Signaler: setting event
Waiter: got the signal, proceeding
```

#### Using `wait(timeout)` for interruptible sleep

A common pattern: you want a thread to sleep for N seconds between iterations, but you also want it to stop immediately when asked. `time.sleep(N)` is not interruptible — the thread will sleep for the full N seconds even if you want it to stop.

`event.wait(timeout)` solves this:

python

```python
stop_event = threading.Event()

def worker():
    while not stop_event.is_set():
        do_work()
        stop_event.wait(timeout=5)   # sleep 5s, OR wake up immediately if stop is set

def stop_worker():
    stop_event.set()   # worker wakes up and exits on next loop iteration
```

#### Where this appears in your code

Your `SolveTracker` uses an `Event` for exactly this pattern:

python

```python
# line 808
self._stop = threading.Event()
```

The `_loop` method uses it for interruptible sleeping:

python

```python
# lines 823-826
slept = 0
while slept < CHECK_INTERVAL and not self._stop.is_set():
    self._stop.wait(1)    # sleep 1 second, OR wake immediately if stop is set
    slept += 1
```

Instead of `time.sleep(CHECK_INTERVAL)` (which would make the thread unresponsive for 30 seconds after the contest ends), the thread sleeps in 1-second increments, checking `_stop` each time. When `tracker.stop()` is called:

python

```python
# line 815
def stop(self):
    self._stop.set()
```

The `_stop.wait(1)` in the loop returns immediately (because the event is now set), `_stop.is_set()` returns `True`, and the loop exits. The thread terminates within 1 second of `stop()` being called rather than up to 30 seconds later.

#### Event vs Lock

These solve different problems:

- **Lock** — controls _access_ to shared data. One thread at a time.
- **Event** — sends a _signal_ between threads. One thread notifies another that something happened.

---

### `threading.Condition`

A `Condition` is the most powerful synchronization primitive. It combines a lock with a **wait/notify** mechanism. It's used when one thread needs to wait until some condition in the shared state becomes true.

It wraps a lock (by default an `RLock`) and adds:

- `.wait(timeout=None)` — **atomically** release the lock and block. When notified, re-acquire the lock and return.
- `.notify(n=1)` — wake up one (or n) waiting threads.
- `.notify_all()` — wake up all waiting threads.

The critical word is **atomically** — releasing the lock and starting to wait happen as one operation. This prevents the race condition where a notification arrives between the unlock and the wait.

#### The producer-consumer problem

This is the canonical use case for Condition. One thread produces items, another consumes them. The consumer must wait when there's nothing to consume.

python

```python
import threading
import time
import random

queue = []
condition = threading.Condition()

def producer():
    for i in range(5):
        time.sleep(random.uniform(0.5, 1.5))
        with condition:
            queue.append(i)
            print(f"Produced: {i}, queue size: {len(queue)}")
            condition.notify()    # wake up one waiting consumer

def consumer():
    consumed = 0
    while consumed < 5:
        with condition:
            while not queue:             # while, not if — spurious wakeups
                condition.wait()         # release lock, sleep, re-acquire on wake
            item = queue.pop(0)
            consumed += 1
        print(f"Consumed: {item}")

t1 = threading.Thread(target=producer)
t2 = threading.Thread(target=consumer)
t1.start()
t2.start()
t1.join()
t2.join()
```

**Why `while not queue` and not `if not queue`:**

When a thread is woken by `notify()`, it doesn't mean the condition is necessarily true anymore. Another thread might have consumed the item between the notify and the woken thread re-acquiring the lock. This is called a **spurious wakeup**. Always re-check the condition in a `while` loop after waking.

This pattern is so standard it has a name: **the monitor pattern**:

python

```python
with condition:
    while not <condition is true>:
        condition.wait()
    # now safe to proceed — condition is guaranteed true here
```

#### Multiple consumers example

python

```python
import threading
import time

buffer = []
MAX_SIZE = 3
condition = threading.Condition()

def producer(name, count):
    for i in range(count):
        with condition:
            while len(buffer) >= MAX_SIZE:
                print(f"{name}: buffer full, waiting")
                condition.wait()                    # wait until consumer makes space
            buffer.append(f"{name}-item{i}")
            print(f"{name}: produced, buffer={buffer}")
            condition.notify_all()                  # wake all — both consumers and producers

def consumer(name, count):
    for _ in range(count):
        with condition:
            while not buffer:
                print(f"{name}: buffer empty, waiting")
                condition.wait()
            item = buffer.pop(0)
            print(f"{name}: consumed {item}, buffer={buffer}")
            condition.notify_all()

threads = [
    threading.Thread(target=producer, args=("P1", 4)),
    threading.Thread(target=consumer, args=("C1", 2)),
    threading.Thread(target=consumer, args=("C2", 2)),
]
for t in threads:
    t.start()
for t in threads:
    t.join()
```

`notify_all()` is used here because after a consumer removes an item, a blocked producer should be notified. After a producer adds an item, blocked consumers should be notified. Since both kinds of waiters exist, `notify_all()` wakes everyone and lets them re-check their condition.

#### Condition vs Event

- **Event** — single binary flag. "Something happened." All waiters are woken when set. No associated lock.
- **Condition** — "wait until some state in shared data is true." Has an associated lock. Supports re-checking the condition after waking. More precise control over who gets woken.

---

### `threading.Semaphore`

A `Semaphore` is a generalization of a lock. Instead of allowing only one thread into a critical section, it allows up to **N** threads simultaneously. It maintains an internal counter initialized to N.

- `.acquire()` — if counter > 0, decrement and proceed. If counter == 0, block until another thread releases.
- `.release()` — increment the counter, wake one waiting thread if any.

A `Lock` is equivalent to `Semaphore(1)`.

#### Use case — limiting concurrent access

The canonical use case: you have a resource that can handle a limited number of concurrent users. For example, a database connection pool, or an API that allows max 5 concurrent requests.

python

```python
import threading
import time

# Only 3 threads allowed to make network requests simultaneously
semaphore = threading.Semaphore(3)

def fetch_data(thread_id):
    print(f"Thread {thread_id}: waiting to fetch")
    with semaphore:
        print(f"Thread {thread_id}: fetching...")
        time.sleep(2)    # simulate network call
        print(f"Thread {thread_id}: done")

threads = [threading.Thread(target=fetch_data, args=(i,)) for i in range(8)]
for t in threads:
    t.start()
for t in threads:
    t.join()
```

Output (interleaved but only 3 "fetching" lines at a time):

```
Thread 0: waiting to fetch
Thread 1: waiting to fetch
...
Thread 0: fetching...
Thread 1: fetching...
Thread 2: fetching...
Thread 3: waiting to fetch    ← blocked, semaphore at 0
Thread 4: waiting to fetch    ← blocked
Thread 0: done                ← releases semaphore
Thread 3: fetching...         ← now gets in
...
```

#### Semaphore as a rate limiter for your fetchers

Your code makes multiple API calls per check cycle. If you ever extend it to run multiple platform checks concurrently (using threads or asyncio), you'd use a semaphore to stay within API rate limits:

python

```python
api_semaphore = threading.Semaphore(2)  # max 2 concurrent API calls

def rate_limited_req(url, **kwargs):
    with api_semaphore:
        return _req(url, **kwargs)
```

#### Semaphore as a "slots available" counter — another pattern

python

```python
import threading

# Simulate 5 parking slots
parking = threading.Semaphore(5)

def car(car_id):
    print(f"Car {car_id} arriving")
    parking.acquire()
    print(f"Car {car_id} parked  ({parking._value} slots left)")
    time.sleep(random.uniform(1, 3))
    print(f"Car {car_id} leaving")
    parking.release()
```

---

### `threading.BoundedSemaphore`

A `BoundedSemaphore` is identical to `Semaphore` with one difference: calling `.release()` more times than `.acquire()` raises a `ValueError`. A regular `Semaphore` allows this silently — the counter goes above the initial value, which is almost always a bug.

python

```python
s = threading.Semaphore(2)
s.release()   # counter goes to 3 — silent bug

bs = threading.BoundedSemaphore(2)
bs.release()  # ValueError: Semaphore released too many times
```

**Always use `BoundedSemaphore` instead of `Semaphore`** unless you have a specific reason not to. If your acquire/release accounting has a bug, `BoundedSemaphore` tells you immediately rather than silently corrupting the counter.

---

### Putting It All Together — Comparison Table

|Primitive|Internal State|Blocks When|Use Case|
|---|---|---|---|
|`Lock`|locked/unlocked|trying to acquire a locked lock|Mutual exclusion — one thread at a time in critical section|
|`RLock`|locked/unlocked + owner + count|non-owner trying to acquire|Same as Lock, but safe for recursive/reentrant acquisition by same thread|
|`Event`|True/False flag|`.wait()` called when flag is False|Signaling — one thread notifies another that something happened|
|`Condition`|lock + waiters queue|`.wait()` called|Waiting for a condition in shared state to become true|
|`Semaphore`|integer counter ≥ 0|counter is 0 and `.acquire()` called|Limiting concurrency — N threads allowed simultaneously|
|`BoundedSemaphore`|integer counter ≥ 0, bounded|counter is 0 or release exceeds bound|Same as Semaphore but with over-release protection|

---

### Full picture of `SolveTracker`

Now that you know all the primitives, here's how `SolveTracker` uses them and why each choice was made:

python

```python
class SolveTracker:
    def __init__(self, problems, start_dt):
        self.status      = {p["id"]: False for p in problems}
        self.solve_times = {}
        self.last_check  = None
        self._lock       = threading.Lock()    # protects status, solve_times, last_check
        self._stop       = threading.Event()   # signals the background thread to stop
```

**`Lock`** — protects `status`, `solve_times`, `last_check` from concurrent access. The background thread writes them in `_do_check`. The main thread reads them in `snapshot()` and `all_solved()`. Without the lock, the main thread could see a partially-updated state mid-write.

**`Event`** — signals the background thread to stop. When the contest ends (timer expires or all solved), `tracker.stop()` sets the event. The background thread's sleep loop checks the event every 1 second and exits promptly.

**Why not `Condition` here?** The main thread never _waits_ for the background thread — it polls via `snapshot()` every second in its own loop. A Condition would be appropriate if the main thread needed to block until the background thread produced new data. Since the main thread drives its own 1-second refresh loop independently, a simple Lock for data protection and an Event for stop signaling is sufficient.

**Why not `Semaphore` here?** There's only one background thread doing checks. Semaphores are for limiting concurrent access to a resource. Not applicable here.