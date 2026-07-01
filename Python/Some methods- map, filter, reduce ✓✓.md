
## `map`, `filter`, `reduce` in Python

---

### The Concept

All three take a **function and an iterable** and apply the function to produce a result. They are tools for processing collections without writing explicit loops. In Python they come from functional programming influence.

---

### `map`

Applies a function to **every element** of an iterable and returns an iterator of the results.

python

```python
map(function, iterable)
```

python

```python
nums = [1, 2, 3, 4, 5]

squared = map(lambda x: x**2, nums)
list(squared)   # [1, 4, 9, 16, 25]
```

`map` returns a **map object** — a lazy iterator, not a list. It computes values on demand. Wrap with `list()` to materialise it.

#### Multiple iterables

`map` accepts multiple iterables. The function must take as many arguments as there are iterables. It stops at the shortest:

python

```python
a = [1, 2, 3]
b = [10, 20, 30]

list(map(lambda x, y: x + y, a, b))   # [11, 22, 33]
```

#### With a named function

python

```python
def double(x):
    return x * 2

list(map(double, [1, 2, 3]))   # [2, 4, 6]
```

#### `map` vs list comprehension

These are equivalent:

python

```python
list(map(lambda x: x**2, nums))
[x**2 for x in nums]
```

List comprehension is generally preferred in Python for readability. `map` is useful when you already have a named function to pass — no need for a lambda:

python

```python
words = ["hello", "world"]

list(map(str.upper, words))       # ['HELLO', 'WORLD']
list(map(int, ["1", "2", "3"]))   # [1, 2, 3]
```

Passing `str.upper` directly — no lambda needed. This is where `map` is cleaner than a comprehension.

---

### `filter`

Applies a **predicate function** (a function that returns `True` or `False`) to each element and returns an iterator of elements for which the predicate returned `True`.

python

```python
filter(function, iterable)
```

python

```python
nums = [1, 2, 3, 4, 5, 6]

evens = filter(lambda x: x % 2 == 0, nums)
list(evens)   # [2, 4, 6]
```

Like `map`, returns a lazy iterator.

#### `None` as the function

Passing `None` uses truthiness as the filter — removes falsy values (`0`, `""`, `None`, `[]`, `False`):

python

```python
values = [0, 1, "", "hello", None, 42, [], [1, 2]]

list(filter(None, values))   # [1, 'hello', 42, [1, 2]]
```

#### `filter` vs list comprehension

python

```python
list(filter(lambda x: x % 2 == 0, nums))
[x for x in nums if x % 2 == 0]
```

Again, comprehension is generally more readable. `filter` is clean when you have a named predicate:

python

```python
def is_positive(x):
    return x > 0

nums = [-2, -1, 0, 1, 2]
list(filter(is_positive, nums))   # [1, 2]
```

---

### `reduce`

Applies a function **cumulatively** to elements, reducing the entire iterable to a single value.

Unlike `map` and `filter`, `reduce` is not a built-in — it lives in `functools`:



```python
from functools import reduce

reduce(function, iterable, initial)
```

The function takes **two arguments**: the accumulated value so far, and the current element. It returns the new accumulated value.



```python
from functools import reduce

nums = [1, 2, 3, 4, 5]

total = reduce(lambda acc, x: acc + x, nums)   # 15
```

Step by step:

```
start:    acc=1,  x=2  →  1+2  =  3
step 2:   acc=3,  x=3  →  3+3  =  6
step 3:   acc=6,  x=4  →  6+4  =  10
step 4:   acc=10, x=5  →  10+5 =  15
```

The first element is used as the initial accumulator if no `initial` is provided. If the iterable is empty and no `initial` is given, `TypeError` is raised.

#### With an initial value

python

```python
reduce(lambda acc, x: acc + x, [1, 2, 3], 100)   # 106
```

`initial` is the starting accumulator value. Also protects against empty iterable:

python

```python
reduce(lambda acc, x: acc + x, [], 0)   # 0  — no error
```

#### More examples

python

```python
# product of all elements
reduce(lambda acc, x: acc * x, [1, 2, 3, 4])   # 24

# maximum
reduce(lambda acc, x: acc if acc > x else x, [3, 1, 4, 1, 5, 9])   # 9

# flatten
reduce(lambda acc, x: acc + x, [[1,2],[3,4],[5,6]])   # [1, 2, 3, 4, 5, 6]
```

#### `reduce` vs built-ins

For common reductions, Python has dedicated built-ins that are clearer and faster:

python

```python
sum([1, 2, 3, 4, 5])          # instead of reduce(lambda a,x: a+x, ...)
max([3, 1, 4, 1, 5])          # instead of reduce(lambda a,x: a if a>x else x, ...)
min([3, 1, 4, 1, 5])
any(x > 3 for x in nums)
all(x > 0 for x in nums)
```

Use `reduce` when no built-in covers your specific accumulation logic.

---

### Chaining All Three

python

```python
from functools import reduce

nums = range(1, 11)   # 1 to 10

result = reduce(
    lambda acc, x: acc + x,
    map(
        lambda x: x**2,
        filter(
            lambda x: x % 2 == 0,
            nums
        )
    )
)

print(result)   # 220  →  4 + 16 + 36 + 64 + 100
```

Reading inside out:

1. `filter` — keep even numbers: `[2, 4, 6, 8, 10]`
2. `map` — square each: `[4, 16, 36, 64, 100]`
3. `reduce` — sum them: `220`

The equivalent comprehension form, which most Python developers would prefer:

python

```python
sum(x**2 for x in range(1, 11) if x % 2 == 0)   # 220
```

---

### When to Use Each

`map` — when you want to **transform** every element with a clean named function:

python

```python
list(map(int, string_numbers))
list(map(str.strip, lines))
```

`filter` — when you want to **select** elements with a clean named predicate:

python

```python
list(filter(str.isdigit, chars))
list(filter(None, values))        # remove falsy
```

`reduce` — when you want to **accumulate** into a single value and no built-in covers it:

python

```python
reduce(lambda acc, x: {**acc, **x}, list_of_dicts)   # merge dicts
```

For everything else — list comprehensions and generator expressions are idiomatic Python and more readable.