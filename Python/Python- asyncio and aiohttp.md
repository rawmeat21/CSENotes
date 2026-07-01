### What is Concurrency vs Parallelism

Before anything else, these two words need to be precisely defined because they're constantly confused.

**Parallelism** — multiple operations executing at the exact same instant on multiple CPU cores. True simultaneous execution. Python threads don't give you this for CPU-bound work because of the GIL, but they do give it for I/O-bound work because the GIL releases during I/O.

**Concurrency** — multiple operations are in progress at the same time, but not necessarily executing simultaneously. One operation starts, hits a waiting point, gets suspended, and another operation runs while the first is waiting. When the first operation's wait is over, it resumes. Only one thing is actually executing at any given instant, but many things are making progress.

![[Pasted image 20260701202026.png]]

![[Pasted image 20260701202156.png]]

`asyncio` is a concurrency model. It runs in a single thread. Multiple operations make progress by yielding control to each other at waiting points.

---

`asyncio` is a Python library for writing concurrent code.

In syncronous code execution, one thing happens after another.
In asynchronous code execution, the code doesn't wait for IO bound tasks, instead it executes other tasks when current task goes for IO.
### The Problem asyncio Solves

Look at how your current fetchers work:

python

```python
# cf_fetch_problems calls _req twice — sequentially
solved_data = _req("https://codeforces.com/api/user.status?...")   # waits ~1s
pool_data   = _req("https://codeforces.com/api/problemset.problems") # waits ~1s
```

The second request doesn't start until the first one finishes. Total time: ~2 seconds.

In `mixed_fetch_problems`:

python

```python
def mixed_fetch_problems(tags=None):
    t1, t2 = [], []
    for fn in (lc_fetch_problems, cf_fetch_problems):
        a, b = fn(tags)    # lc finishes entirely before cf even starts
        t1 += a
        t2 += b
```

LeetCode fetching (30 pages × ~0.5s each = ~15 seconds) completes entirely before Codeforces even starts. Total fetch time: 15s + 2s = ~17 seconds.

While `_req` is waiting for the network response — which is most of the time — your program is doing absolutely nothing. The CPU is idle. asyncio lets you use that idle time to make other requests.

With asyncio, all requests can be in-flight simultaneously:

- Send CF user.status request → don't wait, immediately send next request
- Send CF problemset request → don't wait, immediately send next request
- Send LC page 1 request → don't wait, immediately send next
- Send LC page 2 request → don't wait...
- ...
- All responses arrive roughly simultaneously
- Total time: ~max(individual request times) instead of ~sum

---

### What asyncio Actually Is

`asyncio` is a Python standard library module that implements an **event loop** — a scheduler that runs coroutines.

To understand it you need to understand three things in order: coroutines, the event loop, and then how they work together.

---

![[Pasted image 20260701202420.png]]

![[Pasted image 20260701202551.png]]
![[Pasted image 20260701202624.png]]

Multithreading means the OS gives you multiple threads of execution that can genuinely run **in parallel** (on multi-core CPUs) or be interleaved by the OS scheduler (preemptively, without your code's consent).

- Each thread has its own call stack
- The OS decides when to pause/resume a thread — **you don't control it**
- True parallelism is possible: two threads can execute instructions on two different CPU cores at the exact same instant
- The cost: shared memory + preemption = race conditions, need for locks/mutexes, deadlocks, etc.

Async typically runs on a **single thread** with an event loop. There's no OS-level parallelism happening; instead, your code **cooperatively yields control** at specific points (usually `await`), letting the event loop run something else while waiting on I/O.

- One call stack (per event loop), not many
- Switches only happen at explicit `await`/yield points — **never mid-instruction**
- No true parallelism
- No race conditions in the same way, because only one piece of your code is ever actually running at a given instant

### Why not just use threads? They seem to do the same thing.

### Threads are expensive, tasks are cheap

- Typically **1-8 MB of stack memory** reserved per thread
- Creating one means a **system call into the kernel**
- Switching between threads is a **context switch**

### OS can preempt a thread **mid-instruction**

At any point, without your code's permission. If two threads touch shared state, you need locks, or you get race conditions.

For async tasks, control only switches at explicit `await` points, which _you_ put in your code. Nothing else runs until you hit one. So within a single event loop, you don't need locks for state shared between tasks — only one task's code is ever "live" at once.

**Threads are actually better for CPU bound work.**


### Why use threads even? Just use async?

For CPU-bound work in Python, **neither** threads nor async help. 

What you actually want there is `multiprocessing` — separate OS processes, each with its own Python interpreter and own GIL, giving you real parallelism across cores. 

Also, **Legacy / blocking libraries that don't support async.** 

Async requires the entire call chain to be non-blocking — every library you call has to be `async`-aware (`aiohttp` instead of `requests`, `asyncpg` instead of `psycopg2`, etc.). 

If you're stuck with a blocking library that has no async equivalent, threads let you get concurrency anyway without rewriting the library.


![[Pasted image 20260701202656.png]]

### Coroutines (cooperative function)

A coroutine is a function that can **suspend its own execution** at specific points and **resume later** from exactly where it left off. Normal functions run to completion without interruption. Coroutines can pause mid-execution.

You define a coroutine with `async def`:

python

```python
async def my_coroutine():
    print("start")
    await something()    # suspend here, resume when something() is done
    print("end")
```

The `async def` keyword makes the function a coroutine function. Calling it does NOT run it:

python

```python
async def greet():
    print("hello")

result = greet()   # does NOT print "hello"
print(result)      # <coroutine object greet at 0x...>
```

Calling a coroutine function returns a **coroutine object** — a suspended computation that hasn't started yet. To actually run it, you either `await` it or pass it to the event loop.

![[Pasted image 20260701210615.png]]

How to create event loop object:

![[Pasted image 20260701210907.png]]


---

### `await`

`await` is the suspension point. When a coroutine reaches an `await` expression:

1. It suspends itself
2. Control returns to the event loop
3. The event loop runs other coroutines
4. When the awaited thing completes, the event loop resumes the suspended coroutine


```python
async def fetch():
    print("about to wait")
    await asyncio.sleep(1)    # suspend here for 1 second
    print("resumed after 1 second")
```


`await` can only be used inside `async def` functions. Using it outside is a `SyntaxError`.

You can't just slap `await` in front of anything:

python

```python
await print("hello")   # ❌ TypeError: not awaitable
```

`await` specifically works on an **awaitable object** — something that represents "an operation that will eventually produce a result, but isn't done yet." In practice, this is almost always one of:

1. A **coroutine** — the result of calling an `async def` function
2. A **Task** — a coroutine that's already been scheduled to run
3. A **Future** — a lower-level "promise" that something will eventually complete

---

### The Event Loop

The event loop is the engine that runs coroutines. It maintains a queue of coroutines that are ready to run. It runs one at a time. When a coroutine hits an `await`, it's removed from "running" and put into "waiting". The event loop picks the next ready coroutine and runs it. When the awaited operation completes, the coroutine moves back to "ready".

```
Event Loop:
┌─────────────────────────────────────────────┐
│                                             │
│  Ready queue: [coro_A, coro_C]              │
│  Waiting:     [coro_B (waiting for network)]│
│                                             │
│  Currently running: coro_A                  │
│    → coro_A hits await                      │
│    → coro_A moves to waiting                │
│    → pick next from ready: coro_C           │
│    → run coro_C                             │
│    → network responds for coro_B            │
│    → coro_B moves to ready                  │
│    → ...                                    │
└─────────────────────────────────────────────┘
```

This is all single-threaded. There's no OS thread switching, no GIL contention, no race conditions on the event loop itself. You explicitly yield control with `await`.


This is basically what the event loop thread does:
```python
while True:
    task = ready_queue.pop_next()   # grab a task that's ready to run
    task.run_until_next_await()      # run it until it hits an await or finishes
    # if it awaited something, register what it's waiting for,
    # then move on to the next ready task
```

#### Running the event loop

python

```python
import asyncio

async def main():
    print("hello from async")

# Python 3.7+
asyncio.run(main())   # creates event loop, runs main(), closes loop
```

`asyncio.run()` is the standard entry point. It creates a new event loop, runs the given coroutine until it completes, then closes the loop. You call it once, at the top level of your program.


### What's actually happening when you call `asyncio.run()`?

1. Your main thread calls `asyncio.run(main())`
2. This creates an event loop **object** (just a Python object — a queue, a scheduler)
3. Your main thread then enters that `while True`-style loop I described earlier
4. Your main thread is now _busy running the loop itself_ — **it doesn't return control back to "regular" code until the loop finishes**

---

### `asyncio.sleep`

The async equivalent of `time.sleep`. Unlike `time.sleep` which blocks the entire thread, `asyncio.sleep` suspends only the current coroutine and lets the event loop run other coroutines during the wait.

python

```python
import asyncio

async def task(name, delay):
    print(f"{name}: starting")
    await asyncio.sleep(delay)    # suspend THIS coroutine, others can run
    print(f"{name}: done after {delay}s")

async def main():
    # Run sequentially — total time ~3s
    await task("A", 1)
    await task("B", 2)

asyncio.run(main())
```

Output:

```
A: starting
A: done after 1s
B: starting
B: done after 2s
```

This is still sequential because we `await` each task one at a time. To run them concurrently we need `asyncio.gather` or `asyncio.create_task`.

---

### Example:

```python
async def get_movie_tickets():
    await asyncio.sleep(7)
    print('Got the movie tickets')

async def like_ig():
    await asyncio.sleep(3)
    print('Finshed Instagram')

async def main():
    task1 = asyncio.create_task(get_movie_tickets())
    task2 = asyncio.create_task(like_ig())
    await task1

asyncio.run(main())
```

### Timeline of what actually happens

```
t=0: main() starts
t=0: task1 created & scheduled (get_movie_tickets queued to run)
t=0: task2 created & scheduled (like_ig queued to run)
t=0: main hits `await task1` → main suspends, hands control to the loop
t=0: loop runs task1 → hits `await asyncio.sleep(7)` → task1 suspends, registers a 7s timer, loop moves on
t=0: loop runs task2 → hits `await asyncio.sleep(3)` → task2 suspends, registers a 3s timer, loop moves on
t=0: loop now has nothing ready to run; it blocks until a timer fires
t=3: task2's timer fires → task2 resumes → prints 'Finshed Instagram' → task2 completes
t=7: task1's timer fires → task1 resumes → prints 'Got the movie tickets' → task1 completes
t=7: main was awaiting task1, so task1 completing makes main ready again → main resumes → main returns
```

Total time: **~7 seconds**

#### What `await` does to the event loop's queue, precisely

When `main` executes `await task1`:

1. `main`'s coroutine is **paused** and removed from the "ready to run" set
2. It's registered as **waiting on task1's completion** — specifically, task1 keeps a list of callbacks to fire when it's done, and this registers one that will re-add `main` to the ready queue
3. Control returns to the event loop, which picks whatever else is ready (here: run task1, then task2, until each hits its own `await`)
4. When task1 eventually finishes, the loop fires that registered callback, which pushes `main` back onto the ready queue
5. Next time the loop gets to `main` in the queue, it resumes exactly where `await task1` left off

### If you remove `await task1`

```python
async def main():
    task1 = asyncio.create_task(get_movie_tickets())
    task2 = asyncio.create_task(like_ig())
    # no await
```

`main()` would now run to completion **immediately**, without ever yielding control back to the event loop (no `await` inside `main` means nothing ever suspends it). Since `main()` finishing is what makes `asyncio.run()` return, the loop closes right after `main` returns — **before task1 or task2 ever get a chance to run a single line**, since they were only _scheduled_, not yet executed.


### Two different things `await` can be waiting on

**1. Awaiting a bare coroutine** (no `create_task`):

python

```python
async def main():
    await get_movie_tickets()   # not scheduled until this line runs
    await like_ig()
```

Here, nothing happens until `await` actually runs. `get_movie_tickets()` isn't executing in the background before that — it doesn't even start until you `await` it. So this is fully sequential: 7s, then 3s more = 10s total. `await` here means "start this now, and don't move past this line until it's done." Basically, here we actually don't do any concurrency.

**2. Awaiting a Task** (created via `create_task`):

python

```python
async def main():
    task1 = asyncio.create_task(get_movie_tickets())  # starts running NOW
    task2 = asyncio.create_task(like_ig())              # starts running NOW
    await task1   # just wait for something already in progress
```

Here, the work already started the moment you called `create_task` — `await task1` isn't starting anything, it's just saying "pause me until this thing that's _already running_ finishes." That's why the two sleeps overlap here (7s total), but not in the first example.


`await` means: **"suspend me until this awaitable reaches completion — starting it now if it hasn't started yet, or just waiting if it's already running."**

---


### `asyncio.gather`

`asyncio.gather(*coroutines)` schedules multiple coroutines to run concurrently and waits for all of them to complete. It returns a list of their results in the same order as the input.

python

```python
import asyncio

async def task(name, delay):
    print(f"{name}: starting")
    await asyncio.sleep(delay)
    print(f"{name}: done")
    return f"{name}-result"

async def main():
    # Run concurrently — total time ~2s (max of 1, 2), not ~3s (sum)
    results = await asyncio.gather(
        task("A", 1),
        task("B", 2),
        task("C", 1),
    )
    print(results)   # ['A-result', 'B-result', 'C-result']

asyncio.run(main())
```

Output:

```
A: starting
B: starting
C: starting
A: done        ← after 1s
C: done        ← also after 1s
B: done        ← after 2s
['A-result', 'B-result', 'C-result']
```

All three started almost simultaneously. Total time was ~2 seconds (the longest task), not ~4 seconds (the sum). This is exactly the speedup your fetchers would get.

![[Pasted image 20260701220421.png]]

gather() does this:

- **Wraps each argument into a Task** if it isn't already one. So passing bare coroutines is fine.
- **Schedules all of them to start running immediately**, concurrently.
- **Awaits all of them**, suspending the calling coroutine until every single one completes.
- **Returns results in the same order you passed the arguments in** — not the order they finished.

#### `gather` with `return_exceptions=True`

By default, if any coroutine raises an exception, `gather` immediately cancels the others and propagates the exception. With `return_exceptions=True`, exceptions are returned as results instead of being raised — you get the exception object in the results list at the position of the failing coroutine.

python

```python
async def might_fail(n):
    if n == 2:
        raise ValueError(f"failed on {n}")
    return n * 10

async def main():
    # Default — exception propagates, other results lost
    try:
        results = await asyncio.gather(
            might_fail(1),
            might_fail(2),
            might_fail(3),
        )
    except ValueError as e:
        print(f"Error: {e}")   # Error: failed on 2

    # return_exceptions=True — collect everything
    results = await asyncio.gather(
        might_fail(1),
        might_fail(2),
        might_fail(3),
        return_exceptions=True
    )
    print(results)
    # [10, ValueError('failed on 2'), 30]

    for r in results:
        if isinstance(r, Exception):
            print(f"this one failed: {r}")
        else:
            print(f"result: {r}")

asyncio.run(main())
```

For network requests where individual failures are acceptable — like your fetchers where one failed page shouldn't abort everything — `return_exceptions=True` is the right choice.

---

### `asyncio.create_task`

`create_task` schedules a coroutine as a **Task** that starts running immediately (on the next event loop iteration), without needing to `await` it right away. This lets you start something and then do other things before collecting its result.

python

```python
async def main():
    # create_task schedules immediately — both start before either is awaited
    task_a = asyncio.create_task(fetch_something("A"))
    task_b = asyncio.create_task(fetch_something("B"))

    # do other work here while A and B are running...

    result_a = await task_a   # wait for A to finish
    result_b = await task_b   # wait for B to finish (probably already done)
```

vs `gather` which is cleaner when you just want to run a batch concurrently and collect all results:

python

```python
results = await asyncio.gather(fetch_something("A"), fetch_something("B"))
```

For your use case (batching API requests), `gather` is cleaner. `create_task` is useful when you need more control — cancelling individual tasks, checking if they're done with `.done()`, adding callbacks with `.add_done_callback()`.

---

### What is aiohttp

`requests` is a synchronous HTTP library. When you call `requests.get()`, it blocks the current thread until the response arrives. This is incompatible with asyncio — a blocking call inside a coroutine blocks the entire event loop, preventing all other coroutines from running.

python

```python
async def bad_fetch(url):
    r = requests.get(url)    # BLOCKS THE ENTIRE EVENT LOOP
    return r.json()          # no other coroutine can run while this waits
```

`aiohttp` is an HTTP library built on asyncio. Its network calls are coroutines — they suspend at `await` points and let the event loop run other coroutines during the wait.

Install:

```
pip install aiohttp
```

---

### aiohttp — Core Concepts

#### `ClientSession`

The central object in aiohttp. It manages connections, cookies, headers, and connection pooling. You create one session and reuse it for all requests — this is important because it allows connection reuse (keep-alive), which is faster than opening a new TCP connection for every request.

python

```python
import aiohttp
import asyncio

async def main():
    async with aiohttp.ClientSession() as session:
        # make requests using session
        async with session.get("https://example.com") as response:
            data = await response.json()
            print(data)

asyncio.run(main())
```

Two `async with` statements:

- `async with aiohttp.ClientSession()` — creates the session, closes it and all connections when done
- `async with session.get(url)` — sends the request, receives the response, closes the response when done

`async with` is the async version of `with`. It calls `__aenter__` on entry and `__aexit__` on exit. The `__aexit__` calls are awaitable, which is why the sync `with` can't be used here.

#### Making GET and POST requests

python

```python
async with aiohttp.ClientSession() as session:

    # GET
    async with session.get(url, headers=headers, timeout=timeout) as r:
        data = await r.json()

    # POST with JSON body
    async with session.post(url, headers=headers, json=payload) as r:
        data = await r.json()

    # Check status
    async with session.get(url) as r:
        print(r.status)          # integer status code — note: r.status not r.status_code
        r.raise_for_status()     # same as requests — raises aiohttp.ClientResponseError
        text = await r.text()    # response body as string
        data = await r.json()    # response body parsed as JSON
```

Key difference from `requests`: in aiohttp, reading the response body (`r.json()`, `r.text()`) is also async — you must `await` it. The response headers arrive first, then the body streams in. `await r.json()` waits for the full body to arrive and parses it.

#### Timeouts

python

```python
import aiohttp
import asyncio

timeout = aiohttp.ClientTimeout(total=15)   # 15 second total timeout

async with aiohttp.ClientSession(timeout=timeout) as session:
    async with session.get(url) as r:
        data = await r.json()
```

`aiohttp.ClientTimeout` accepts:

- `total` — total time for the entire request including connection and reading
- `connect` — time to establish the connection
- `sock_read` — time to read the response

On timeout, raises `asyncio.TimeoutError`.

---

### Rewriting `_req` as an async function

Here is the current synchronous `_req`:

python

```python
def _req(url, method="GET", headers=None, json_data=None, retries=3):
    for attempt in range(retries):
        try:
            if method == "POST":
                r = requests.post(url, headers=headers, json=json_data, timeout=15)
            else:
                r = requests.get(url, headers=headers, timeout=12)
            r.raise_for_status()
            return r.json()
        except requests.exceptions.HTTPError as e:
            sc = e.response.status_code
            if sc in (429, 500, 502, 503, 504):
                time.sleep(2 ** attempt)
            else:
                return None
        except Exception:
            time.sleep(2 ** attempt)
    return None
```

Here is the async equivalent using aiohttp:

python

```python
import aiohttp
import asyncio

async def _req_async(session, url, method="GET", headers=None, json_data=None, retries=3):
    for attempt in range(retries):
        try:
            if method == "POST":
                async with session.post(url, headers=headers, json=json_data) as r:
                    r.raise_for_status()
                    return await r.json()
            else:
                async with session.get(url, headers=headers) as r:
                    r.raise_for_status()
                    return await r.json()
        except aiohttp.ClientResponseError as e:
            if e.status in (429, 500, 502, 503, 504):
                await asyncio.sleep(2 ** attempt)   # async sleep — doesn't block event loop
            else:
                return None
        except Exception:
            await asyncio.sleep(2 ** attempt)
    return None
```

Key differences:

- `async def` instead of `def`
- Takes a `session` parameter — the `ClientSession` is created once and passed in
- `async with session.get(...)` instead of `requests.get(...)`
- `await r.json()` instead of `r.json()`
- `await asyncio.sleep(...)` instead of `time.sleep(...)` — critical: `time.sleep` in an async function blocks the entire event loop
- `aiohttp.ClientResponseError` instead of `requests.exceptions.HTTPError`
- `e.status` instead of `e.response.status_code`

---

### Rewriting `cf_fetch_problems` as async

Current synchronous version (simplified):

python

```python
def cf_fetch_problems(tags=None):
    solved_data = _req(f"https://codeforces.com/api/user.status?handle={CF_HANDLE}&from=1&count=10000")
    pool_data   = _req("https://codeforces.com/api/problemset.problems")
    ...
```

These two requests run sequentially. With async:

python

```python
async def cf_fetch_problems_async(session, tags=None):
    # Both requests fire simultaneously
    solved_data, pool_data = await asyncio.gather(
        _req_async(session, f"https://codeforces.com/api/user.status?handle={CF_HANDLE}&from=1&count=10000"),
        _req_async(session, "https://codeforces.com/api/problemset.problems"),
    )
    # rest of the logic is identical — just data processing, no I/O
    ...
```

Both requests are in-flight at the same time. The result arrives when both complete. The filtering/processing logic after this point doesn't change at all — it's pure Python data manipulation, no I/O, doesn't need to be async.

---

### Rewriting `lc_fetch_problems` — the big win

This is where async has the most dramatic effect. Currently:

python

```python
for page in range(30):    # 30 sequential requests
    data = _req("https://leetcode.com/graphql", method="POST", ...)
```

30 requests × ~0.5s each = ~15 seconds sequentially.

With async, all 30 requests fire simultaneously:

python

```python
async def lc_fetch_problems_async(session, tags=None):
    hdrs = {
        "content-type": "application/json",
        "cookie": f"LEETCODE_SESSION={LC_SESSION_COOKIE}",
    }

    async def fetch_page(page):
        variables = {
            "categorySlug": "",
            "skip": page * 100,
            "limit": 100,
            "filters": {"orderBy": "FRONTEND_ID", "sortOrder": "DESCENDING", "premiumOnly": False},
        }
        return await _req_async(
            session,
            "https://leetcode.com/graphql",
            method="POST",
            headers=hdrs,
            json_data={"query": query, "variables": variables}
        )

    # Fire all 30 page requests simultaneously
    results = await asyncio.gather(
        *[fetch_page(page) for page in range(30)],
        return_exceptions=True
    )

    questions = []
    for data in results:
        if isinstance(data, Exception) or not data or "errors" in data:
            continue
        batch = data["data"]["problemsetQuestionList"]["data"]
        questions.extend(batch)
    ...
```

30 requests simultaneously → total time ~0.5s instead of ~15s.

`return_exceptions=True` is important here — if one page fails, we still get the other 29 results. We skip failed pages with the `isinstance(data, Exception)` check.

---

### Rewriting `mixed_fetch_problems` — combining everything

The biggest gain. Currently CF and LC fetch sequentially:

python

```python
def mixed_fetch_problems(tags=None):
    for fn in (lc_fetch_problems, cf_fetch_problems):
        a, b = fn(tags)
        ...
```

With async, both platforms fetch simultaneously:

python

```python
async def mixed_fetch_problems_async(tags=None):
    async with aiohttp.ClientSession(timeout=aiohttp.ClientTimeout(total=30)) as session:
        (lc_t1, lc_t2), (cf_t1, cf_t2) = await asyncio.gather(
            lc_fetch_problems_async(session, tags),
            cf_fetch_problems_async(session, tags),
        )
    return lc_t1 + cf_t1, lc_t2 + cf_t2
```

LC and CF fetch in parallel. Total time ≈ max(LC time, CF time) instead of LC time + CF time.

---

### The top-level entry point

Since your `prompt_setup` is synchronous (it uses `input()` for user prompts), you'd call the async fetcher like this:

python

```python
def prompt_setup():
    ...
    print(clr(f"  Fetching problems from {platform.upper()}...", C.YELLOW))

    if platform == "cf":
        t1, t2 = asyncio.run(cf_fetch_problems_async_wrapper(tags))
    elif platform == "lc":
        t1, t2 = asyncio.run(lc_fetch_problems_async_wrapper(tags))
    else:
        t1, t2 = asyncio.run(mixed_fetch_problems_async(tags))
```

Where each wrapper creates the session and calls the async fetcher:

python

```python
async def cf_fetch_problems_async_wrapper(tags=None):
    async with aiohttp.ClientSession(timeout=aiohttp.ClientTimeout(total=20)) as session:
        return await cf_fetch_problems_async(session, tags)
```

`asyncio.run()` bridges the synchronous world (your `prompt_setup` function) and the async world (the fetchers). You call it once, it runs the event loop until the coroutine completes, and returns the result synchronously.

---

### What You Cannot Do in Async Code

#### Blocking calls inside coroutines

python

```python
# WRONG — blocks the entire event loop, all other coroutines freeze
async def bad():
    r = requests.get(url)      # synchronous — blocks event loop
    time.sleep(2)              # synchronous — blocks event loop
    data = open("file").read() # synchronous file I/O — blocks event loop

# CORRECT
async def good():
    async with session.get(url) as r:   # async — suspends, others run
        data = await r.json()
    await asyncio.sleep(2)              # async — suspends, others run
```

If you must call a blocking function from async code, use `asyncio.run_in_executor` which runs it in a thread pool so it doesn't block the event loop:

python

```python
import asyncio

async def call_blocking():
    loop = asyncio.get_event_loop()
    result = await loop.run_in_executor(None, requests.get, url)
    # None means default ThreadPoolExecutor
```

#### Mixing `asyncio.run` with an already-running event loop

python

```python
# WRONG — asyncio.run creates a NEW event loop, can't nest them
async def outer():
    asyncio.run(inner())    # RuntimeError: This event loop is already running

# CORRECT — just await the coroutine directly
async def outer():
    await inner()
```

---

### asyncio vs threading — when to use which

||`threading`|`asyncio`|
|---|---|---|
|Execution model|Multiple OS threads, preemptive scheduling|Single thread, cooperative scheduling|
|Switching|OS decides when to switch (preemptive)|You decide when to yield (cooperative, via `await`)|
|Race conditions|Yes — need locks|No — only one coroutine runs at a time|
|I/O bound work|Good|Excellent — lower overhead, more scalable|
|CPU bound work|Limited by GIL|No benefit — still single-threaded|
|Existing sync libraries|Use directly|Need async versions (aiohttp instead of requests)|
|Code complexity|Familiar sync style|Requires async/await throughout|

For your CP tool:

- **`SolveTracker`** stays as a thread — it runs a long-lived background loop and the threading model is clean and already working
- **Fetchers** are the right candidate for async — they make many independent I/O calls that are currently sequential and would benefit enormously from concurrency

---

### Full working example — async batch fetch

This is a complete, runnable example showing the entire pattern end to end:

python

```python
import aiohttp
import asyncio

async def fetch_url(session, url, name):
    try:
        async with session.get(url, timeout=aiohttp.ClientTimeout(total=10)) as r:
            r.raise_for_status()
            data = await r.json()
            return name, data
    except Exception as e:
        return name, None

async def fetch_all():
    urls = {
        "cf_problems":    "https://codeforces.com/api/problemset.problems",
        "cf_user":        f"https://codeforces.com/api/user.status?handle={CF_HANDLE}&from=1&count=100",
        "ac_problems":    "https://kenkoooo.com/atcoder/resources/problems.json",
        "ac_models":      "https://kenkoooo.com/atcoder/resources/problem-models.json",
    }

    async with aiohttp.ClientSession() as session:
        results = await asyncio.gather(
            *[fetch_url(session, url, name) for name, url in urls.items()],
            return_exceptions=True
        )

    return {name: data for name, data in results if data is not None}

data = asyncio.run(fetch_all())
print(data.keys())
# All 4 requests fired simultaneously — total time ~max(individual times)
```

---