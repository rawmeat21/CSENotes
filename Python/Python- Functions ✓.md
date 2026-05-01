### Returning Multiple Values

```python
def min_max(lst):
    return min(lst), max(lst)

lo, hi = min_max([3, 1, 4, 1, 5])
```

What looks like returning multiple values is actually returning a **tuple** — a fixed-size immutable sequence. The left side `lo, hi` is **tuple unpacking**, same as you saw with `enumerate()`. Python assigns each element of the tuple to each variable positionally.

Under the hood:

```
return min(lst), max(lst)   →   returns (min(lst), max(lst))  ← a tuple
lo, hi = (1, 5)             →   lo = 1, hi = 5
```

You can also ignore values you don't need using `_` by convention:

```python
_, hi = min_max([3, 1, 4, 1, 5])   # only care about max
```

### Positional vs Keyword Arguments

When calling a function, you can pass arguments **by position** or **by name**:

```python
def connect(host, port, timeout):
    ...

connect("localhost", 8080, 30)           # positional
connect(host="localhost", port=8080, timeout=30)   # keyword
connect("localhost", port=8080, timeout=30)        # mixed — positional first
```

With keyword arguments, **order doesn't matter**. With positional, order is everything — same as C++.

Mixed calls: positional arguments must always come before keyword arguments.

---

### `*args` — Variable Positional Arguments

When you want a function to accept any number of positional arguments:

```python
def add(*args):
    return sum(args)

add(1, 2)           # 3
add(1, 2, 3, 4, 5)  # 15
```

`*args` collects all extra positional arguments into a **tuple**. The name `args` is convention — the `*` is what matters syntactically.

```python
def f(*args):
    print(type(args))   # <class 'tuple'>
    print(args)         # (1, 2, 3)

f(1, 2, 3)
```

You can have normal parameters before `*args`:

```python
def f(a, b, *args):
    print(a, b, args)

f(1, 2, 3, 4, 5)   # a=1, b=2, args=(3, 4, 5)
```

Nothing can come after `*args` as a positional parameter — any parameter after it becomes keyword-only.

---

### `**kwargs` — Variable Keyword Arguments

When you want a function to accept any number of keyword arguments:

```python
def display(**kwargs):
    for key, value in kwargs.items():
        print(f"{key} = {value}")

display(name="Romit", age=20, city="Delhi")
# name = Romit
# age = 20
# city = Delhi
```

`**kwargs` collects all extra keyword arguments into a **dict**. Again, `kwargs` is convention — `**` is the syntax.
```python
def f(**kwargs):
    print(type(kwargs))   # <class 'dict'>
```

#### Combining all forms
```python
def f(a, b, *args, **kwargs):
    print(a, b, args, kwargs)

f(1, 2, 3, 4, x=10, y=20)
# a=1, b=2, args=(3, 4), kwargs={'x': 10, 'y': 20}
```

The ordering rule for parameters is strict:

```
def f(positional, *args, keyword_only, **kwargs)
         ↑           ↑          ↑           ↑
     normal      var-pos    after *args   var-kw
```

---

### Keyword-Only Parameters

Any parameter **after `*args`** (or after a bare `*`) can only be passed by keyword:

python

```python
def f(a, b, *, verbose=False):
    ...

f(1, 2)                # ok, verbose=False
f(1, 2, verbose=True)  # ok
f(1, 2, True)          # TypeError — verbose must be passed by keyword
```

The bare `*` here means "no var-args, but everything after this is keyword-only". This lets you enforce named arguments without allowing arbitrary `*args`.

---

### Unpacking into function calls

The `*` and `**` syntax also works at the **call site** to unpack sequences/dicts into arguments:

python

```python
def add(a, b, c):
    return a + b + c

nums = [1, 2, 3]
add(*nums)          # same as add(1, 2, 3)

params = {'a': 1, 'b': 2, 'c': 3}
add(**params)       # same as add(a=1, b=2, c=3)
```

`*` unpacks a list/tuple into positional arguments. `**` unpacks a dict into keyword arguments.

### Type Hints

Python is dynamically typed — you don't declare types. But you can add **optional type hints** for readability and tooling:
```python
def add(a: int, b: int) -> int:
    return a + b
```

These are **not enforced at runtime**. Python ignores them during execution.

```python
def greet(name: str, times: int = 1) -> None:
    for _ in range(times):
        print(f"Hello, {name}")
```

`-> None` signals the function returns nothing meaningful.

