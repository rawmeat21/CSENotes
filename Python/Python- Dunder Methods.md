## Magic / Dunder Methods in Python

---

### What They Are

Dunder methods (double underscore on both sides — `__name__`) are methods Python calls **automatically** in response to specific operations or built-in function calls. You do not call them directly — you define them, and Python invokes them when the corresponding syntax or function is used.

In C++ this is operator overloading, but Python's system is broader — it covers not just operators but object construction, string representation, container behavior, context management, attribute access, and more.

The full set of dunder methods defines an object's **protocol** — what operations it supports and how it behaves when used with Python's built-in machinery.

---

### Object Lifecycle

#### `__new__(cls, ...)`

Called first. Creates and returns the new instance. Receives the class as first argument. Rarely overridden.

#### `__init__(self, ...)`

Called after `__new__`. Initializes the instance. The one you always write.

#### `__del__(self)`

Called when the object is about to be garbage collected. Unreliable timing — the GC decides when, not you.

python

```python
class MyClass:
    def __new__(cls, *args, **kwargs):
        print("creating")
        return super().__new__(cls)

    def __init__(self, x):
        print("initializing")
        self.x = x

    def __del__(self):
        print("destroying")
```

---

### String Representation

#### `__repr__(self)` → `str`

Unambiguous representation. Should ideally be valid Python that recreates the object. Used in the REPL, in containers, and as fallback when `__str__` is absent.

#### `__str__(self)` → `str`

Human-readable string. Used by `print()` and `str()`.

#### `__format__(self, spec)` → `str`

Called by `format()` and f-strings when a format spec is provided:

python

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Vector({self.x!r}, {self.y!r})"

    def __str__(self):
        return f"<{self.x}, {self.y}>"

    def __format__(self, spec):
        if spec == 'polar':
            r = (self.x**2 + self.y**2) ** 0.5
            return f"|{r:.2f}|"
        return str(self)
```

python

```python
v = Vector(3, 4)

repr(v)          # "Vector(3, 4)"
str(v)           # "<3, 4>"
print(v)         # "<3, 4>"  — uses __str__
f"{v}"           # "<3, 4>"
f"{v:polar}"     # "|5.00|"  — uses __format__ with spec='polar'
```

The `!r` inside f-strings forces `repr()` on the value. `!s` forces `str()`.

---

### Comparison Operators

python

```python
__eq__(self, other)    # ==
__ne__(self, other)    # !=
__lt__(self, other)    # 
__le__(self, other)    # <=
__gt__(self, other)    # >
__ge__(self, other)    # >=
```

python

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    def __eq__(self, other):
        return self.celsius == other.celsius

    def __lt__(self, other):
        return self.celsius < other.celsius

    def __le__(self, other):
        return self.celsius <= other.celsius
```

#### `__eq__` and `__hash__` coupling

If you define `__eq__`, Python automatically sets `__hash__` to `None` — making the object unhashable. This is intentional: if two objects are equal, they must have the same hash. Since Python can't know that your `__eq__` is consistent with hash, it removes hashability to be safe.

If you want your object to be both equality-comparable and hashable (usable as a dict key or in a set), you must explicitly define `__hash__`:

python

```python
def __hash__(self):
    return hash(self.celsius)
```

#### `@functools.total_ordering`

If you define `__eq__` and one of `__lt__`, `__le__`, `__gt__`, `__ge__`, this decorator fills in the rest automatically:

python

```python
from functools import total_ordering

@total_ordering
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    def __eq__(self, other):
        return self.celsius == other.celsius

    def __lt__(self, other):
        return self.celsius < other.celsius

# __le__, __gt__, __ge__ are now automatically derived
```

---

### Arithmetic Operators

python

```python
__add__(self, other)         # self + other
__sub__(self, other)         # self - other
__mul__(self, other)         # self * other
__truediv__(self, other)     # self / other
__floordiv__(self, other)    # self // other
__mod__(self, other)         # self % other
__pow__(self, other)         # self ** other
__matmul__(self, other)      # self @ other  (matrix multiply, Python 3.5+)
```

#### Reflected operators

When Python evaluates `a + b`, it first tries `a.__add__(b)`. If that returns `NotImplemented`, Python then tries `b.__radd__(a)`. This handles cases where the left operand doesn't know how to handle the right operand's type:

python

```python
__radd__(self, other)    # other + self  (reflected)
__rsub__(self, other)    # other - self
__rmul__(self, other)    # other * self
# ... and so on for all arithmetic ops
```

python

```python
class Vector:
    def __mul__(self, scalar):
        if isinstance(scalar, (int, float)):
            return Vector(self.x * scalar, self.y * scalar)
        return NotImplemented

    def __rmul__(self, scalar):     # handles: 3 * vector
        return self.__mul__(scalar)
```

python

```python
v = Vector(1, 2)
v * 3    # calls v.__mul__(3)  → works
3 * v    # calls (3).__mul__(v) → int doesn't know Vector → NotImplemented
         # Python then tries v.__rmul__(3) → works
```

Returning `NotImplemented` (not raising it — returning it) signals to Python that this operation is not supported with these types, and Python should try the reflected version.

#### In-place operators

Called when `+=`, `-=`, etc. are used. Should mutate `self` and return `self`. If not defined, Python falls back to `__add__` + rebind:

python

```python
__iadd__(self, other)    # +=
__isub__(self, other)    # -=
__imul__(self, other)    # *=
# ... etc
```

python

```python
def __iadd__(self, other):
    self.x += other.x
    self.y += other.y
    return self             # must return self
```

#### Unary operators

python

```python
__neg__(self)      # -obj
__pos__(self)      # +obj
__abs__(self)      # abs(obj)
__invert__(self)   # ~obj  (bitwise NOT)
```

---

### Bitwise Operators

python

```python
__and__(self, other)      # &
__or__(self, other)       # |
__xor__(self, other)      # ^
__lshift__(self, other)   # 
__rshift__(self, other)   # >>
```

All have reflected (`__rand__`, etc.) and in-place (`__iand__`, etc.) variants.

---

### Type Conversion

python

```python
__bool__(self)     # bool(obj), truthiness in if/while
__int__(self)      # int(obj)
__float__(self)    # float(obj)
__complex__(self)  # complex(obj)
__bytes__(self)    # bytes(obj)
__str__(self)      # str(obj)
__repr__(self)     # repr(obj)
```

#### `__bool__` in depth

If `__bool__` is not defined, Python falls back to `__len__` — if length is 0, the object is falsy. If neither is defined, the object is always truthy.

python

```python
class Container:
    def __init__(self, items):
        self.items = items

    def __len__(self):
        return len(self.items)

c = Container([])
if c:              # False — __len__ returns 0
    ...

c = Container([1])
if c:              # True — __len__ returns 1
    ...
```

---

### Container / Collection Protocol

These make your object behave like a list, dict, or set.

python

```python
__len__(self)                    # len(obj)
__getitem__(self, key)           # obj[key]
__setitem__(self, key, value)    # obj[key] = value
__delitem__(self, key)           # del obj[key]
__contains__(self, item)         # item in obj
__iter__(self)                   # iter(obj), for loops
__next__(self)                   # next(obj)
__reversed__(self)               # reversed(obj)
```

A full custom sequence:

python

```python
class FixedList:
    def __init__(self, *args):
        self._data = list(args)

    def __len__(self):
        return len(self._data)

    def __getitem__(self, index):
        return self._data[index]

    def __setitem__(self, index, value):
        self._data[index] = value

    def __delitem__(self, index):
        del self._data[index]

    def __contains__(self, item):
        return item in self._data

    def __iter__(self):
        return iter(self._data)

    def __reversed__(self):
        return reversed(self._data)
```

Once you define `__getitem__` and `__len__`, Python can derive basic iteration from them even without `__iter__` — it will call `__getitem__` with indices 0, 1, 2, ... until `IndexError`. But explicitly defining `__iter__` is more correct and efficient.

---

### Callable Objects

`__call__` makes an instance callable like a function:

python

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor

    def __call__(self, x):
        return x * self.factor

double = Multiplier(2)
double(5)     # 10
double(10)    # 20
```

`double` here behaves exactly like a function. `callable(double)` returns `True`. This is how function objects, decorators, and many library constructs work internally.

---

### Attribute Access

Already covered in getters/setters, included here for completeness:

python

```python
__getattr__(self, name)             # called when attribute not found normally
__getattribute__(self, name)        # called on every attribute access
__setattr__(self, name, value)      # called on every attribute assignment
__delattr__(self, name)             # called on every attribute deletion
__dir__(self)                       # called by dir(obj) — returns list of attributes
```

python

```python
class Frozen:
    def __setattr__(self, name, value):
        raise AttributeError("This object is immutable")
```

---

### Descriptors

A descriptor is a class that defines `__get__`, `__set__`, or `__delete__`. When an instance of such a class is assigned as a class attribute, Python invokes these methods on attribute access instead of returning the descriptor object directly.

`@property` is built on descriptors. Here is a manual descriptor:

python

```python
class Positive:
    def __set_name__(self, owner, name):
        self.name = name                  # called when class is defined

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self                   # accessed on class, not instance
        return obj.__dict__[self.name]

    def __set__(self, obj, value):
        if value <= 0:
            raise ValueError(f"{self.name} must be positive")
        obj.__dict__[self.name] = value

class Circle:
    radius = Positive()      # descriptor instance assigned as class attribute
    area   = Positive()

c = Circle()
c.radius = 5     # calls Positive.__set__
c.radius         # calls Positive.__get__
c.radius = -1    # ValueError
```

`__set_name__` is called at class creation time and tells the descriptor what attribute name it was assigned to. This is how `@property` and many ORM field systems (like Django models) work.

---

### Context Manager Protocol

Makes an object usable with the `with` statement:

python

```python
__enter__(self)           # called on entering the with block, return value bound to 'as' target
__exit__(self, exc_type, exc_val, exc_tb)  # called on exiting, even if exception occurred
```

python

```python
class ManagedFile:
    def __init__(self, path, mode):
        self.path = path
        self.mode = mode

    def __enter__(self):
        self.file = open(self.path, self.mode)
        return self.file               # bound to 'as' variable

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.file.close()
        return False                   # False = don't suppress exceptions
                                       # True  = suppress any exception that occurred
```

python

```python
with ManagedFile("data.txt", "r") as f:
    content = f.read()
# file is closed here regardless of whether an exception occurred
```

`__exit__` receives exception info if an exception was raised inside the `with` block. Returning `True` suppresses it. Returning `False` or `None` lets it propagate.

---

### Hashing and Identity

python

```python
__hash__(self)      # hash(obj) — must return an int
                    # equal objects must have equal hashes
                    # hash must not change during object's lifetime
```

python

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    def __hash__(self):
        return hash((self.x, self.y))   # hash a tuple of the fields
```

Hashing a tuple of the defining fields is the standard pattern. It ensures equal objects produce equal hashes, and the hash is consistent.

---

### Copy Protocol

python

```python
__copy__(self)        # called by copy.copy()  — shallow copy
__deepcopy__(self, memo)   # called by copy.deepcopy() — deep copy
```

python

```python
import copy

class MyClass:
    def __copy__(self):
        new = MyClass()
        new.__dict__.update(self.__dict__)
        return new

    def __deepcopy__(self, memo):
        new = MyClass()
        memo[id(self)] = new
        for k, v in self.__dict__.items():
            setattr(new, k, copy.deepcopy(v, memo))
        return new
```

`memo` in `__deepcopy__` is a dict that tracks already-copied objects to handle circular references — you must pass it through to any nested `deepcopy` calls.

---

### Pickling Protocol

Pickling is Python's object serialization — converting an object to bytes and back. Relevant for saving state, multiprocessing, and caching:

python

```python
__getstate__(self)          # return state to be pickled
__setstate__(self, state)   # restore state from unpickled data
__reduce__(self)            # full control over pickling
```

---

### Numeric Type Protocol — Full Picture

python

```python
__index__(self)    # called when Python needs an exact integer — for slicing,
                   # bin(), hex(), oct(). Must return an int.
                   # Different from __int__ — __index__ promises lossless conversion.

__round__(self, ndigits)    # round(obj, n)
__floor__(self)             # math.floor(obj)
__ceil__(self)              # math.ceil(obj)
__trunc__(self)             # math.trunc(obj)
```

---

### `__slots__`

Not strictly a dunder method — it's a class variable. But it changes how instances store attributes.

By default, every Python instance has a `__dict__` — a dictionary storing its attributes. This is flexible but uses memory. `__slots__` replaces `__dict__` with a fixed set of slots:

python

```python
class Point:
    __slots__ = ('x', 'y')    # only these attributes allowed

    def __init__(self, x, y):
        self.x = x
        self.y = y
```

python

```python
p = Point(1, 2)
p.x = 10       # fine
p.z = 5        # AttributeError — z not in __slots__
p.__dict__     # AttributeError — no __dict__
```

Benefits: less memory per instance, slightly faster attribute access. Cost: no dynamic attribute creation, no `__dict__`, some edge cases with multiple inheritance.

---

### Complete Reference

```
Lifecycle:        __new__, __init__, __del__
Representation:   __repr__, __str__, __format__, __bytes__
Comparison:       __eq__, __ne__, __lt__, __le__, __gt__, __ge__
Arithmetic:       __add__, __sub__, __mul__, __truediv__, __floordiv__,
                  __mod__, __pow__, __matmul__
Reflected arith:  __radd__, __rsub__, __rmul__, ... (all of the above prefixed r)
In-place arith:   __iadd__, __isub__, __imul__, ... (all prefixed i)
Unary:            __neg__, __pos__, __abs__, __invert__
Bitwise:          __and__, __or__, __xor__, __lshift__, __rshift__
Type conversion:  __bool__, __int__, __float__, __complex__, __index__
Numeric:          __round__, __floor__, __ceil__, __trunc__
Container:        __len__, __getitem__, __setitem__, __delitem__,
                  __contains__, __iter__, __next__, __reversed__
Callable:         __call__
Attribute access: __getattr__, __getattribute__, __setattr__, __delattr__, __dir__
Descriptor:       __get__, __set__, __delete__, __set_name__
Context manager:  __enter__, __exit__
Hashing:          __hash__
Copy:             __copy__, __deepcopy__
Pickle:           __getstate__, __setstate__, __reduce__
Class machinery:  __init_subclass__, __class_getitem__
```