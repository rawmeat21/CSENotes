
Python supports treating functions as [first-class objects](https://dbader.org/blog/python-first-class-functions). This means that _functions can be passed around and used as arguments_, just like [any other object like `str`, `int`, `float`, `list`, and so on](https://realpython.com/python-data-types/)

### Inner Functions[](https://realpython.com/primer-on-python-decorators/#inner-functions "Permanent link")

It’s possible to [define functions](https://realpython.com/defining-your-own-python-function/) _inside other functions_. Such functions are called [inner functions](https://realpython.com/inner-functions-what-are-they-good-for/). Here’s an example of a function with two inner functions:


```python
def parent():  
   
	print("Printing from parent()")   
	   
	def first_child():         
		print("Printing from first_child()")      
	def second_child():         
		print("Printing from second_child()")      
		
	second_child()     
	first_child()
```


## What are decorators?

A decorator in Python is a function that takes another function and extends its behavior without explicitly modifying it. Decorators are often used to add “wrapping” functionality to existing code in a concise and reusable manner.

### The Long Way First

python

```python
def shout(func):
    def wrapper():
        result = func()
        return result.upper()
    return wrapper

def greet():
    return "hello"

greet = shout(greet)    # manually decorating
greet()                 # "HELLO"
```

---

### The `@` Syntax

python

```python
def shout(func):
    def wrapper():
        result = func()
        return result.upper()
    return wrapper

@shout
def greet():
    return "hello"

greet()   # "HELLO"
```

`@shout` above `greet` is **exactly** equivalent to writing `greet = shout(greet)` after the definition. 

---

### Decorating Functions That Take Arguments

The `wrapper` above only works for functions with no arguments. To handle any function, use `*args` and `**kwargs` in the wrapper:

python

```python
def logger(func):
    def wrapper(*args, **kwargs):
        print(f"calling {func.__name__} with {args} {kwargs}")
        result = func(*args, **kwargs)         # pass everything through
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@logger
def add(a, b):
    return a + b

add(3, 4)
# calling add with (3, 4) {}
# add returned 7
```

`wrapper` receives whatever arguments the caller passes, and forwards them to the original function unchanged. This is the standard pattern for every decorator you write.

---

### `functools.wraps` — Preserving Identity

When you wrap a function, the wrapper replaces it. This means metadata like `__name__` and `__doc__` are lost:

python

```python
def logger(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@logger
def add(a, b):
    return a + b

add.__name__   # "wrapper" — wrong, should be "add"
```

Fix with `@functools.wraps`:

python

```python
import functools

def logger(func):
    @functools.wraps(func)        # copies __name__, __doc__, etc. from func to wrapper
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@logger
def add(a, b):
    return a + b

add.__name__   # "add" — correct
```

Always use `@functools.wraps` in decorators you write. It is the correct practice.

---

### Decorators With Arguments

Now: what if you want to pass arguments to the decorator itself?

python

```python
@repeat(3)
def greet():
    print("hello")
```

Here `repeat(3)` is called first — it must return a decorator. So you add one more layer:

python

```python
import functools

def repeat(n):                          # outer — takes decorator argument
    def decorator(func):                # middle — takes the function
        @functools.wraps(func)
        def wrapper(*args, **kwargs):   # inner — the actual replacement
            for _ in range(n):
                func(*args, **kwargs)
        return wrapper
    return decorator

@repeat(3)
def greet():
    print("hello")

greet()
# hello
# hello
# hello
```

The three layers:

```
repeat(3)         → returns decorator
decorator(greet)  → returns wrapper
wrapper()         → runs greet 3 times
```

`@repeat(3)` is equivalent to `greet = repeat(3)(greet)`. The extra `()` calls `repeat` first to get the decorator, then that decorator is applied to `greet`.

---

### Stacking Multiple Decorators

You can apply multiple decorators to one function:

python

```python
@decorator_a
@decorator_b
def func():
    ...
```

This is equivalent to:

python

```python
func = decorator_a(decorator_b(func))
```

They apply **bottom up** — `decorator_b` wraps `func` first, then `decorator_a` wraps that result.

python

```python
def bold(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return f"<b>{func(*args, **kwargs)}</b>"
    return wrapper

def italic(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return f"<i>{func(*args, **kwargs)}</i>"
    return wrapper

@bold
@italic
def text():
    return "hello"

text()   # "<b><i>hello</i></b>"
# italic wraps text first → "<i>hello</i>"
# bold wraps that result → "<b><i>hello</i></b>"
```

---

### Class-Based Decorators

A decorator just needs to be callable. A class with `__call__` is callable, so it can be a decorator:

python

```python
import functools

class Logger:
    def __init__(self, func):
        functools.update_wrapper(self, func)   # equivalent of @functools.wraps for classes
        self.func = func
        self.call_count = 0

    def __call__(self, *args, **kwargs):
        self.call_count += 1
        print(f"call #{self.call_count} to {self.func.__name__}")
        return self.func(*args, **kwargs)

@Logger
def add(a, b):
    return a + b

add(1, 2)   # call #1 to add
add(3, 4)   # call #2 to add
add.call_count   # 2 — state persists on the instance
```

The advantage over function-based decorators: you can store state on the instance without needing a closure.

---

### Decorating Methods in a Class

When decorating instance methods, `self` is just the first positional argument — `*args` captures it fine:

python

```python
def logger(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):      # args[0] will be self
        print(f"calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

class MyClass:
    @logger
    def my_method(self, x):
        return x * 2
```

No special handling needed — `self` flows through `*args` transparently.

---

### Real-World Patterns

#### Timing

python

```python
import time, functools

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} took {end - start:.4f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
```

#### Caching — `functools.lru_cache`

Python has a built-in caching decorator:

python

```python
from functools import lru_cache

@lru_cache(maxsize=128)    # caches up to 128 unique argument sets
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

fibonacci(50)   # fast — repeated calls hit cache
```

Without caching, `fibonacci(50)` would make exponential recursive calls. With `lru_cache`, each unique `n` is computed once and stored.

#### Access control

python

```python
def require_auth(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        if not is_logged_in():
            raise PermissionError("not authenticated")
        return func(*args, **kwargs)
    return wrapper

@require_auth
def get_user_data():
    ...
```

#### Input validation

python

```python
def positive_args(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        if any(a < 0 for a in args if isinstance(a, (int, float))):
            raise ValueError("all arguments must be positive")
        return func(*args, **kwargs)
    return wrapper

@positive_args
def sqrt(x):
    return x ** 0.5
```

---

### Summary

```
Basic decorator:

def decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # before
        result = func(*args, **kwargs)
        # after
        return result
    return wrapper


Decorator with arguments:

def decorator(arg):
    def inner(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # use arg here
            return func(*args, **kwargs)
        return wrapper
    return inner
```

|Syntax|Equivalent to|
|---|---|
|`@dec`|`f = dec(f)`|
|`@dec(arg)`|`f = dec(arg)(f)`|
|`@a` then `@b`|`f = a(b(f))`|

The `@` is always just shorthand for function reassignment. Every decorator pattern reduces to that one fact.