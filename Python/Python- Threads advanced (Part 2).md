### Daemon Threads

A daemon thread is a thread that gets **killed automatically when the main program exits**. Non-daemon threads (the default) keep the program alive — Python will not exit until every non-daemon thread finishes. Daemon threads are the opposite — they don't block program exit at all.

python

```python
import threading
import time

def background_task():
    while True:
        print("background running...")
        time.sleep(1)

# Non-daemon — program will NEVER exit because this thread runs forever
t = threading.Thread(target=background_task)
t.start()
# ^ program hangs after main thread finishes, waiting for t to finish

# Daemon — program exits as soon as main thread finishes
t = threading.Thread(target=background_task, daemon=True)
t.start()
# ^ program exits cleanly, thread is killed automatically
```

The rule: use daemon threads for background workers that exist to serve the main program and have no cleanup work to do. If your thread needs to finish cleanly (flush a file, close a DB connection, send a final network request), do NOT make it a daemon — you need explicit `stop()` + `join()` instead.

Your `SolveTracker` uses a daemon thread:

python

```python
# line 811
self._thread = threading.Thread(target=self._loop, daemon=True)
```

This is the right call because the background thread just polls APIs in a loop. It has nothing critical to finish. If the main thread exits (Ctrl+C, contest over, crash), killing the background thread immediately is fine — no data loss, no corruption.

---

### `threading.Thread` — Full API

python

```python
t = threading.Thread(
    target=some_function,     # callable to run in the new thread
    args=(arg1, arg2),        # positional arguments to target — must be a tuple
    kwargs={"key": "value"},  # keyword arguments to target — must be a dict
    daemon=True,              # daemon flag — default is None (inherits from parent)
    name="my-thread"          # optional name, useful for debugging
)
```

#### Starting and joining

python

```python
t.start()       # spawn the thread and begin execution — returns immediately
t.join()        # block until the thread finishes
t.join(timeout=5)  # block for at most 5 seconds, then return regardless
t.is_alive()    # True if thread is running, False if not started or finished
```

#### Common mistake — `args` must be a tuple

python

```python
# WRONG — passes the integer 5 as args, not a tuple containing 5
t = threading.Thread(target=fn, args=5)

# WRONG — common mistake with single argument
t = threading.Thread(target=fn, args=(5))   # (5) is just 5 in parentheses, not a tuple

# CORRECT
t = threading.Thread(target=fn, args=(5,))  # trailing comma makes it a tuple
t = threading.Thread(target=fn, args=[5])   # list also works
```

#### Common mistake — calling the function instead of passing it

python

```python
# WRONG — fn() is called immediately in the main thread, return value passed as target
t = threading.Thread(target=fn())

# CORRECT — pass the function object itself
t = threading.Thread(target=fn)
```

#### Starting without joining — a real bug

python

```python
def write_result(results, value):
    time.sleep(0.1)    # simulate work
    results.append(value)

results = []
threads = []

for i in range(5):
    t = threading.Thread(target=write_result, args=(results, i))
    t.start()
    threads.append(t)

# WRONG — main thread might print before any thread finishes
print(results)   # often prints [] or partial list

# CORRECT — wait for all threads to finish
for t in threads:
    t.join()
print(results)   # guaranteed to have all 5 results
```

---

### Wrong vs Right — Lock with bare acquire/release

python

```python
lock = threading.Lock()
counter = 0

# WRONG — exception between acquire and release leaves lock permanently locked
def broken_increment():
    global counter
    lock.acquire()
    counter += might_raise_exception()   # if this throws, release() never called
    lock.release()                       # never reached

# Every thread that calls broken_increment after the exception will block forever
# trying to acquire the lock. Your program deadlocks silently.

# CORRECT — with statement guarantees release even on exception
def safe_increment():
    global counter
    with lock:
        counter += might_raise_exception()   # exception? lock still released
```

---

### Wrong vs Right — Using `if` instead of `while` in Condition

python

```python
condition = threading.Condition()
queue = []

# WRONG — spurious wakeup or race condition can make this fail
def consumer_broken():
    with condition:
        if not queue:          # checks once, then proceeds unconditionally
            condition.wait()
        item = queue.pop(0)    # queue might STILL be empty here — IndexError

# Scenario that breaks it:
# 1. Two consumers both wake up from condition.wait()
# 2. First consumer takes the item
# 3. Second consumer reaches queue.pop(0) — queue is empty — IndexError

# CORRECT — always re-check in a while loop
def consumer_correct():
    with condition:
        while not queue:       # re-check after every wakeup
            condition.wait()
        item = queue.pop(0)    # safe — while loop guarantees queue is non-empty
```

---

### Wrong vs Right — Reading shared state without a lock

python

```python
# Shared state written by background thread
status = {"abc100_a": False, "abc100_b": False}
lock = threading.Lock()

# WRONG — main thread reads while background thread might be mid-write
def display_loop():
    while True:
        for pid, solved in status.items():   # background thread modifying status
            print(pid, solved)               # simultaneously? RuntimeError or stale data
        time.sleep(1)

# CORRECT — always read inside the lock
def display_loop():
    while True:
        with lock:
            snapshot = dict(status)    # take a copy inside the lock
        for pid, solved in snapshot.items():   # iterate the copy outside the lock
            print(pid, solved)
        time.sleep(1)
```

Note the pattern: take a copy inside the lock, then do the slow/iterative work outside the lock. This keeps the lock held for the minimum amount of time — you don't want to hold the lock while printing or doing anything slow, because the background thread would be blocked for that entire duration.

This is exactly what your `snapshot()` does:

python

```python
# line 846-848
def snapshot(self):
    with self._lock:
        return dict(self.status), dict(self.solve_times), self.last_check
        # ^^^^ copies made inside lock, returned and used outside lock
```

---

### The `_loop` / `_do_check` split — why two methods

python

```python
def _loop(self):
    self._do_check()              # first check immediately on start
    while not self._stop.is_set():
        slept = 0
        while slept < CHECK_INTERVAL and not self._stop.is_set():
            self._stop.wait(1)
            slept += 1
        if not self._stop.is_set():
            self._do_check()

def _do_check(self):
    check_started = datetime.now()
    with self._lock:
        self.last_check = check_started
    try:
        fresh = check_solved(self.problems)
        elapsed = (datetime.now() - self.start_dt).total_seconds()
        with self._lock:
            for pid, solved in fresh.items():
                if solved and not self.status[pid]:
                    self.solve_times[pid] = elapsed
                self.status[pid] = solved
    except Exception:
        pass
```

**Why not put everything in `_loop` directly?**

There are two reasons:

**Reason 1 — called from two places.** The first check happens immediately when the tracker starts (before the first sleep interval). The subsequent checks happen after each sleep. If everything were in `_loop`, you'd have to duplicate the check logic or restructure the loop awkwardly. Extracting `_do_check` means the check logic is written once and called from both places cleanly.

**Reason 2 — exception isolation.** `_do_check` has a broad `except Exception: pass`. This ensures that any crash inside the check (network error, bad JSON, KeyError) never propagates up to `_loop` and kills the background thread. The thread keeps running and will try again on the next interval. If the check logic were inside `_loop` without this isolation, an unhandled exception would terminate the thread silently — your solve status would stop updating and you'd have no idea why.

The pattern in full:

python

```python
# Clean version of the pattern for reference
def _loop(self):
    self._do_check()                        # immediate first check
    while not self._stop.is_set():          # loop until stopped
        self._stop.wait(CHECK_INTERVAL)     # sleep (interruptible)
        if not self._stop.is_set():         # don't check if we were stopped
            self._do_check()

# Your actual code uses 1-second increments instead of wait(CHECK_INTERVAL)
# for more responsive stop behaviour — same idea, finer granularity
```

The `if not self._stop.is_set()` guard before `_do_check()` at the end of the sleep is important — if `stop()` was called during the sleep, you don't want to fire off one final network request as the program is shutting down.