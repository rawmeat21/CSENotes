
## OOP in Python

---

### Defining a Class

python

```python
class Dog:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def bark(self):
        print(f"{self.name} says woof")
```

- `class` keyword, no template parameters
- `def __init__` is the constructor — called automatically when an object is created
- `self` is the first parameter of every instance method — it is the reference to the instance itself, equivalent to `this` in C++. Unlike C++, you must declare it explicitly in every method signature
- `self.name` creates an **instance attribute** — it belongs to that specific object

Creating an object:

python

```python
d = Dog("Rex", 3)
d.bark()
```

No `new` keyword. No `delete`. Memory is managed by Python's garbage collector.

---

### `self` in depth

In C++, `this` is an implicit pointer the compiler passes behind the scenes. In Python, `self` is an **explicit** first argument. When you call `d.bark()`, Python internally translates it to `Dog.bark(d)` — it passes the instance as the first argument. `self` is just the conventional name — you could name it anything, but never do.

---

### Instance vs Class Attributes

python

```python
class Dog:
    species = "Canis lupus"    # class attribute — shared across all instances

    def __init__(self, name):
        self.name = name       # instance attribute — unique per object
```

python

```python
a = Dog("Rex")
b = Dog("Max")

a.species    # "Canis lupus"
b.species    # "Canis lupus"
Dog.species  # "Canis lupus" — accessible on the class itself

a.name       # "Rex"
b.name       # "Max"
```

Class attributes are like `static` member variables in C++. They live on the class object, not on instances. If you assign to `a.species`, it creates a new **instance attribute** that shadows the class attribute for that object only — it does not modify the class attribute.

python

```python
a.species = "changed"
a.species      # "changed"   — instance attribute shadows class attribute
Dog.species    # "Canis lupus" — class attribute unchanged
b.species      # "Canis lupus" — b unaffected
```

---

### Access Control

Python has **no enforced access control**. No `private`, `protected`, `public` keywords. Everything is technically accessible. Instead, Python uses **naming conventions** to signal intent:

#### Public

No prefix. Accessible from anywhere.

python

```python
self.name = "Rex"
```

#### Protected — single underscore `_`

python

```python
self._age = 3
```

Convention meaning: "internal use, don't access from outside unless you know what you're doing." The underscore is a signal to other developers, not a language enforcement. You can still access `obj._age` from outside — Python won't stop you.

#### Private — double underscore `__`

python

```python
self.__salary = 50000
```

Double underscore triggers **name mangling**. Python internally renames `__salary` to `_ClassName__salary`. This means:

python

```python
class Employee:
    def __init__(self):
        self.__salary = 50000

e = Employee()
e.__salary              # AttributeError — name was mangled
e._Employee__salary     # 50000 — still accessible if you know the mangled name
```

Name mangling is not true privacy — it's a mechanism to **avoid name collisions in inheritance**, not to enforce security. The attribute is still reachable.

#### Summary

|Convention|Meaning|Enforced?|
|---|---|---|
|`name`|public|—|
|`_name`|protected, internal use|no|
|`__name`|private, name-mangled|no — just renamed|

### Types of Methods

#### Instance method

Regular method. Takes `self`. Operates on instance data.

python

```python
def bark(self):
    print(self.name)
```

#### Class method

Decorated with `@classmethod`. Takes `cls` (the class itself) as first argument instead of `self`. Can access and modify class attributes. Cannot access instance attributes.

python

```python
class Dog:
    count = 0

    def __init__(self):
        Dog.count += 1

    @classmethod
    def get_count(cls):
        return cls.count
```

`cls` refers to the class — like `self` but for the class object. Equivalent to `static` methods in C++ that access class-level state.

#### Static method

Decorated with `@staticmethod`. Takes no `self` or `cls`. It is a plain function that lives in the class namespace for organizational purposes. Cannot access instance or class data without explicit reference.

python

```python
class Math:
    @staticmethod
    def add(a, b):
        return a + b

Math.add(3, 4)   # 7
```

Equivalent to `static` methods in C++ that don't touch `this`.

#### Difference: classmethod vs staticmethod

||classmethod|staticmethod|
|---|---|---|
|First param|`cls` (the class)|nothing|
|Access class attrs|yes via `cls`|only if passed explicitly|
|Access instance attrs|no|no|
|Inheritance aware|yes — `cls` is the actual subclass|no|

### Constructors

Python has one constructor: `__init__`. But there are related methods:

#### `__init__` — initializer

Called after the object is created. Sets up the object's initial state.

python

```python
def __init__(self, name, age):
    self.name = name
    self.age = age
```

#### `__new__` — the actual constructor

Called **before** `__init__`. Actually creates and returns the new instance. `__init__` then receives that instance as `self`. You rarely override `__new__` unless implementing singletons, custom metaclasses, or immutable type subclassing.

python

```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

Every call to `Singleton()` returns the same object.

#### `__del__` — destructor

Called when the object is about to be garbage collected. Unreliable timing — do not depend on it for resource cleanup. Use context managers (`with` statement) instead.

python

```python
def __del__(self):
    print("object destroyed")
```

#### Multiple constructors?

C++ supports constructor overloading. Python does not — there can only be one `__init__`. The Pythonic ways to simulate multiple constructors:

**Default arguments:**

python

```python
def __init__(self, name, age=0):
    ...
```

**Class methods as alternative constructors:**

python

```python
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    @classmethod
    def from_string(cls, s):       # "2024-01-15"
        y, m, d = s.split('-')
        return cls(int(y), int(m), int(d))

    @classmethod
    def today(cls):
        import datetime
        t = datetime.date.today()
        return cls(t.year, t.month, t.day)

d1 = Date(2024, 1, 15)
d2 = Date.from_string("2024-01-15")
d3 = Date.today()
```

This is the standard Python pattern. `@classmethod` constructors are named, which is more readable than overloaded constructors.

---

### Inheritance

python

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        print(f"{self.name} makes a sound")

class Dog(Animal):
    def speak(self):
        print(f"{self.name} barks")
```

Syntax: `class Child(Parent)`. No `public`/`private`/`protected` inheritance distinction — Python only has one kind.

#### Calling the parent constructor — `super()`

python

```python
class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)    # calls Animal.__init__
        self.breed = breed
```

`super()` returns a proxy object that delegates method calls to the parent class.

If you don't call `super().__init__()`, the parent's initialization code doesn't run — the parent's attributes won't be set on the instance.

#### Method resolution — `super()` is not just for `__init__`

python

```python
class Dog(Animal):
    def speak(self):
        super().speak()           # calls Animal.speak first
        print(f"{self.name} also barks")
```

---

### Overriding

Python has method overriding — a subclass defines a method with the same name as the parent, and that definition takes precedence for instances of the subclass.

There is no `override` keyword like C++20. There is no `virtual` keyword — **all methods are virtual by default** in Python. Every method call is dynamically dispatched based on the actual type of the object at runtime.

python

```python
class Animal:
    def speak(self):
        print("some sound")

class Dog(Animal):
    def speak(self):       # overrides Animal.speak
        print("bark")

a = Animal()
d = Dog()
a.speak()    # "some sound"
d.speak()    # "bark"

# dynamic dispatch
animal_ref: Animal = Dog()
animal_ref.speak()    # "bark" — calls Dog's version, not Animal's
```

This is what C++ requires `virtual` to achieve. In Python it is always the behavior — no opt-in required.

---

### Abstract Classes

Python has abstract classes via the `abc` module (`abc` = Abstract Base Class).

python

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

    @abstractmethod
    def perimeter(self):
        pass

    def describe(self):                  # concrete method — not abstract
        print(f"Area is {self.area()}")
```

- Inherit from `ABC`
- Decorate abstract methods with `@abstractmethod`
- If a subclass doesn't implement all abstract methods, it cannot be instantiated

python

```python
class Circle(Shape):
    def __init__(self, r):
        self.r = r

    def area(self):
        return 3.14 * self.r * self.r

    def perimeter(self):
        return 2 * 3.14 * self.r

Shape()     # TypeError — cannot instantiate abstract class
Circle(5)   # works — all abstract methods implemented
```

If a subclass only implements some abstract methods, it is still abstract and cannot be instantiated.

Abstract properties also exist:

python

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @property
    @abstractmethod
    def sound(self):
        pass

class Dog(Animal):
    @property
    def sound(self):
        return "bark"
```

---

### Polymorphism

Python's polymorphism is **duck typing** — if an object has the required method, it works, regardless of its type. There is no need for a common base class or interface declaration.

python

```python
class Dog:
    def speak(self):
        return "bark"

class Cat:
    def speak(self):
        return "meow"

class Robot:
    def speak(self):
        return "beep boop"

def make_speak(obj):
    print(obj.speak())   # works for any object with a speak() method

make_speak(Dog())     # bark
make_speak(Cat())     # meow
make_speak(Robot())   # beep boop
```

`Dog`, `Cat`, `Robot` share no common parent, yet all work with `make_speak`. The only requirement is that the object has a `.speak()` method. This is duck typing — the type is not checked, only the presence of the method matters.

---

### Operator Overloading

Yes, Python supports operator overloading via **dunder methods** (double underscore methods, also called magic methods).

python

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):        # +
        return Vector(self.x + other.x, self.y + other.y)

    def __sub__(self, other):        # -
        return Vector(self.x - other.x, self.y - other.y)

    def __mul__(self, scalar):       # *
        return Vector(self.x * scalar, self.y * scalar)

    def __eq__(self, other):         # ==
        return self.x == other.x and self.y == other.y

    def __repr__(self):              # string representation
        return f"Vector({self.x}, {self.y})"

    def __len__(self):               # len()
        return int((self.x**2 + self.y**2) ** 0.5)

    def __getitem__(self, index):    # [] indexing
        return (self.x, self.y)[index]

    def __contains__(self, val):     # in operator
        return val in (self.x, self.y)
```

python

```python
a = Vector(1, 2)
b = Vector(3, 4)

a + b         # Vector(4, 6)
a == b        # False
print(a)      # Vector(1, 2)
len(a)        # 2
a[0]          # 1
1 in a        # True
```

#### Key dunder methods

|Dunder|Operator / function|
|---|---|
|`__add__`|`+`|
|`__sub__`|`-`|
|`__mul__`|`*`|
|`__truediv__`|`/`|
|`__floordiv__`|`//`|
|`__mod__`|`%`|
|`__pow__`|`**`|
|`__eq__`|`==`|
|`__ne__`|`!=`|
|`__lt__`|`<`|
|`__le__`|`<=`|
|`__gt__`|`>`|
|`__ge__`|`>=`|
|`__len__`|`len()`|
|`__str__`|`str()`, `print()`|
|`__repr__`|`repr()`, fallback for `__str__`|
|`__getitem__`|`obj[key]`|
|`__setitem__`|`obj[key] = val`|
|`__delitem__`|`del obj[key]`|
|`__contains__`|`in`|
|`__call__`|`obj()` — makes instance callable|
|`__iter__`|`for x in obj`|
|`__next__`|iteration protocol|
|`__enter__` / `__exit__`|`with` statement|
|`__hash__`|`hash()`|
|`__bool__`|`bool()`, truthiness|

#### `__str__` vs `__repr__`

- `__str__` — human-readable string. Used by `print()` and `str()`
- `__repr__` — unambiguous representation, ideally valid Python to recreate the object. Used in the REPL and as fallback when `__str__` is not defined

python

```python
def __repr__(self):
    return f"Vector({self.x}, {self.y})"   # recreatable

def __str__(self):
    return f"<{self.x}, {self.y}>"         # readable
```

---

### Multiple Inheritance

Python supports multiple inheritance directly:

python

```python
class A:
    def hello(self):
        print("A")

class B(A):
    def hello(self):
        print("B")

class C(A):
    def hello(self):
        print("C")

class D(B, C):
    pass

D().hello()   # "B"
```

Which `hello` does `D` call? This is resolved by the **MRO — Method Resolution Order**.

---

### Diamond Inheritance and MRO

The diamond problem:

```
    A
   / \
  B   C
   \ /
    D
```

In C++, this causes ambiguity and requires virtual inheritance. Python resolves it deterministically using the **C3 linearization algorithm**.

The MRO defines a linear order in which base classes are searched for a method. You can inspect it:

python

```python
D.__mro__
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

The rule: left to right through the inheritance list, depth-first, but a class is never placed before its subclasses. The result is always a consistent linear order with no class appearing twice.

So for `D(B, C)`:

1. `D` itself
2. `B` (left parent first)
3. `C` (right parent)
4. `A` (shared base, appears once at the end)
5. `object` (every class inherits from `object` implicitly)

When `D().hello()` is called, Python walks this list and calls the first `hello` it finds — which is `B.hello`.

#### `super()` and MRO

`super()` does not mean "call the parent class". It means "call the next class in the MRO". This is critical to understand in multiple inheritance:

python

```python
class A:
    def hello(self):
        print("A")

class B(A):
    def hello(self):
        print("B")
        super().hello()

class C(A):
    def hello(self):
        print("C")
        super().hello()

class D(B, C):
    def hello(self):
        print("D")
        super().hello()

D().hello()
# D
# B
# C
# A
```

MRO of D: `D → B → C → A`

- `D.hello` calls `super().hello()` → next in MRO is `B`
- `B.hello` calls `super().hello()` → next in MRO is `C`
- `C.hello` calls `super().hello()` → next in MRO is `A`
- `A.hello` prints and returns

Each `super()` call moves one step forward in the MRO — not to the direct parent. This cooperative design means each class in the chain gets called exactly once.

---

### Friendship

Python has **no friendship mechanism**. There is no `friend` keyword. No class can be granted special access to another class's private members.

Since Python's private members are just name-mangled and still technically accessible, and since there is no enforced access control, the concept of friendship is not needed — and not present.

---

### `isinstance()` and `issubclass()`

python

```python
class A: pass
class B(A): pass

b = B()

isinstance(b, B)    # True
isinstance(b, A)    # True — B is a subclass of A
isinstance(b, object)  # True — everything is

issubclass(B, A)    # True
issubclass(A, B)    # False
```

`isinstance` is the Python equivalent of C++'s `dynamic_cast` check. Use it instead of `type(obj) == SomeClass` — the latter does not account for inheritance.

---

### Properties — Getters and Setters

Python does not use `getX()` / `setX()` methods by convention. Instead, it uses the `@property` decorator to define getters and setters that look like attribute access:

python

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):           # getter
        return self._radius

    @radius.setter
    def radius(self, value):    # setter
        if value < 0:
            raise ValueError("Radius cannot be negative")
        self._radius = value

    @radius.deleter
    def radius(self):           # deleter
        del self._radius
```

python

```python
c = Circle(5)
c.radius         # calls getter — returns 5
c.radius = 10    # calls setter
c.radius = -1    # ValueError
del c.radius     # calls deleter
```

From the outside, it looks like plain attribute access. The property mechanism intercepts it. This is how Python achieves encapsulation without explicit getter/setter method calls.

---

### Dataclasses (Python 3.7+)

For classes that are primarily data containers, writing `__init__`, `__repr__`, `__eq__` manually is repetitive. `@dataclass` generates them automatically:

python

```python
from dataclasses import dataclass, field

@dataclass
class Point:
    x: float
    y: float
    z: float = 0.0   # default value

@dataclass
class Student:
    name: str
    grades: list = field(default_factory=list)  # mutable default — use field()
```

python

```python
p = Point(1.0, 2.0)
p            # Point(x=1.0, y=2.0, z=0.0)  — __repr__ generated
p == Point(1.0, 2.0)  # True — __eq__ generated
```

`@dataclass` generates: `__init__`, `__repr__`, `__eq__` by default.

Optional flags:

python

```python
@dataclass(order=True)   # generates __lt__, __le__, __gt__, __ge__
@dataclass(frozen=True)  # makes instance immutable — also makes it hashable
```
