## Dictionaries in Python

---

### What is a Dict?

A Python `dict` is a **hash map** — same underlying concept as `std::unordered_map<K, V>` in C++. It stores **key-value pairs**, with O(1) average case for insert, delete, and lookup.

Key differences from C++ unordered_map:

- Keys can be of **any hashable type**
- Values can be any type
- Mixed key types in the same dict are valid
- Dicts maintain **insertion order** — guaranteed by the language spec

---

### Creating Dicts

python

```python
a = {'name': 'Romit', 'age': 20}     # literal
b = {}                                 # empty dict
c = dict()                             # empty dict via constructor
d = dict(name='Romit', age=20)        # keyword args to constructor
e = dict([('name', 'Romit'), ('age', 20)])  # from list of (key, value) pairs
```

---

### Accessing Values

python

```python
d = {'name': 'Romit', 'age': 20}

d['name']        # 'Romit'
d['age']         # 20
d['missing']     # KeyError — key doesn't exist
```

#### `.get(key, default)`

Safe access — returns `default` instead of raising `KeyError` if key is absent:

python

```python
d.get('name')           # 'Romit'
d.get('missing')        # None  — default default is None
d.get('missing', 0)     # 0     — your specified default
```

Use `[]` when you are certain the key exists and want an exception on failure. Use `.get()` when absence is possible.

---

### Inserting and Updating

python

```python
d = {'name': 'Romit'}

d['age'] = 20          # insert new key
d['name'] = 'Someone'  # update existing key — overwrites
```

Both insert and update use the same `[]` syntax. If the key exists, it overwrites. If not, it creates.

#### `.update()`

Merges another dict (or key-value pairs) into the current dict. Existing keys are overwritten:

python

```python
d = {'a': 1, 'b': 2}
d.update({'b': 99, 'c': 3})
# {'a': 1, 'b': 99, 'c': 3}

d.update(x=10, y=20)   # keyword args also work
```

---

### Removing Elements

#### `del`

python

```python
d = {'a': 1, 'b': 2, 'c': 3}
del d['b']     # {'a': 1, 'c': 3}
del d['x']     # KeyError
```

#### `.pop(key, default)`

Removes key and **returns its value**. If key absent and default provided, returns default. If absent and no default, raises `KeyError`:

python

```python
d = {'a': 1, 'b': 2}
d.pop('a')         # returns 1, d is now {'b': 2}
d.pop('x', 0)      # returns 0, d unchanged
d.pop('x')         # KeyError
```

#### `.popitem()`

Removes and returns the **last inserted** key-value pair as a tuple. Useful for iterative processing. Raises `KeyError` on empty dict:


```python
d = {'a': 1, 'b': 2, 'c': 3}
d.popitem()   # ('c', 3), d is now {'a': 1, 'b': 2}
```

#### `.clear()`

Removes all entries:

python

```python
d.clear()   # {}
```

---

### Membership Test

`in` checks for **key** existence, not value:

python

```python
d = {'a': 1, 'b': 2}

'a' in d      # True
'x' in d      # False
'x' not in d  # True
1 in d        # False — 1 is a value, not a key
```

O(1) average case — hashing, not linear scan.

---

### Iterating

#### Over keys (default)

python

```python
d = {'a': 1, 'b': 2, 'c': 3}

for key in d:
    print(key)
# a b c
```

Iterating a dict directly gives you keys.

#### `.keys()`, `.values()`, `.items()`

These return **view objects** — not lists, not copies. They are live views into the dict. If the dict changes, the view reflects that change.

python

```python
d.keys()    # dict_keys(['a', 'b', 'c'])
d.values()  # dict_values([1, 2, 3])
d.items()   # dict_items([('a', 1), ('b', 2), ('c', 3)])
```

Iterating:

python

```python
for key in d.keys():
    print(key)

for val in d.values():
    print(val)

for key, val in d.items():   # tuple unpacking
    print(key, val)
```

Converting to a list if you need one:

python

```python
list(d.keys())     # ['a', 'b', 'c']
list(d.values())   # [1, 2, 3]
list(d.items())    # [('a', 1), ('b', 2), ('c', 3)]
```

---

### `len()`

python

```python
len({'a': 1, 'b': 2})   # 2 — number of key-value pairs
```

---

### What is Hashable?

A dict key must be **hashable** — meaning it has a `__hash__()` method that returns an integer, and it implements equality comparison. The hash must not change over the object's lifetime.

**Hashable types (valid keys):**

- `int`, `float`, `str`, `bool`
- `tuple` — only if all its elements are also hashable
- `None`
- Custom objects (by default, hashed by identity)

**Not hashable (invalid keys):**

- `list` — mutable, hash would change
- `dict` — mutable
- `set` — mutable

python

```python
d = {}
d[(1, 2)] = "tuple key"     # valid
d[[1, 2]] = "list key"      # TypeError: unhashable type: 'list'
```

The rule: **mutable containers are not hashable**.

---

### Dict Comprehensions

Same idea as list comprehensions, but produces a dict:

python

```python
{key_expr: val_expr for var in iterable if condition}
```

python

```python
squares = {x: x*x for x in range(6)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

filtered = {k: v for k, v in squares.items() if v > 5}
# {3: 9, 4: 16, 5: 25}

# invert a dict (swap keys and values)
d = {'a': 1, 'b': 2, 'c': 3}
inverted = {v: k for k, v in d.items()}
# {1: 'a', 2: 'b', 3: 'c'}
```

---

### Nested Dicts

python

```python
users = {
    'alice': {'age': 30, 'role': 'admin'},
    'bob':   {'age': 25, 'role': 'user'},
}

users['alice']['role']     # 'admin'
users['bob']['age']        # 25
```

Safe nested access — chaining `.get()`:

python

```python
users.get('alice', {}).get('role', 'unknown')
```

If `'alice'` doesn't exist, `.get()` returns `{}`, and calling `.get('role')` on that returns `'unknown'` — no `KeyError`.

---

### `defaultdict`

From the `collections` module. A subclass of `dict` that automatically creates a default value for missing keys instead of raising `KeyError`:



```python
from collections import defaultdict

d = defaultdict(int)    # default factory is int, int() returns 0
d['a'] += 1             # no KeyError — 'a' didn't exist, was set to 0 first
d['a'] += 1
d['a']                  # 2

d = defaultdict(list)   # default factory is list, list() returns []
d['fruits'].append('apple')
d['fruits'].append('banana')
d['vegs'].append('carrot')
# {'fruits': ['apple', 'banana'], 'vegs': ['carrot']}
```

The argument to `defaultdict` is a **callable** that takes no arguments and returns the default value. `int`, `list`, `set` are all callables that return `0`, `[]`, `set()` respectively.

This replaces the `.setdefault()` pattern cleanly.

---

### `Counter`

From `collections`. A dict subclass specifically for **counting hashable objects**:

python

```python
from collections import Counter

c = Counter("abracadabra")
# Counter({'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})

c = Counter([1, 1, 2, 3, 1, 2])
# Counter({1: 3, 2: 2, 3: 1})

c['a']        # 5
c['z']        # 0 — missing keys return 0, not KeyError

c.most_common(2)   # [('a', 5), ('b', 2)] — top 2 most frequent
```

Counter supports arithmetic:

python

```python
a = Counter("aab")
b = Counter("abb")

a + b   # Counter({'a': 3, 'b': 3}) — add counts
a - b   # Counter({'a': 1})         — subtract, drop zero/negative
a & b   # Counter({'a': 1, 'b': 1}) — min of each
a | b   # Counter({'a': 2, 'b': 2}) — max of each
```

---

### `OrderedDict`

From `collections`. Before Python 3.7, regular dicts did not guarantee insertion order. `OrderedDict` was the solution. Today, regular dicts preserve insertion order, so `OrderedDict` is mostly legacy.

One thing it still does that regular dict doesn't: `.move_to_end(key)`:

python

```python
from collections import OrderedDict

d = OrderedDict([('a', 1), ('b', 2), ('c', 3)])
d.move_to_end('a')         # moves 'a' to end
d.move_to_end('c', last=False)  # moves 'c' to front
```

---

### Copying

Same shallow/deep copy distinction as lists:

python

```python
import copy

d = {'a': [1, 2], 'b': [3, 4]}

e = d.copy()           # shallow — inner lists are shared
e = copy.deepcopy(d)   # deep — inner lists are independent
```

---

### Sorting a Dict

Dicts themselves are not sorted by key (they preserve insertion order, not sorted order). To iterate in sorted order:

python

```python
d = {'banana': 3, 'apple': 5, 'cherry': 1}

# by key
for k in sorted(d):
    print(k, d[k])

# by value
for k in sorted(d, key=lambda k: d[k]):
    print(k, d[k])

# by value, descending
for k in sorted(d, key=lambda k: d[k], reverse=True):
    print(k, d[k])

# sorted items as list of tuples
sorted(d.items(), key=lambda item: item[1])
# [('cherry', 1), ('banana', 3), ('apple', 5)]
```

## Tuples in Python

---

### What is a Tuple?

A tuple is an **immutable sequence**. Structurally it is like a list — ordered, indexable, iterable — but once created, it **cannot be modified**. No appending, no item assignment, no removal.

In C++ there is `std::tuple<T1, T2, ...>` which is fixed-size and typed. Python's tuple is dynamically typed like everything else, but similarly fixed once created.

python

```python
a = (1, 2, 3)
b = (1, "hello", 3.14, None)   # mixed types, valid
```

---

### Creating Tuples

python

```python
a = (1, 2, 3)          # standard
b = 1, 2, 3            # parentheses are optional — the comma makes it a tuple
c = (42,)              # single-element tuple — trailing comma is REQUIRED
d = 42,                # also a single-element tuple
e = ()                 # empty tuple
f = tuple()            # empty tuple via constructor
g = tuple([1, 2, 3])   # from iterable
h = tuple("hello")     # ('h', 'e', 'l', 'l', 'o')
```

The critical point about single-element tuples:

python

```python
x = (42)    # this is NOT a tuple — just 42 in parentheses, type is int
x = (42,)   # this IS a tuple — the comma is what defines it
```

The **comma** is the tuple constructor, not the parentheses. Parentheses are just for grouping and readability.

---

### Indexing and Slicing

Identical to lists:

python

```python
t = (10, 20, 30, 40, 50)

t[0]      # 10
t[-1]     # 50
t[1:3]    # (20, 30)  — slicing a tuple returns a tuple
t[::-1]   # (50, 40, 30, 20, 10)
```

What you **cannot** do:

python

```python
t[0] = 99    # TypeError: 'tuple' object does not support item assignment
```

---

### Immutability — What it Actually Means

The tuple itself cannot be modified — but if it contains mutable objects, those objects can still be mutated internally:

python

```python
t = ([1, 2], [3, 4])

t[0] = [9, 9]      # TypeError — cannot reassign the slot
t[0].append(99)    # valid — mutating the list object itself
print(t)           # ([1, 2, 99], [3, 4])
```

```
t ──→ [ ptr0 | ptr1 ]   ← this structure is frozen
          ↓        ↓
       [1,2]    [3,4]    ← these list objects can still change
```

The tuple's slots are frozen — what they point to cannot change. But the objects they point to can change internally if they are mutable.

This is why a tuple containing a list is **not hashable** — the contents can change even though the tuple structure can't:

python

```python
hash((1, 2, 3))       # works
hash((1, [2, 3]))     # TypeError — contains unhashable list
```

---

### Tuple Packing and Unpacking

**Packing** — combining values into a tuple:

python

```python
t = 1, 2, 3      # packed into (1, 2, 3)
```

**Unpacking** — assigning tuple elements to variables:

python

```python
t = (1, 2, 3)
a, b, c = t      # a=1, b=2, c=3
```

The number of variables must match the number of elements, otherwise `ValueError`.

#### Extended unpacking with `*`

The `*` can absorb multiple elements into a list:

python

```python
a, *b, c = (1, 2, 3, 4, 5)
# a=1, b=[2, 3, 4], c=5

first, *rest = (1, 2, 3, 4)
# first=1, rest=[2, 3, 4]

*init, last = (1, 2, 3, 4)
# init=[1, 2, 3], last=4
```

`*b` here captures everything not claimed by the named variables. It always produces a **list**, even if the source is a tuple.

Only one `*` variable is allowed per unpacking expression.

#### Swapping variables

In C++ you need a temp variable or `std::swap`. In Python:

python

```python
a, b = b, a
```

What happens: the right side `b, a` creates a tuple first, then it gets unpacked into `a, b`. No temp variable needed.

---

### Tuple Methods

Tuples have only two methods — because mutation methods don't exist:

#### `.count(x)`

python

```python
(1, 2, 2, 3, 2).count(2)   # 3
```

#### `.index(x)`

python

```python
(10, 20, 30).index(20)     # 1
(10, 20, 30).index(99)     # ValueError
```

Everything else — `len()`, `in`, `+`, `*`, slicing, `min()`, `max()`, `sum()`, `sorted()`, `enumerate()`, `zip()` — all work the same as with lists.

---

### Tuple vs List — When to Use Which

This is a design decision, not just a performance one. The distinction:

- **List** — a collection of items that conceptually belong to the same category. The length can vary. Homogeneous by intent.
- **Tuple** — a fixed structure where each position has a specific meaning. Heterogeneous by intent.

python

```python
# list — a bunch of scores, all same kind
scores = [95, 87, 92, 78]

# tuple — a record where position 0 is name, position 1 is age
person = ("Romit", 20)
```

This is the same conceptual distinction as a C `struct` vs an array. A tuple is closer to a lightweight struct where fields are accessed by position.

#### Performance

Tuples are slightly faster than lists for iteration and access. They also use less memory because they don't need to maintain capacity for growth.

python

```python
import sys
sys.getsizeof([1, 2, 3])    # 88 bytes (on typical 64-bit system)
sys.getsizeof((1, 2, 3))    # 64 bytes
```

Tuples are also **hashable** (when all elements are hashable), so they can be used as dict keys or set elements — lists cannot.

---

### Tuples as Dict Keys

Since tuples are hashable (if contents are), they work as dict keys:

python

```python
graph = {}
graph[(0, 0)] = "origin"
graph[(1, 2)] = "point A"

graph[(1, 2)]   # "point A"
```

---

### Named Tuples

A **named tuple** is a tuple subclass where each position has a name. From the `collections` module:

python

```python
from collections import namedtuple

Point = namedtuple('Point', ['x', 'y'])

p = Point(3, 4)
p.x        # 3  — access by name
p.y        # 4
p[0]       # 3  — access by index still works
```

It behaves like a regular tuple (immutable, hashable, unpackable) but fields are accessible by name:

python

```python
x, y = p       # unpacking still works
p._asdict()    # {'x': 3, 'y': 4} — convert to dict
p._replace(x=10)  # returns new Point(10, 4) — tuple is immutable so _replace makes a copy
```

Named tuples are useful when you want a lightweight record type without defining a full class.

In modern Python (3.7+) there is also `dataclasses` for more complex record types, but named tuples are lighter and still common.

---

### `zip()` returns tuples

Worth noting — `zip()` produces tuples, not lists:

python

```python
list(zip([1, 2, 3], ['a', 'b', 'c']))
# [(1, 'a'), (2, 'b'), (3, 'c')]
```

Each paired element is a tuple.