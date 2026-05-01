### Creating Lists

```python
a = [1, 2, 3]                   # literal
b = list()                      # empty list via constructor
c = list("hello")               # ['h', 'e', 'l', 'l', 'o'] — iterates the string
d = list(range(5))              # [0, 1, 2, 3, 4]
e = [0] * 5                     # [0, 0, 0, 0, 0] — repetition
f = [[0] * 3 for _ in range(3)] # 3x3 2D list 
```
---
### Indexing and Slicing

Identical to strings — same rules apply since both are sequences

```python
a = [10, 20, 30, 40, 50]

a[0]      # 10
a[-1]     # 50
a[1:3]    # [20, 30]
a[::2]    # [10, 30, 50]
a[::-1]   # [50, 40, 30, 20, 10] — reversed
```
---

### Adding Elements

#### `.append(x)`

```python
a = [1, 2, 3]
a.append(4)     # [1, 2, 3, 4]
```

#### `.insert(i, x)`

Inserts `x` at index `i`. Everything from `i` onwards shifts right.

```python
a = [1, 2, 3]
a.insert(1, 99)   # [1, 99, 2, 3]
a.insert(0, 0)    # insert at beginning
a.insert(len(a), 99)  # equivalent to append
```

#### `.extend(iterable)`

Appends all elements from an iterable to the end. Modifies in place.

python

```python
a = [1, 2, 3]
a.extend([4, 5, 6])   # [1, 2, 3, 4, 5, 6]
a.extend(range(3))    # works with any iterable
```

Difference from `.append()`:

python

```python
a.append([4, 5])    # [1, 2, 3, [4, 5]]  ← nested list
a.extend([4, 5])    # [1, 2, 3, 4, 5]    ← elements added flat
```

#### `+` operator

Concatenation — creates a **new list**, does not modify either operand:

python

```python
a = [1, 2]
b = [3, 4]
c = a + b     # [1, 2, 3, 4]
# a and b unchanged
```

#### `+=` operator

Extends in place — equivalent to `.extend()`:

python

```python
a = [1, 2]
a += [3, 4]   # a is now [1, 2, 3, 4]
```

### Removing Elements

#### `.remove(x)`

Removes the **first occurrence** of value `x`. Raises `ValueError` if not found. O(n).

python

```python
a = [1, 2, 3, 2, 4]
a.remove(2)   # [1, 3, 2, 4] — only first 2 removed
a.remove(99)  # ValueError
```

#### `.pop(i)`

Removes and **returns** the element at index `i`. If no index given, removes and returns the last element. O(1) for last element, O(n) for arbitrary index.

python

```python
a = [1, 2, 3, 4]
a.pop()     # returns 4, a is now [1, 2, 3]
a.pop(0)    # returns 1, a is now [2, 3]
a.pop(1)    # returns 3, a is now [2]
```

#### `del` statement

`del` is a Python statement (not a method) that removes an element or slice by index:

python

```python
a = [1, 2, 3, 4, 5]
del a[1]      # [1, 3, 4, 5]
del a[1:3]    # removes a slice — [1, 5]
del a         # deletes the variable itself
```

#### `.clear()`

Removes all elements. List becomes empty.

python

```python
a = [1, 2, 3]
a.clear()   # []
```

---

### Searching

#### `in` operator

python

```python
3 in [1, 2, 3, 4]    # True
9 in [1, 2, 3, 4]    # False
```

Linear search, O(n).

#### `.index(x)`

Returns the index of the **first occurrence** of `x`. Raises `ValueError` if not found.

python

```python
a = [10, 20, 30, 20]
a.index(20)      # 1
a.index(20, 2)   # 3 — second arg is start index for search
a.index(99)      # ValueError
```

#### `.count(x)`

Returns how many times `x` appears. O(n).

python

```python
[1, 2, 2, 3, 2].count(2)   # 3
```

---

### Sorting

#### `.sort()`

Sorts the list **in place**. Returns `None`. Uses Timsort — O(n log n).

python

```python
a = [3, 1, 4, 1, 5, 9]
a.sort()              # [1, 1, 3, 4, 5, 9]
a.sort(reverse=True)  # [9, 5, 4, 3, 1, 1]
```

#### `sorted()`

Returns a **new sorted list**, original unchanged:

python

```python
a = [3, 1, 4]
b = sorted(a)           # b = [1, 3, 4], a unchanged
b = sorted(a, reverse=True)
```

#### `key` parameter

Both `.sort()` and `sorted()` accept a `key` argument — a function applied to each element before comparison. The element itself is not changed, only the comparison is based on the key:

python

```python
words = ["banana", "apple", "cherry", "fig"]
words.sort(key=len)             # sort by string length
# ['fig', 'apple', 'banana', 'cherry']

words.sort(key=lambda x: x[-1]) # sort by last character
```

For sorting complex objects:

python

```python
people = [("Alice", 30), ("Bob", 25), ("Charlie", 35)]
people.sort(key=lambda p: p[1])  # sort by age
# [('Bob', 25), ('Alice', 30), ('Charlie', 35)]
```

#### `.reverse()`

Reverses the list **in place**:

python

```python
a = [1, 2, 3]
a.reverse()   # [3, 2, 1]
```

---

### Copying

This is a critical area. In Python, assignment does **not** copy:

python

```python
a = [1, 2, 3]
b = a           # b and a point to the SAME list
b.append(4)
print(a)        # [1, 2, 3, 4] — a was also changed
```

```
a ──→ [ 1 | 2 | 3 ]
b ──↗
```

#### Shallow copy

A shallow copy creates a new list, but **the elements themselves are not copied** — they are the same objects:

python

```python
a = [1, 2, 3]
b = a.copy()      # new list, same elements
b = a[:]          # slice copy — same effect
b = list(a)       # constructor copy — same effect

b.append(4)
print(a)   # [1, 2, 3] — a is unaffected now
```

For a list of immutable objects (ints, strings), shallow copy is sufficient.

For a list of **mutable objects** (like nested lists), shallow copy is not enough:

python

```python
a = [[1, 2], [3, 4]]
b = a.copy()

b[0].append(99)
print(a)   # [[1, 2, 99], [3, 4]] — a[0] was also modified
```

```
a ──→ [ ptr0 | ptr1 ]
b ──→ [ ptr0 | ptr1 ]   ← different list, but ptr0 and ptr1 are the same objects
           ↓
        [1, 2]   ← shared, mutating via b also affects a
```

#### Deep copy

Recursively copies all nested objects:

python

```python
import copy

a = [[1, 2], [3, 4]]
b = copy.deepcopy(a)

b[0].append(99)
print(a)   # [[1, 2], [3, 4]] — unaffected
```

---

### 2D Lists

#### The wrong way:

python

```python
grid = [[0] * 3] * 3
grid[0][0] = 99
print(grid)  # [[99, 0, 0], [99, 0, 0], [99, 0, 0]]
```

`* 3` repeats the **same inner list reference** three times — same problem as shallow copy.

#### The correct way — list comprehension:

python

```python
grid = [[0] * 3 for _ in range(3)]
grid[0][0] = 99
print(grid)  # [[99, 0, 0], [0, 0, 0], [0, 0, 0]]
```

Each iteration of the comprehension creates a **new** inner list.

Accessing elements:

python

```python
grid[row][col]
```

Python has no `grid[row, col]` syntax for plain lists (that's NumPy-specific).

---

### List Comprehensions

A **list comprehension** is a concise way to build a list from an iterable with an optional filter. It replaces the pattern of creating an empty list and appending in a loop.

```
[expression  for  variable  in  iterable  if  condition]
     ↑              ↑              ↑              ↑
  what to        loop var       source        optional
  produce                                      filter
```

python

```python
squares = [x * x for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

evens = [x for x in range(20) if x % 2 == 0]
# [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

words = ["hello", "world", "python"]
upper = [w.upper() for w in words]
# ['HELLO', 'WORLD', 'PYTHON']
```

Nested comprehension for 2D:

python

```python
flat = [grid[i][j] for i in range(3) for j in range(3)]
```

---

### Other Useful Operations

#### `min()`, `max()`, `sum()`

python

```python
a = [3, 1, 4, 1, 5]
min(a)    # 1
max(a)    # 5
sum(a)    # 14
```

#### `any()`, `all()`

python

```python
a = [0, 1, 2, 3]
any(a)   # True — at least one element is truthy
all(a)   # False — not all elements are truthy (0 is falsy)

any(x > 2 for x in a)   # True
all(x > 0 for x in a)   # False
```

#### `zip()` with lists

python

```python
a = [1, 2, 3]
b = [4, 5, 6]
list(zip(a, b))   # [(1, 4), (2, 5), (3, 6)]
```

#### `enumerate()`

python

```python
for i, val in enumerate([10, 20, 30]):
    print(i, val)
```

#### `reversed()`

Returns an iterator over the list in reverse — does not modify the original:

python

```python
for x in reversed([1, 2, 3]):
    print(x)   # 3 2 1
```

#### Flattening a nested list

python

```python
nested = [[1, 2], [3, 4], [5, 6]]
flat = [x for sublist in nested for x in sublist]
# [1, 2, 3, 4, 5, 6]
```

---

### Stack and Queue behavior

A Python list can function as a **stack** directly:

python

```python
stack = []
stack.append(1)   # push
stack.append(2)
stack.pop()       # pop — O(1)
```

As a **queue**, using a list is inefficient — `pop(0)` is O(n) because all elements shift. Use `collections.deque` instead:

python

```python
from collections import deque

queue = deque()
queue.append(1)      # enqueue — O(1)
queue.append(2)
queue.popleft()      # dequeue — O(1)
```

