## Sets and Frozensets in Python

---

### What is a Set?

A `set` is an **unordered collection of unique hashable objects**. Directly maps to `std::unordered_set<T>` in C++. Backed by a hash table — O(1) average for insert, delete, and membership test.

Two defining properties:

- **No duplicates** — inserting an existing element does nothing
- **No ordering** — elements have no index, no guaranteed iteration order

---

### Creating Sets

python

```python
a = {1, 2, 3}                  # literal
b = {1, 2, 2, 3, 3}            # {1, 2, 3} — duplicates removed
c = set()                       # empty set — NOT {} which is an empty dict
d = set([1, 2, 2, 3])          # from iterable — {1, 2, 3}
e = set("hello")               # {'h', 'e', 'l', 'o'} — unique chars
```

The `{}` syntax without key-value pairs creates an empty **dict**, not a set. For an empty set you must use `set()`.

Elements must be **hashable** — same rule as dict keys. So no lists, no dicts, no sets inside a set.

---

### Adding and Removing

#### `.add(x)`

Adds a single element. If already present, does nothing. O(1).

python

```python
s = {1, 2, 3}
s.add(4)    # {1, 2, 3, 4}
s.add(2)    # {1, 2, 3, 4} — no change, no error
```

#### `.remove(x)`

Removes `x`. Raises `KeyError` if not present.

python

```python
s.remove(2)    # removes 2
s.remove(99)   # KeyError
```

#### `.discard(x)`

Removes `x`. Does **nothing** if not present — no exception.

python

```python
s.discard(2)    # removes 2
s.discard(99)   # no error, no change
```

#### `.pop()`

Removes and returns an **arbitrary** element. Since sets are unordered, you have no control over which element is removed. Raises `KeyError` on empty set.

python

```python
s = {1, 2, 3}
s.pop()   # returns some element, unpredictable which
```

#### `.clear()`

Removes all elements.

python

```python
s.clear()   # set()
```

---

### Membership Test

python

```python
s = {1, 2, 3}
2 in s     # True  — O(1)
9 in s     # False
9 not in s # True
```

This is the primary reason to use a set over a list. `x in list` is O(n). `x in set` is O(1).

---

### Set Operations

This is where sets are most powerful. Python implements all standard mathematical set operations both as **methods** and as **operators**.

#### Union — all elements from both sets

python

```python
a = {1, 2, 3}
b = {3, 4, 5}

a | b             # {1, 2, 3, 4, 5}
a.union(b)        # same
```

```
a = { 1  2  3 }
b =       { 3  4  5 }
    ───────────────────
a|b = { 1  2  3  4  5 }
```

#### Intersection — elements present in both

python

```python
a & b                    # {3}
a.intersection(b)        # same
```

```
a = { 1  2  3 }
b =       { 3  4  5 }
    ───────────────────
a&b =        { 3 }
```

#### Difference — elements in a but not in b

python

```python
a - b                 # {1, 2}
a.difference(b)       # same

b - a                 # {4, 5} — order matters
```

```
a = { 1  2  3 }
b =       { 3  4  5 }
    ───────────────────
a-b = { 1  2 }
b-a =          { 4  5 }
```

#### Symmetric Difference — elements in either but not both

python

```python
a ^ b                          # {1, 2, 4, 5}
a.symmetric_difference(b)      # same
```

```
a = { 1  2  3 }
b =       { 3  4  5 }
    ───────────────────
a^b = { 1  2     4  5 }   (3 excluded — it's in both)
```

---

### In-place Set Operations

These modify the set in place instead of returning a new one:

python

```python
a |= b    # a.update(b)                    — union in place
a &= b    # a.intersection_update(b)       — intersection in place
a -= b    # a.difference_update(b)         — difference in place
a ^= b    # a.symmetric_difference_update(b)
```

---

### Subset and Superset

python

```python
a = {1, 2}
b = {1, 2, 3, 4}

a.issubset(b)      # True  — every element of a is in b
a <= b             # True  — same

a < b              # True  — proper subset: a <= b AND a != b

b.issuperset(a)    # True  — b contains all elements of a
b >= a             # True  — same

b > a              # True  — proper superset
```

```
b = { 1  2  3  4 }
a =   { 1  2 }        ← a is a subset of b
```

#### `.isdisjoint()`

Returns `True` if two sets have **no elements in common**:

python

```python
{1, 2}.isdisjoint({3, 4})   # True
{1, 2}.isdisjoint({2, 3})   # False
```

---

### Set Methods that Accept Any Iterable

The operator versions (`|`, `&`, `-`, `^`) require both sides to be sets. The method versions accept any iterable:

python

```python
{1, 2, 3}.union([3, 4, 5])          # works — list passed
{1, 2, 3} | [3, 4, 5]               # TypeError — | needs a set on right side
```

---

### `len()` and Iteration

python

```python
s = {1, 2, 3}
len(s)           # 3

for x in s:
    print(x)     # order is not guaranteed
```

Since sets are unordered, you cannot index them:

python

```python
s[0]    # TypeError — sets do not support indexing
```

---

### Set Comprehensions

Same syntax as dict/list comprehensions:

python

```python
squares = {x*x for x in range(10)}
# {0, 1, 4, 9, 16, 25, 36, 49, 64, 81}

evens = {x for x in range(20) if x % 2 == 0}
```

Duplicates are eliminated automatically:

python

```python
{x % 3 for x in range(10)}   # {0, 1, 2}
```

---

### Common Use Cases

#### Deduplication

python

```python
lst = [1, 2, 2, 3, 3, 3, 4]
unique = list(set(lst))   # [1, 2, 3, 4] — order not guaranteed
```

#### Fast membership testing

python

```python
valid_commands = {'start', 'stop', 'pause', 'resume'}

if command in valid_commands:   # O(1) vs O(n) for a list
    ...
```

#### Finding common or distinct elements

python

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

common = a & b          # {3, 4}
only_in_a = a - b       # {1, 2}
all_unique = a | b      # {1, 2, 3, 4, 5, 6}
```

---

### Frozenset

A `frozenset` is an **immutable set**. Same structure as a set — hash table, unique elements, unordered — but it cannot be modified after creation.

python

```python
f = frozenset([1, 2, 3])
f = frozenset({1, 2, 3})   # from a set
f = frozenset()             # empty frozenset
```

#### What you lose vs set

python

```python
f.add(4)      # AttributeError — no add
f.remove(1)   # AttributeError — no remove
f.discard(1)  # AttributeError — no discard
f.pop()       # AttributeError — no pop
f.clear()     # AttributeError — no clear
```

No mutation methods at all.

#### What you keep

All read operations and set operations still work:

python

```python
1 in f               # True
len(f)               # 3
f | frozenset([4])   # frozenset({1, 2, 3, 4}) — returns new frozenset
f & frozenset([2])   # frozenset({2})
f - frozenset([1])   # frozenset({2, 3})
```

Set operations between a `frozenset` and a `set` are allowed — the result type follows the left operand:

python

```python
frozenset([1,2]) | {3, 4}    # frozenset({1, 2, 3, 4})
{1, 2} | frozenset([3, 4])   # {1, 2, 3, 4}  — regular set
```

#### Why frozenset exists — hashability

A regular `set` is not hashable — you cannot use it as a dict key or put it inside another set. A `frozenset` **is hashable**:

python

```python
s = {1, 2, 3}
hash(s)              # TypeError — set is not hashable

f = frozenset([1, 2, 3])
hash(f)              # works

d = {frozenset([1,2]): "group A"}   # valid dict key
nested = {frozenset([1,2]), frozenset([3,4])}  # set of frozensets
```

This is the same relationship as list → tuple. You use frozenset when you need set semantics but also need the object to be hashable or immutable.

---

### Internal Structure — Why O(1)

Both `set` and `frozenset` are backed by a **hash table**. When you insert or look up an element:

1. Python calls `hash(element)` to get an integer
2. That integer is mapped to a slot in the table
3. The element is stored or found at that slot

```
element → hash() → slot index → bucket
```

Collisions are handled internally. This is why only hashable elements are allowed — you need a stable hash to find the slot again later.