## Iterators and Generators in Python

---

### The Iteration Protocol

Before iterators make sense, you need to know what Python does under the hood when you write `for x in obj`.

Python requires two things from any object that can be iterated:

1. `__iter__()` — returns an **iterator object**
2. `__next__()` — returns the next value each time it is called, and raises `StopIteration` when there are no more values

An object that implements `__iter__` is called an **iterable**. An object that implements both `__iter__` and `__next__` is called an **iterator**.

When you write:

python

```python
for x in obj:
    print(x)
```

Python internally does exactly this:

python

```python
it = iter(obj)        # calls obj.__iter__()
while True:
    try:
        x = next(it)  # calls it.__next__()
        print(x)
    except StopIteration:
        break
```

`iter()` and `next()` are built-in functions that call the dunder methods. The `for` loop is syntactic sugar over this exact pattern.

---

### Iterables vs Iterators

These two terms are distinct:

- **Iterable** — has `__iter__`, returns an iterator when you call `iter()` on it. Lists, tuples, dicts, sets, strings are all iterables. You can loop over them. You can call `iter()` on them multiple times and get fresh iterators each time.
- **Iterator** — has both `__iter__` and `__next__`. Maintains internal state about where it currently is in the sequence. Once exhausted, it stays exhausted.

python

```python
lst = [1, 2, 3]         # iterable, not an iterator
it = iter(lst)          # iterator

next(it)   # 1
next(it)   # 2
next(it)   # 3
next(it)   # StopIteration

# iterables can produce fresh iterators
it1 = iter(lst)
it2 = iter(lst)   # completely independent iterator
```

An iterator is always an iterable — its `__iter__` just returns `self`. But an iterable is not necessarily an iterator.

```
Iterable: has __iter__
               ↓
           returns Iterator: has __iter__ + __next__
```

---

### Building an Iterator Manually

python

```python
class CountUp:
    def __init__(self, start, stop):
        self.current = start
        self.stop = stop

    def __iter__(self):
        return self         # the object itself is the iterator

    def __next__(self):
        if self.current >= self.stop:
            raise StopIteration
        val = self.current
        self.current += 1
        return val
```

python

```python
for n in CountUp(1, 5):
    print(n)
# 1 2 3 4
```

Step by step what happens:

1. `for` calls `iter(CountUp(1,5))` → `__iter__` returns `self`
2. `for` calls `next(it)` repeatedly → `__next__` returns values
3. When `current >= stop`, `__next__` raises `StopIteration`
4. `for` catches `StopIteration` and stops

This works but is verbose. Generator functions exist to replace exactly this pattern.

---

### Generator Functions

A **generator function** is a function that contains a `yield` statement. Calling it does not execute the body — it returns a **generator object**, which is an iterator.

python

```python
def count_up(start, stop):
    current = start
    while current < stop:
        yield current
        current += 1
```

python

```python
gen = count_up(1, 5)   # function body does NOT run yet
next(gen)              # 1 — runs until first yield, pauses
next(gen)              # 2 — resumes from after yield, runs to next yield
next(gen)              # 3
next(gen)              # 4
next(gen)              # StopIteration — function returned
```

#### What `yield` actually does

`yield` does two things simultaneously:

1. **Suspends** the function — freezes its entire execution state (local variables, instruction pointer, call stack frame)
2. **Produces** a value — sends that value to whoever called `next()`

When `next()` is called again, execution resumes from **exactly where it paused** — the line after `yield` — with all local variables intact.

```
def count_up(1, 5):
    current = 1
    while current < 5:
──→     yield current        ← pauses here, sends 1 out
        current += 1         ← resumes here on next next()
    while current < 5:
──→     yield current        ← pauses here, sends 2 out
        current += 1
    ...
    # function body ends → StopIteration raised automatically
```

This is fundamentally different from a regular function. A regular function runs to completion and returns once. A generator function can pause and resume multiple times.

---

### Why Generators Exist — Lazy Evaluation

A generator produces values **on demand** — it does not compute or store all values upfront.

Compare:

python

```python
def first_n_squares_list(n):
    return [x*x for x in range(n)]   # computes ALL n values, stores in memory

def first_n_squares_gen(n):
    for x in range(n):
        yield x*x                     # computes one value at a time
```

For `n = 1_000_000`:

- The list version allocates a list of 1 million integers immediately
- The generator version holds only the current `x` in memory at any time

This is **lazy evaluation** — values are computed only when requested. Generators are the mechanism Python uses for this.

`range()` itself behaves like a generator in this sense — it does not store all numbers.

---

### `yield from`

`yield from` delegates to another iterable or generator, yielding each of its values in sequence:

python

```python
def chain(a, b):
    yield from a
    yield from b

list(chain([1, 2], [3, 4]))   # [1, 2, 3, 4]
```

This is equivalent to:

python

```python
def chain(a, b):
    for x in a:
        yield x
    for x in b:
        yield x
```

`yield from` is cleaner and also handles more complex cases involving sub-generators sending values back (covered below).

Flattening nested structures:

python

```python
def flatten(lst):
    for item in lst:
        if isinstance(item, list):
            yield from flatten(item)   # recursive generator
        else:
            yield item

list(flatten([1, [2, [3, 4]], 5]))   # [1, 2, 3, 4, 5]
```

---

### Generator Expressions

Same as list comprehensions but with `()` instead of `[]`. Produces a generator instead of a list:

python

```python
squares_list = [x*x for x in range(10)]    # list — computed immediately
squares_gen  = (x*x for x in range(10))    # generator — lazy

next(squares_gen)   # 0
next(squares_gen)   # 1
```

When passed directly to a function that accepts an iterable, the outer parentheses can be dropped:

python

```python
sum(x*x for x in range(10))     # no double parentheses needed
max(x*x for x in range(10))
```

---

### Sending Values into a Generator

A generator is not just a one-way producer. You can send values **into** a running generator using `.send()`:

python

```python
def accumulator():
    total = 0
    while True:
        value = yield total    # yield sends total out AND receives the sent value
        total += value
```

python

```python
gen = accumulator()
next(gen)        # 0 — must prime the generator first (advance to first yield)
gen.send(10)     # 10 — sends 10 in, total becomes 10, yields 10
gen.send(5)      # 15
gen.send(3)      # 18
```

The `yield` expression here has two roles:

- Sends `total` out (as usual)
- Receives the sent value and assigns it to `value`

The first call must be `next(gen)` (or `gen.send(None)`) to advance the generator to the first `yield`. You cannot send a non-None value into a generator that hasn't started yet.

---

### Throwing Exceptions and Closing

#### `.throw(exc)`

Raises an exception inside the generator at the point where it is paused:

python

```python
def gen():
    try:
        yield 1
        yield 2
    except ValueError:
        yield 99

g = gen()
next(g)           # 1
g.throw(ValueError)  # 99 — exception caught inside generator
```

#### `.close()`

Throws a `GeneratorExit` exception into the generator, causing it to terminate. Called automatically when the generator is garbage collected:

python

```python
def gen():
    try:
        while True:
            yield 1
    finally:
        print("generator closed")   # runs on close()

g = gen()
next(g)
g.close()   # prints "generator closed"
```

---

### `return` in a Generator

A generator function can have a `return` statement. It does not yield a value — it causes `StopIteration` to be raised. The return value becomes the `value` attribute of the `StopIteration` exception:

python

```python
def gen():
    yield 1
    yield 2
    return "done"

g = gen()
next(g)   # 1
next(g)   # 2
try:
    next(g)
except StopIteration as e:
    print(e.value)   # "done"
```

With `yield from`, the return value of a sub-generator is the result of the `yield from` expression:

python

```python
def sub():
    yield 1
    return "sub done"

def main():
    result = yield from sub()
    print(result)    # "sub done"
    yield 2
```

---

### `itertools` — Standard Library for Iterators

Python's `itertools` module provides a collection of functions that work with iterators and return iterators (all lazy):

python

```python
import itertools
```

#### `itertools.count(start, step)`

Infinite counter:

python

```python
for n in itertools.count(10, 2):   # 10, 12, 14, 16, ...
    if n > 20: break
```

#### `itertools.cycle(iterable)`

Cycles through an iterable infinitely:

python

```python
for x in itertools.cycle([1, 2, 3]):   # 1, 2, 3, 1, 2, 3, ...
    ...
```

#### `itertools.repeat(val, n)`

Repeats a value `n` times (or infinitely if `n` omitted):

python

```python
list(itertools.repeat(0, 5))   # [0, 0, 0, 0, 0]
```

#### `itertools.chain(*iterables)`

Chains iterables sequentially:

python

```python
list(itertools.chain([1,2], [3,4], [5]))   # [1, 2, 3, 4, 5]
```

#### `itertools.islice(iterable, stop)`

Slices an iterator — works even on infinite ones:

python

```python
list(itertools.islice(itertools.count(), 5))   # [0, 1, 2, 3, 4]
```

#### `itertools.takewhile(pred, iterable)`

Takes elements while predicate is true, stops at first false:

python

```python
list(itertools.takewhile(lambda x: x < 5, [1, 3, 5, 2]))   # [1, 3]
```

#### `itertools.dropwhile(pred, iterable)`

Drops elements while predicate is true, yields the rest:

python

```python
list(itertools.dropwhile(lambda x: x < 5, [1, 3, 5, 2]))   # [5, 2]
```

#### `itertools.product(*iterables)`

Cartesian product:

python

```python
list(itertools.product([1,2], ['a','b']))
# [(1,'a'), (1,'b'), (2,'a'), (2,'b')]
```

#### `itertools.combinations(iterable, r)`

All r-length combinations without repetition:

python

```python
list(itertools.combinations([1,2,3], 2))
# [(1,2), (1,3), (2,3)]
```

#### `itertools.permutations(iterable, r)`

All r-length permutations:

python

```python
list(itertools.permutations([1,2,3], 2))
# [(1,2), (1,3), (2,1), (2,3), (3,1), (3,2)]
```
