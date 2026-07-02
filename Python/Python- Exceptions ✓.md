## Exception Handling in Python

---

### What is an Exception

An exception is an object. When something goes wrong at runtime, Python **raises** an exception object — creates it and propagates it up the call stack until something **catches** it or the program terminates.

In C++ exceptions are objects too, but Python's exception system is more uniform — everything that can be raised is an instance of a class that inherits from `BaseException`.

---

### Basic Syntax

python

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("cannot divide by zero")
```

- `try` block — code that might raise
- `except` block — code that runs if the specified exception is raised

If no exception is raised, the `except` block is skipped entirely.

---

### Catching the Exception Object

python

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(e)           # division by zero
    print(type(e))     # <class 'ZeroDivisionError'>
```

`as e` binds the exception object to the name `e`. You can inspect its message, type, and attributes.

---

### Multiple `except` Clauses

python

```python
try:
    x = int(input())
    result = 10 / x
except ValueError:
    print("not a valid integer")
except ZeroDivisionError:
    print("cannot divide by zero")
```

Python checks each `except` clause top to bottom and runs the **first one that matches**. Order matters — put more specific exceptions before more general ones.

---

### Catching Multiple Exceptions in One Clause

python

```python
try:
    ...
except (ValueError, TypeError) as e:
    print(f"value or type error: {e}")
```

A tuple of exception types in one `except` clause. Any of them will match.

---

### `else` Clause

Runs if the `try` block completed **without raising any exception**:

python

```python
try:
    result = 10 / 2
except ZeroDivisionError:
    print("error")
else:
    print(f"result is {result}")   # only runs if no exception
```

The distinction between putting code in `else` vs at the end of `try`:

- Code in `try` — if it raises, the `except` might catch it
- Code in `else` — exceptions here are **not** caught by the preceding `except` clauses

This makes intent clear: `try` contains only the risky operation, `else` contains what to do on success.

---

### `finally` Clause

Always runs — whether an exception was raised or not, whether it was caught or not:

python

```python
try:
    f = open("file.txt")
    data = f.read()
except FileNotFoundError:
    print("file not found")
finally:
    print("this always runs")
```

`finally` runs even if:

- No exception occurred
- An exception was raised and caught
- An exception was raised and **not** caught
- A `return`, `break`, or `continue` was executed inside `try`

Use case: cleanup that must happen regardless — closing files, releasing locks, closing network connections.

---

### Full Structure

python

```python
try:
    ...            # risky code
except SomeError:
    ...            # handle specific error
except (A, B):
    ...            # handle multiple
except Exception as e:
    ...            # handle any remaining
else:
    ...            # runs if no exception
finally:
    ...            # always runs
```

All four clauses together. In practice you use whichever subset you need.

---

### The Exception Hierarchy

Every exception is a class. The hierarchy matters because catching a parent class catches all its subclasses:

```
BaseException
├── SystemExit                  # raised by sys.exit()
├── KeyboardInterrupt           # Ctrl+C
├── GeneratorExit               # generator .close() called
└── Exception                   # all "normal" exceptions inherit from here
    ├── ArithmeticError
    │   ├── ZeroDivisionError
    │   ├── OverflowError
    │   └── FloatingPointError
    ├── LookupError
    │   ├── IndexError
    │   └── KeyError
    ├── ValueError
    ├── TypeError
    ├── AttributeError
    ├── NameError
    │   └── UnboundLocalError
    ├── OSError
    │   ├── FileNotFoundError
    │   ├── PermissionError
    │   ├── TimeoutError
    │   └── IsADirectoryError
    ├── RuntimeError
    │   └── RecursionError
    ├── StopIteration
    ├── ImportError
    │   └── ModuleNotFoundError
    ├── MemoryError
    ├── NotImplementedError
    └── AssertionError
```

`BaseException` is the root. `SystemExit`, `KeyboardInterrupt`, and `GeneratorExit` are direct children of `BaseException` — they intentionally sit outside `Exception` so that a bare `except Exception` does not accidentally swallow them.

python

```python
except Exception:     # catches everything except SystemExit, KeyboardInterrupt, GeneratorExit
except BaseException: # catches literally everything — almost never do this
```

---

### Specificity Order Matters

python

```python
try:
    ...
except Exception:        # too broad — catches everything
    ...
except ValueError:       # never reached — already caught above
    ...
```

Always put subclasses before parent classes:

python

```python
try:
    ...
except ValueError:       # specific first
    ...
except Exception:        # general fallback last
    ...
```

---

### Raising Exceptions

#### `raise ExceptionType(message)`

python

```python
def set_age(age):
    if age < 0:
        raise ValueError(f"Age cannot be negative, got {age}")
    return age
```

#### `raise` — re-raise current exception

Inside an `except` block, bare `raise` re-raises the caught exception without modifying it:

python

```python
try:
    risky()
except ValueError as e:
    log(e)
    raise        # re-raises the same ValueError
```

#### `raise X from Y` — exception chaining

Explicitly chains a new exception to the one that caused it:

python

```python
try:
    int("abc")
except ValueError as e:
    raise RuntimeError("conversion failed") from e
```

```
RuntimeError: conversion failed
  The above exception was the direct cause of the following exception:
ValueError: invalid literal for int() with base 10: 'abc'
```

The `__cause__` attribute of the new exception holds the original. Python displays both in the traceback.

#### `raise X from None` — suppress chaining

Raises a new exception and explicitly suppresses the implicit chaining context:

python

```python
try:
    int("abc")
except ValueError:
    raise RuntimeError("conversion failed") from None
```

Only `RuntimeError` appears in the traceback — no mention of `ValueError`.

---

### Exception Chaining — Implicit vs Explicit

When you raise inside an `except` block without `from`, Python still records the context:

python

```python
try:
    int("abc")
except ValueError:
    raise RuntimeError("something went wrong")
```

```
RuntimeError: something went wrong
During handling of the above exception, another exception occurred:
ValueError: invalid literal...
```

This is **implicit chaining** — `__context__` is set automatically.

`raise X from Y` is **explicit chaining** — sets `__cause__`, which displays differently in the traceback ("direct cause of" vs "during handling of").

`raise X from None` clears both `__cause__` and `__context__`.

---

### Custom Exceptions

Define by inheriting from `Exception` (or a more specific subclass):

python

```python
class AppError(Exception):
    pass

class ValidationError(AppError):
    def __init__(self, field, message):
        self.field = field
        self.message = message
        super().__init__(f"{field}: {message}")

class DatabaseError(AppError):
    pass
```


```python
try:
    raise ValidationError("email", "invalid format")
except ValidationError as e:
    print(e.field)    # "email"
    print(e.message)  # "invalid format"
    print(e)          # "email: invalid format"
except AppError:
    print("some app error")
```

#### Why define a hierarchy

Having a base `AppError` lets callers catch all your library's exceptions with one clause, or catch specific subtypes for fine-grained handling. Same design principle as the standard library hierarchy.

---

### `try` / `except` in a Loop

Common pattern — keep trying until success or explicit exit:

python

```python
while True:
    try:
        x = int(input("Enter a number: "))
        break
    except ValueError:
        print("That's not a number, try again")
```

---

### `contextlib.suppress`

Suppresses specified exceptions — cleaner than a try/except that does nothing:

python

```python
from contextlib import suppress

with suppress(FileNotFoundError):
    os.remove("maybe_exists.txt")
```

Equivalent to:

python

```python
try:
    os.remove("maybe_exists.txt")
except FileNotFoundError:
    pass
```

---

### `assert` Statement

Raises `AssertionError` if the condition is false:

python

```python
assert x > 0, f"x must be positive, got {x}"
```

Second argument is the error message. Used for **internal invariants** — conditions that should never be false if the code is correct. Not for user input validation.

Important: assertions are disabled when Python runs with the `-O` (optimize) flag. Never use `assert` for security checks or input validation that must run in production.

---

### Exception Attributes

Every exception has:

python

```python
e.args          # tuple of arguments passed to the constructor
str(e)          # string representation — usually the message
e.__traceback__ # traceback object
e.__cause__     # explicitly chained exception (from raise X from Y)
e.__context__   # implicitly chained exception
e.__suppress_context__  # True if raise X from None was used
```

#### Working with tracebacks

python

```python
import traceback

try:
    1 / 0
except ZeroDivisionError as e:
    traceback.print_exc()              # prints full traceback to stderr
    tb_str = traceback.format_exc()    # returns traceback as string
```

---

### `ExceptionGroup` (Python 3.11+)

Allows raising and catching multiple exceptions simultaneously. Relevant for concurrent code where several tasks may each raise:

python

```python
try:
    raise ExceptionGroup("multiple errors", [
        ValueError("bad value"),
        TypeError("bad type"),
        KeyError("missing key")
    ])
except* ValueError as eg:
    print(f"caught value errors: {eg.exceptions}")
except* TypeError as eg:
    print(f"caught type errors: {eg.exceptions}")
```

`except*` (with asterisk) matches all exceptions of that type within the group. Unmatched ones propagate. This is different from regular `except` — multiple `except*` clauses can all match and run.

---

### Common Exceptions and When They Occur

|Exception|When|
|---|---|
|`ValueError`|right type, wrong value — `int("abc")`|
|`TypeError`|wrong type — `"a" + 1`|
|`IndexError`|list index out of range|
|`KeyError`|dict key not found|
|`AttributeError`|attribute doesn't exist on object|
|`NameError`|variable not defined|
|`FileNotFoundError`|file doesn't exist|
|`PermissionError`|no permission to access file|
|`ZeroDivisionError`|division by zero|
|`OverflowError`|numeric result too large|
|`RecursionError`|max recursion depth exceeded|
|`StopIteration`|iterator exhausted|
|`NotImplementedError`|abstract method not implemented|
|`ImportError`|module not found or import failed|
|`MemoryError`|out of memory|
|`RuntimeError`|generic runtime error|
|`AssertionError`|`assert` condition failed|
|`OSError`||