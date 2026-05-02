## 2024 — Part II (Python)

---

### Q1(a) — Compare List and Tuple sequences in Python [5 marks]

**List** is a mutable, ordered sequence. Once created, elements can be added, removed, or modified. Defined using square brackets `[]`.

**Tuple** is an immutable, ordered sequence. Once created, it cannot be modified — no additions, deletions, or changes to elements. Defined using parentheses `()`.

|Property|List|Tuple|
|---|---|---|
|Mutability|Mutable|Immutable|
|Syntax|`[1, 2, 3]`|`(1, 2, 3)`|
|Methods available|Many — `append`, `remove`, `sort`, etc.|Few — only `count`, `index`|
|Memory|More (due to dynamic resizing overhead)|Less|
|Hashable|No — cannot be used as dictionary key|Yes — can be used as dictionary key if elements are hashable|
|Use case|When data needs to change|When data is fixed / should not change|
|Performance|Slightly slower for iteration|Slightly faster|

Both support indexing, slicing, and iteration. Both can hold heterogeneous data types. Tuples are commonly used for fixed data records, function return values, and as dictionary keys.

---

### Q1(b) — File object methods: read(), readline(), readlines(), seek(), write() [5 marks]

These are methods available on a file object obtained via `open()`.

**`read(size=-1)`** Reads the entire file content as a single string. If `size` is specified, reads at most that many characters. The file pointer advances to the end.

**`readline(size=-1)`** Reads one line at a time, up to and including the newline character `\n`. Each successive call reads the next line. If `size` is given, reads at most that many characters from the current line.

**`readlines()`** Reads all lines from the file and returns them as a **list of strings**, where each string is one line including the `\n`. Useful when you need random access to specific lines.

**`seek(offset, whence=0)`** Moves the file pointer to a specified position. `offset` is the number of bytes to move. `whence` controls the reference point:

- `0` — from the beginning of the file
- `1` — from the current position
- `2` — from the end of the file

This is the mechanism for random access in files.

**`write(string)`** Writes the given string to the file. Does **not** automatically add a newline. Returns the number of characters written. The file must be opened in write (`'w'`) or append (`'a'`) mode.

---

### Q1(c) — How does Generalization differ from Encapsulation [5 marks]

**Encapsulation** is the mechanism of bundling data (attributes) and the methods that operate on that data within a single unit (a class), while restricting direct access to the internal state from outside the class. In Python, this is achieved through naming conventions: a single underscore prefix (`_attr`) signals a protected member, and a double underscore prefix (`__attr`) triggers name mangling, effectively making the attribute private. The class exposes controlled access through getter/setter methods or properties. The purpose is to hide internal implementation details and protect object integrity.

```
class BankAccount:
    def __init__(self):
        self.__balance = 0       # private attribute

    def deposit(self, amount):
        self.__balance += amount

    def get_balance(self):
        return self.__balance
```

Here, `__balance` is hidden. The outside world can only interact with it via `deposit` and `get_balance`.

**Generalization** (also called Inheritance) is the mechanism by which a class (subclass/derived class) acquires the properties and behaviors of another class (superclass/base class). It represents an **"is-a"** relationship. The subclass inherits all non-private members of the superclass and can extend or override them.

In Python, this is written as:

```
class Animal:
    def speak(self):
        pass

class Dog(Animal):       # Dog generalizes from Animal
    def speak(self):
        return "Bark"
```

**Key difference:**

|Aspect|Encapsulation|Generalization|
|---|---|---|
|Concern|Data hiding and access control within a class|Reuse and hierarchy between classes|
|Relationship|Internal to a single class|Between two or more classes|
|Mechanism|Access modifiers, name mangling, properties|Inheritance (`class Sub(Base)`)|
|Purpose|Protect internal state|Promote code reuse, model hierarchy|

Encapsulation operates at the **class level** — it's about what is visible inside a class boundary. Generalization operates at the **design level** — it's about how classes relate to each other in a hierarchy.

---

### Q1(d) — Explain try, except, and finally block [5 marks]

Python's exception handling mechanism allows a program to respond to runtime errors without terminating abruptly.

**Structure:**

```
try:
    # code that may raise an exception
except ExceptionType as e:
    # code that executes if that exception occurs
else:
    # executes only if NO exception occurred in try
finally:
    # always executes, regardless of exception
```

**`try` block:** Contains the code that is expected to potentially raise an exception. Python monitors this block during execution. If an exception is raised anywhere in this block, execution of the block stops immediately and control transfers to the matching `except` block.

**`except` block:** Catches and handles the exception. You can specify:

- A specific exception type: `except ValueError`
- Multiple types: `except (ValueError, TypeError)`
- A base class: `except Exception` catches most exceptions
- No type: `except:` catches everything (not recommended)

You can have multiple `except` blocks for different exception types. Python checks them in order and executes only the first matching one.

**`else` block:** Executes if and only if the `try` block completed without raising any exception. Used to place code that should only run on success, keeping it separate from the error-handling logic.

**`finally` block:** Always executes — whether an exception was raised or not, whether it was caught or not. This makes it suitable for **cleanup operations** such as closing files, releasing locks, or closing network connections. Even if a `return` statement is hit in the `try` block, `finally` still runs.

**Example:**

python

```python
try:
    f = open("data.txt", "r")
    content = f.read()
    value = int(content)
except FileNotFoundError:
    print("File not found")
except ValueError:
    print("Content is not an integer")
else:
    print("Successfully read:", value)
finally:
    f.close()   # always runs
```

**Exception hierarchy in Python:** All exceptions inherit from `BaseException`. Most user-facing exceptions inherit from `Exception`. You can define custom exceptions by subclassing `Exception`.

---

### Q1(e) — Read lines from file, tokenize, convert tokens to unique numerical values using dictionary [5 marks]

**The problem:** Read a text file line by line. Split each line into tokens (words). Assign a unique integer to each unique token using a Python dictionary, so the same word always maps to the same number.

**Developer's thinking:**

- A dictionary maps each unique word → a unique integer ID.
- We maintain a counter. Every time we see a new word, we assign it the next counter value and store it.
- If the word is already in the dictionary, we skip assignment and just use the existing ID.

**Concept walkthrough:**

1. Open the file.
2. For each line, call `.split()` which splits on whitespace and returns a list of tokens.
3. For each token, check if it's already in the dictionary.
4. If not, add it with the current counter value, then increment the counter.
5. Print or store the numeric representation.

**Skeleton logic:**

python

```python
token_map = {}
counter = 0

with open("file.txt", "r") as f:
    for line in f:
        tokens = line.split()
        for token in tokens:
            if token not in token_map:
                token_map[token] = counter
                counter += 1
        numeric = [token_map[t] for t in tokens]
        print(numeric)
```

This is essentially building a **vocabulary index** — a standard technique in NLP preprocessing.

---

I'll continue with Q2, Q3, Q4, Q5 next. Want me to proceed immediately, or do you want to discuss any of the above first?

## You said: go ahead

go ahead

May 1

## Claude responded: 2024 — Part II (Python) continued

## 2024 — Part II (Python) continued

---

### Q2 — Discuss different mutable and immutable data types in Python [10 marks]

**Core concept:** In Python, every variable is a reference to an object in memory. The distinction between mutable and immutable is about whether the **object itself** can be changed after creation, not whether the variable can be reassigned.

When you do `x = 5` and then `x = 10`, you are not changing the object `5` — you are making `x` point to a different object `10`. The object `5` itself is unchanged. This is what immutability means at the object level.

---

#### Immutable Data Types

Once created, the object's value cannot be modified. Any operation that appears to "change" the value actually creates a new object.

**1. `int`** Integer objects are immutable. `a = 5; a += 1` does not modify the integer object `5`. It creates a new integer object `6` and binds `a` to it. Python also caches small integers (typically -5 to 256) for performance — the same object is reused for these values.

**2. `float`** Same behavior as `int`. Floating point numbers are immutable objects. Arithmetic always produces new objects.

**3. `bool`** A subclass of `int`. `True` and `False` are singleton immutable objects. `True == 1` and `False == 0`.

**4. `str`** Strings are immutable sequences of Unicode characters. You cannot change a character at a specific index. `s[0] = 'x'` raises a `TypeError`. Operations like concatenation (`s + "abc"`) or `.replace()` produce a **new string object**. The original string is untouched.

This has a performance implication: building a string by repeated concatenation in a loop is inefficient because each concatenation allocates a new string. The correct approach is to collect parts in a list and use `"".join()` at the end.

**5. `tuple`** An immutable ordered sequence. Once a tuple is created, elements cannot be added, removed, or replaced. `t[0] = 5` raises a `TypeError`. However, if a tuple contains a mutable object (like a list), the list itself can be modified — the tuple's references are fixed, but the objects those references point to may be mutable.

python

```python
t = (1, [2, 3], 4)
t[1].append(5)   # valid — modifying the list object, not the tuple
```

**6. `frozenset`** An immutable version of `set`. Once created, elements cannot be added or removed. Because it is immutable and hashable, it can be used as a dictionary key or as an element of another set.

**7. `bytes`** An immutable sequence of integers in the range 0–255. Represents raw binary data. The mutable counterpart is `bytearray`.

---

#### Mutable Data Types

The object's internal state can be modified in place after creation. The same object in memory is changed — no new object is allocated.

**1. `list`** An ordered, mutable sequence. Supports in-place operations:

- `append(x)` — adds element to end
- `insert(i, x)` — inserts at index
- `remove(x)` — removes first occurrence
- `pop(i)` — removes and returns element at index
- `sort()` — sorts in place (modifies the list, returns `None`)
- Direct index assignment: `lst[0] = 99`

Because lists are mutable, assignment does **not** copy:

python

```python
a = [1, 2, 3]
b = a         # both point to same list object
b.append(4)
print(a)      # [1, 2, 3, 4] — a is affected
```

To get an independent copy, use `b = a.copy()` or `b = a[:]`.

**2. `dict`** An unordered (insertion-ordered from Python 3.7+) collection of key-value pairs. Keys must be hashable (therefore immutable types). Values can be any type. Mutable operations include adding new keys, deleting keys, updating values.

python

```python
d = {"a": 1}
d["b"] = 2     # adds new key
del d["a"]     # removes key
d["b"] = 99    # modifies value
```

**3. `set`** An unordered collection of unique, hashable elements. Supports in-place operations: `add()`, `remove()`, `discard()`, `update()`. The elements themselves must be immutable (hashable), but the set container is mutable.

**4. `bytearray`** The mutable counterpart of `bytes`. Individual bytes can be modified by index assignment.

---

#### Why this distinction matters

**Hashability:** Only immutable (and therefore hashable) objects can be used as dictionary keys or set elements. `hash()` must return the same value for the lifetime of an object — mutable objects cannot guarantee this, so they are not hashable.

**Function arguments:** Python passes object references. If you pass a mutable object to a function and the function modifies it, the caller sees the change. If you pass an immutable object, the function cannot modify the original — any apparent modification just creates a new object locally.

**Memory and identity:** The `id()` function returns the memory address of an object. For immutable types, operations that "change" a value produce a new `id`. For mutable types, in-place operations preserve the `id`.

python

```python
a = [1, 2]
print(id(a))
a.append(3)
print(id(a))   # same id — same object modified

s = "hello"
print(id(s))
s += " world"
print(id(s))   # different id — new object created
```

---

### Q3 — Multithreading in Python [5+5 = 10 marks]

#### Part 1 — What is multithreading?

**Multithreading** is a programming model in which multiple threads of execution exist within the context of a single process. A **thread** is the smallest unit of execution managed by the operating system. All threads within a process share the same memory space — they share global variables, heap memory, and open file handles — but each thread has its own stack and program counter.

The purpose of multithreading is to allow concurrent execution of tasks within a single program. This is useful when:

- A program performs I/O-bound operations (file reading, network requests) where one thread can wait while another executes.
- A program has independent tasks that can logically proceed simultaneously.

**The GIL (Global Interpreter Lock):** In CPython (the standard Python implementation), there is a mechanism called the **Global Interpreter Lock**. The GIL ensures that only one thread executes Python bytecode at a time, even on multi-core processors. This means Python threads do **not** achieve true parallelism for CPU-bound tasks. However, the GIL is released during I/O operations, so multithreading is still effective for I/O-bound workloads.

For CPU-bound parallelism in Python, the `multiprocessing` module is used instead, as each process has its own GIL.

---

#### Part 2 — How to create a multithreading program in Python

Python provides the `threading` module for thread-based concurrency.

**Method 1: Passing a callable to `Thread`**

The `Thread` class accepts a `target` parameter — a callable (function) that the thread will execute — and an optional `args` tuple for arguments.

python

```python
import threading

def task(name):
    print(f"Thread {name} running")

t = threading.Thread(target=task, args=("A",))
t.start()    # starts the thread
t.join()     # blocks calling thread until t finishes
```

`start()` initiates the thread. `join()` makes the main thread wait for the child thread to complete before proceeding.

**Method 2: Subclassing `Thread`**

You create a class that extends `threading.Thread` and override the `run()` method. When `start()` is called, it internally calls `run()`.

python

```python
import threading

class MyThread(threading.Thread):
    def __init__(self, name):
        super().__init__()
        self.name = name

    def run(self):
        print(f"Thread {self.name} executing")

t = MyThread("Worker-1")
t.start()
t.join()
```

**Synchronization — `Lock`:**

When multiple threads access shared data, race conditions can occur — the outcome depends on the order of thread scheduling, which is non-deterministic. The solution is mutual exclusion using a `Lock`.

python

```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    with lock:          # acquires lock, releases on exit
        counter += 1

threads = [threading.Thread(target=increment) for _ in range(100)]
for t in threads: t.start()
for t in threads: t.join()
print(counter)   # guaranteed to be 100
```

`with lock:` is equivalent to calling `lock.acquire()` at the start and `lock.release()` at the end, but is safer because it guarantees release even if an exception occurs.

**`wait()` and `notify()` — Condition variables:**

For producer-consumer style coordination, Python provides `threading.Condition`. A `Condition` wraps a lock and adds `wait()` and `notify()` methods.

- `wait()` — releases the lock and blocks the thread until another thread calls `notify()` on the same condition.
- `notify()` — wakes up one waiting thread.
- `notify_all()` — wakes up all waiting threads.

python

```python
condition = threading.Condition()

# Consumer
with condition:
    while not data_available:
        condition.wait()
    # process data

# Producer
with condition:
    # produce data
    condition.notify()
```

---

### Q4 — Compare OOP and Event-driven programming. Advantages of Event-driven programming [5+5 = 10 marks]

_(Skipping GUI code as instructed)_

#### Object-Oriented Programming (OOP)

OOP is a programming paradigm where a program is structured as a collection of objects. Each object is an instance of a class, encapsulating both state (attributes) and behavior (methods). Program execution follows a defined flow — the programmer controls what happens and in what sequence. The main program calls methods, which call other methods, and execution proceeds according to the code's logic.

The four pillars are: **Encapsulation**, **Inheritance**, **Polymorphism**, and **Abstraction**.

Control flow is explicit and sequential — the program drives execution.

#### Event-Driven Programming

Event-driven programming is a paradigm in which the flow of program execution is determined by **events** — external or internal signals such as user interactions (mouse clicks, key presses), messages from other programs, or hardware interrupts.

The program does not call procedures sequentially. Instead, it:

1. Sets up an **event loop** that continuously listens for events.
2. Registers **event handlers** (also called listeners or callbacks) — functions that are bound to specific event types.
3. When an event occurs, the event loop dispatches it to the appropriate handler.
4. The handler executes and returns control to the event loop.

The program is **reactive** — it responds to events rather than driving a sequence.

#### Comparison

|Aspect|OOP|Event-Driven|
|---|---|---|
|Control flow|Sequential, programmer-driven|Reactive, event-driven|
|Execution trigger|Method calls|Events (user input, messages)|
|Structure|Classes and objects|Event loop + event handlers|
|Predictability|Flow is deterministic from code|Flow depends on runtime events|
|Use case|General-purpose computation|GUIs, servers, real-time systems|
|Coupling|Tight — caller knows the callee|Loose — handler registered, not called directly|

Note: The two paradigms are **not mutually exclusive**. GUI frameworks like Tkinter are OOP-based but use event-driven execution. Classes define components; events trigger their behavior.

#### Advantages of Event-Driven Programming

**1. Responsiveness:** The event loop continuously polls for events, so the program remains responsive to user input at all times. The UI does not freeze because handlers execute quickly and return control to the loop.

**2. Loose coupling:** Event producers (sources of events) do not need to know about event consumers (handlers). A button does not call a specific function directly — it fires an event, and whoever registered a handler for that event responds. This makes components independent and replaceable.

**3. Modularity:** Each event handler is a self-contained unit responsible for one type of response. Adding new behavior means registering a new handler, not modifying existing code. This aligns with the Open/Closed principle.

**4. Scalability:** Event-driven architectures handle many concurrent inputs efficiently. A single event loop can manage thousands of events without spawning a new thread for each, making it suitable for servers and networked applications.

**5. Natural fit for interactive systems:** User interfaces are inherently event-driven — a user's actions are unpredictable in timing and type. Event-driven programming directly models this reality, making it the natural paradigm for GUI, game, and real-time system development.

**6. Asynchronous operation:** Event-driven programs naturally support asynchronous behavior — they do not block waiting for slow operations. While waiting for a file to load or a network response, the event loop can process other events.

---

### Q5 — Search for palindrome and unique words using class method and string method [5+5 = 10 marks]

**OR — Fibonacci using generator, trace, memorize, pickling/unpickling [5+5 = 10 marks]**

I'll cover both since both appear across years.

---

#### Palindrome and unique words

**Palindrome:** A word that reads the same forwards and backwards. `"racecar"` reversed is `"racecar"`. The string method `.lower()` is used to normalize case before comparison, and `[::-1]` is the slice syntax for reversing a string.

**Unique words:** Words that appear exactly once in the text.

**Developer's thinking:**

- Split the text into words using `.split()`.
- Normalize each word — strip punctuation, convert to lowercase — so `"Racecar,"` and `"racecar"` are treated as the same.
- For palindrome check: compare word to its reverse.
- For unique words: use a dictionary or `Counter` to count occurrences, then filter those with count == 1.

**Using a class with class methods and string methods:**

A **class method** is a method bound to the class rather than an instance. It receives the class (`cls`) as the first argument instead of `self`. Decorated with `@classmethod`. It can be called on the class directly without creating an object.

python

```python
class TextAnalyzer:

    @classmethod
    def find_palindromes(cls, text):
        words = text.lower().split()
        cleaned = [w.strip(".,!?") for w in words]
        return [w for w in cleaned if w == w[::-1] and len(w) > 1]

    @classmethod
    def find_unique(cls, text):
        words = text.lower().split()
        cleaned = [w.strip(".,!?") for w in words]
        freq = {}
        for w in cleaned:
            freq[w] = freq.get(w, 0) + 1
        return [w for w, count in freq.items() if count == 1]
```

Called as `TextAnalyzer.find_palindromes("racecar level hello world hello")` — no object instantiation needed.

---

#### Fibonacci using generator, trace, pickling/unpickling

**Generator:** A generator is a function that uses the `yield` keyword instead of `return`. When called, it returns a **generator object** — it does not execute the function body immediately. Each call to `next()` on the generator resumes execution from where it last `yield`ed, runs until the next `yield`, and pauses again. This allows generating values **lazily** — one at a time, without computing or storing all values in memory.

python

```python
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b
```

**Using `next()` to get first seven:**

python

```python
gen = fibonacci()
for _ in range(7):
    print(next(gen))
```

**Trace of execution for first 7 values:**

|Call|`a` before yield|Yielded|After yield: new `a, b`|
|---|---|---|---|
|1st `next()`|0|0|a=1, b=1|
|2nd `next()`|1|1|a=1, b=2|
|3rd `next()`|1|1|a=2, b=3|
|4th `next()`|2|2|a=3, b=5|
|5th `next()`|3|3|a=5, b=8|
|6th `next()`|5|5|a=8, b=13|
|7th `next()`|8|8|a=13, b=21|

**Memorize (Memoization):** Memoization is an optimization technique where the results of expensive function calls are cached so that repeated calls with the same arguments return the cached result instead of recomputing. In Python, `functools.lru_cache` decorator implements this automatically.

python

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    if n <= 1:
        return n
    return fib(n-1) + fib(n-2)
```

Without memoization, recursive Fibonacci has exponential time complexity O(2^n) because the same subproblems are recomputed repeatedly. With memoization, it becomes O(n).

**Pickling and Unpickling:**

**Pickling** is the process of serializing a Python object into a byte stream. The `pickle` module converts an in-memory Python object (list, dict, class instance, etc.) into a sequence of bytes that can be written to a file or transmitted over a network.

**Unpickling** is the reverse — deserializing a byte stream back into a Python object.

python

```python
import pickle

data = [0, 1, 1, 2, 3, 5, 8]

# Pickling — writing to file
with open("fib.pkl", "wb") as f:
    pickle.dump(data, f)

# Unpickling — reading from file
with open("fib.pkl", "rb") as f:
    loaded = pickle.load(f)
```

`pickle.dump(obj, file)` — serializes and writes. `pickle.load(file)` — reads and deserializes.

**Important:** Pickle is Python-specific — the byte stream is not readable by other languages. It is also not secure against maliciously crafted data — never unpickle data from an untrusted source.

---

That completes all of 2024 Python. Want me to move on to **2023** now?

## You said: Yes, you will find some questions that get repeated.

Yes, you will find some questions that get repeated. Skip them

May 1

## Claude responded: 2023 — Part B (Python)

## 2023 — Part B (Python)

---

### Q1 — Discuss advantages of Python over Java as an OOP Language [10 marks]

**1. Dynamic Typing:** Python uses dynamic typing — the type of a variable is determined at runtime, not at compile time. There is no need to declare variable types explicitly. This reduces boilerplate and allows faster prototyping. Java is statically typed — every variable must be declared with an explicit type, and type errors are caught at compile time.

**2. Simpler Syntax for OOP constructs:** In Python, defining a class requires minimal syntax. There are no access modifier keywords (`public`, `private`, `protected`) as mandatory declarations — access control is convention-based. There are no type declarations for attributes. This results in significantly less code for equivalent functionality.

**3. Multiple Inheritance:** Python directly supports multiple inheritance — a class can inherit from more than one base class simultaneously. Java does not support multiple inheritance of classes (only of interfaces) to avoid the diamond problem. Python resolves the diamond problem using the **Method Resolution Order (MRO)** — specifically the C3 linearization algorithm — which defines a deterministic order in which base classes are searched.

**4. Duck Typing and Polymorphism:** Python's polymorphism is based on duck typing — if an object has the required method or attribute, it can be used in that context, regardless of its class hierarchy. There is no requirement to implement a specific interface or extend a specific class. Java's polymorphism is type-hierarchy-based — an object must be an instance of the expected type or interface.

**5. No need for explicit getter/setter methods:** Python provides the `@property` decorator, which allows a method to be accessed like an attribute. This gives the syntactic simplicity of direct attribute access while retaining the ability to add logic (validation, computation) transparently. In Java, encapsulation typically requires explicit `getX()` and `setX()` methods.

**6. First-class functions and operator overloading:** Python treats functions as first-class objects — they can be passed as arguments, returned from functions, and stored in data structures. Python also supports operator overloading through dunder (double underscore) methods — `__add__`, `__eq__`, `__lt__`, etc. — allowing objects to define behavior for standard operators. Java does not support operator overloading.

**7. Built-in data structures:** Python provides built-in, flexible data structures (list, dict, set, tuple) that work seamlessly with OOP. Java requires importing collection classes and specifying generic types, resulting in more verbose code.

**8. Introspection and Reflection:** Python has extensive built-in introspection capabilities. Functions like `type()`, `isinstance()`, `hasattr()`, `getattr()`, `dir()` allow examining and manipulating objects at runtime with minimal effort. While Java has reflection, it is significantly more verbose and complex to use.

**9. Less ceremony for class design:** Python does not require everything to be inside a class. Utility functions can exist at the module level. Java mandates that all code reside inside a class, even when a class structure is not logically necessary (e.g., a class with only a `main` method).

**10. Interactive development:** Python can be run interactively via the REPL (Read-Eval-Print Loop), allowing developers to test class behavior, inspect objects, and experiment with OOP designs instantly without a compile step.

---

### Q2 — Why is Python called a dynamic language? Discuss chr(), int(), oct(), complex(), float() [5+5 = 10 marks]

#### Part 1 — Python as a dynamic language

Python is called a **dynamically typed** language because type checking is performed at **runtime** rather than at compile time. Variables do not have fixed types — a variable can hold an integer at one point and a string at another. The type is a property of the **object**, not the **variable**.

python

```python
x = 10       # x refers to an int object
x = "hello"  # x now refers to a str object — valid in Python
```

In addition to dynamic typing, Python is also **dynamically bound** — method resolution (which method gets called) is determined at runtime based on the actual type of the object, not the declared type.

Python is also considered a **strongly typed** language despite being dynamic — it does not perform implicit type coercions between unrelated types. `"5" + 5` raises a `TypeError` rather than silently converting. This distinguishes it from weakly typed languages like JavaScript.

The combination of dynamic typing and strong typing means: types are flexible at the variable level but strict at the operation level.

---

#### Part 2 — Type conversion functions

These are built-in functions that explicitly convert a value from one type to another.

**`int(x, base=10)`** Converts `x` to an integer. If `x` is a float, it truncates (does not round) the decimal part. If `x` is a string, it parses it as an integer in the given base.

python

```python
int(3.9)       # 3 — truncated, not rounded
int("42")      # 42
int("1010", 2) # 10 — binary string to int
int("FF", 16)  # 255 — hex string to int
```

**`float(x)`** Converts `x` to a floating-point number.

python

```python
float(5)       # 5.0
float("3.14")  # 3.14
float("inf")   # inf — Python supports infinity
```

**`chr(i)`** Converts an integer to the Unicode character it represents. The integer must be in the range 0 to 1,114,111 (the Unicode code point range).

python

```python
chr(65)    # 'A'
chr(97)    # 'a'
chr(8364)  # '€'
```

The inverse function is `ord(c)`, which converts a character back to its Unicode code point.

**`oct(x)`** Converts an integer to its octal (base-8) string representation. The result is always prefixed with `'0o'` to indicate octal.

python

```python
oct(8)    # '0o10'
oct(255)  # '0o377'
oct(64)   # '0o100'
```

Note: `oct()` only accepts integers, not floats.

**`complex(real, imag=0)`** Creates a complex number with a real and imaginary part. Both parts are stored as floats internally. Can also accept a string representation.

python

```python
complex(3, 4)     # (3+4j)
complex(5)        # (5+0j)
complex("3+4j")   # (3+4j) — parsed from string
```

Complex numbers support arithmetic operations. Attributes `.real` and `.imag` extract the respective parts. The `abs()` function returns the magnitude (modulus).

---

### Q3 — Three dictionaries, concatenate into fourth, check if empty [5+5 = 10 marks]

#### Part 1 — Creating and concatenating dictionaries

A **dictionary** in Python is a mutable, unordered (insertion-ordered from 3.7+) collection of key-value pairs. Keys must be unique and hashable.

**Creating three dictionaries:**

python

```python
d1 = {"a": 1, "b": 2}
d2 = {"c": 3, "d": 4}
d3 = {"e": 5, "f": 6}
```

**Methods to concatenate (merge) into a fourth dictionary:**

**Method 1 — `update()`:** `update()` merges another dictionary into the calling dictionary in place. To create a fourth without modifying originals:

python

```python
d4 = {}
d4.update(d1)
d4.update(d2)
d4.update(d3)
```

If there are duplicate keys, the last `update()` call's value overwrites the earlier one.

**Method 2 — Unpacking with `**` (Python 3.5+):** The `**` operator unpacks a dictionary into keyword arguments. Inside a dict literal, it spreads the key-value pairs.

python

```python
d4 = {**d1, **d2, **d3}
```

This is the most concise and Pythonic approach. Again, later keys overwrite earlier ones on conflict.

**Method 3 — `|` operator (Python 3.9+):**

python

```python
d4 = d1 | d2 | d3
```

Returns a new merged dictionary.

---

#### Part 2 — Check if a dictionary is empty

A dictionary is empty if it contains no key-value pairs.

**Method 1 — Direct boolean evaluation:** An empty dictionary evaluates to `False` in a boolean context. A non-empty dictionary evaluates to `True`. This is the most Pythonic approach.

python

```python
if not d4:
    print("Dictionary is empty")
else:
    print("Dictionary is not empty")
```

**Method 2 — Using `len()`:** `len(d)` returns the number of key-value pairs. An empty dictionary has length 0.

python

```python
if len(d4) == 0:
    print("Empty")
```

**Method 3 — Comparing to empty dict literal:**

python

```python
if d4 == {}:
    print("Empty")
```

This works but is less idiomatic than the boolean evaluation approach.

---

### Q6 — List functions: len(), sum(), any(), all(), sorted(). Remove duplicate words and sort [5+5 = 10 marks]

#### Part 1 — List functions

**`len(iterable)`** Returns the number of elements in the iterable. For a list, it returns the count of top-level elements (not recursive).

python

```python
len([1, 2, 3])        # 3
len([[1, 2], [3, 4]]) # 2 — two sublists, not four elements
```

**`sum(iterable, start=0)`** Returns the arithmetic sum of all elements. The optional `start` parameter is added to the total. Elements must be numeric.

python

```python
sum([1, 2, 3, 4])      # 10
sum([1, 2, 3], 10)     # 16 — 10 + 1 + 2 + 3
```

**`any(iterable)`** Returns `True` if **at least one** element of the iterable is truthy. Returns `False` if the iterable is empty or all elements are falsy. Uses short-circuit evaluation — stops at the first truthy value.

python

```python
any([0, 0, 1, 0])   # True
any([0, 0, 0])      # False
any([])             # False
```

**`all(iterable)`** Returns `True` if **all** elements of the iterable are truthy. Returns `True` for an empty iterable (vacuous truth). Uses short-circuit evaluation — stops at the first falsy value.

python

```python
all([1, 2, 3])      # True
all([1, 0, 3])      # False
all([])             # True
```

**`sorted(iterable, key=None, reverse=False)`** Returns a **new sorted list** from the elements of any iterable. Does not modify the original. The `key` parameter accepts a function that is applied to each element before comparison. The `reverse` parameter controls ascending (default) or descending order.

This is distinct from `list.sort()`, which sorts in place and returns `None`.

python

```python
sorted([3, 1, 2])                    # [1, 2, 3]
sorted([3, 1, 2], reverse=True)      # [3, 2, 1]
sorted(["banana", "apple", "cherry"],
       key=len)                      # ['apple', 'banana', 'cherry']
```

---

#### Part 2 — Remove duplicate words and sort alphanumerically

**Developer's thinking:**

- Split the string into words.
- Eliminate duplicates — a `set` does this automatically since sets only hold unique elements.
- Sort the result.
- Join back into a string.

python

```python
def process(text):
    words = text.split()
    unique = set(words)
    result = sorted(unique)        # sorted() on a set returns a sorted list
    return " ".join(result)
```

`sorted()` on a set performs **alphanumeric** (lexicographic) ordering by default — uppercase letters come before lowercase in ASCII, so to make it fully case-insensitive, pass `key=str.lower`.

---

### Q7 — Generalization vs Encapsulation + Monkey Patching [5+5 = 10 marks]

**Generalization vs Encapsulation** — already covered in 2024 Q1(c). Skip.

#### Monkey Patching

**Monkey patching** is the practice of dynamically modifying or extending a class or module at runtime by replacing or adding attributes or methods outside of the original class definition.

In Python, because classes are objects and their attributes are stored in a mutable dictionary (`__dict__`), it is possible to assign new methods or replace existing ones at runtime.

python

```python
class MyClass:
    def greet(self):
        return "Hello"

def new_greet(self):
    return "Hi there"

MyClass.greet = new_greet   # replacing the method at runtime

obj = MyClass()
obj.greet()   # returns "Hi there"
```

You can also patch individual instances:

python

```python
import types
obj.greet = types.MethodType(new_greet, obj)
```

**Use cases:**

- In testing — replacing a method with a mock implementation to isolate behavior without modifying the original class.
- Applying fixes or extensions to third-party libraries without modifying their source code.

**Risks:**

- Makes code harder to understand and debug — behavior of a class may differ from its definition in the source file.
- Can cause conflicts if multiple patches target the same method.
- Breaks encapsulation principles — internal behavior is altered from outside.

Monkey patching is a feature enabled by Python's dynamic nature. Frameworks like `pytest` use it extensively via `monkeypatch` fixture for test isolation.

---

### Q8 — Functional programming in Python + Stack with IndexError [5+5 = 10 marks]

#### Part 1 — Functional programming features in Python

Functional programming is a paradigm where computation is expressed as the evaluation of mathematical functions, avoiding mutable state and side effects. Python is a multi-paradigm language that supports functional programming features alongside OOP.

**1. First-class and higher-order functions:** Functions in Python are first-class objects — they can be assigned to variables, passed as arguments, and returned from other functions. A higher-order function is one that takes a function as an argument or returns one.

python

```python
def apply(func, value):
    return func(value)

apply(str.upper, "hello")   # 'HELLO'
```

**2. `lambda` functions:** Anonymous, single-expression functions defined inline. Used where a full `def` would be unnecessarily verbose, typically as arguments to higher-order functions.

python

```python
square = lambda x: x ** 2
sorted([3, 1, 2], key=lambda x: -x)   # [3, 2, 1]
```

**3. `map(func, iterable)`:** Applies a function to every element of an iterable and returns a map object (lazy iterator). Does not modify the original.

python

```python
list(map(lambda x: x*2, [1, 2, 3]))   # [2, 4, 6]
```

**4. `filter(func, iterable)`:** Returns an iterator of elements for which the function returns `True`.

python

```python
list(filter(lambda x: x % 2 == 0, [1, 2, 3, 4]))   # [2, 4]
```

**5. `reduce(func, iterable)` — from `functools`:** Applies a function cumulatively to the elements of a sequence, reducing it to a single value.

python

```python
from functools import reduce
reduce(lambda x, y: x + y, [1, 2, 3, 4])   # 10
```

**6. List comprehensions:** A concise, Pythonic way to construct lists using a declarative expression. Functionally equivalent to `map` + `filter` combined.

python

```python
[x**2 for x in range(10) if x % 2 == 0]
```

**7. Immutability and pure functions:** Python does not enforce immutability, but functional style encourages writing pure functions — functions that always return the same output for the same input and have no side effects.

**8. Closures:** A closure is a function that captures variables from its enclosing scope, even after the outer function has returned. This enables state to be stored in a function without using a class.

python

```python
def counter():
    count = 0
    def increment():
        nonlocal count
        count += 1
        return count
    return increment
```

---

#### Part 2 — Stack with IndexError

**Developer's thinking:** A stack is a Last-In-First-Out (LIFO) data structure with two primary operations — push (add to top) and pop (remove from top). We need a fixed-size stack that raises `IndexError` on overflow (push when full) and underflow (pop when empty).

We use a Python list as the internal storage and track size with `maxsize`.

python

```python
class Stack:
    def __init__(self, maxsize):
        self.maxsize = maxsize
        self.data = []

    def push(self, item):
        if len(self.data) >= self.maxsize:
            raise IndexError("Stack is full")
        self.data.append(item)

    def pop(self):
        if len(self.data) == 0:
            raise IndexError("Stack is empty")
        return self.data.pop()

    def peek(self):
        if not self.data:
            raise IndexError("Stack is empty")
        return self.data[-1]
```

`IndexError` is the semantically appropriate exception here — it signals an invalid index-based access, which is what an overflow/underflow represents on a bounded sequence structure.

---

That completes 2023. Moving to **2022** next — I expect significant overlap. Want me to proceed?

## You said: Yes, go

Yes, go

May 1

## Claude responded: 2022 — Part B (Python)

## 2022 — Part B (Python)

---

### Q1 — Advantages of Python over Java as OOP Language [10 marks]

Already covered in 2023 Q1. Skip.

---

### Q2 — Why Python is dynamic and strongly typed? Discuss ord(), hex(), oct(), complex(), float() [5+5 = 10 marks]

#### Part 1 — Dynamic and strongly typed

Already covered in 2023 Q2 Part 1. Skip.

#### Part 2 — Type conversion functions

`oct()`, `float()`, `complex()` already covered in 2023 Q2. Covering the new ones:

**`ord(c)`** The inverse of `chr()`. Takes a single Unicode character and returns its integer Unicode code point.

python

```python
ord('A')    # 65
ord('a')    # 97
ord('€')    # 8364
```

Raises `TypeError` if the argument is not a single character string.

**`hex(x)`** Converts an integer to its hexadecimal (base-16) string representation. The result is always prefixed with `'0x'` to indicate hexadecimal. Accepts only integers.

python

```python
hex(255)    # '0xff'
hex(16)     # '0x10'
hex(0)      # '0x0'
```

For negative numbers: `hex(-255)` gives `'-0xff'`. To get uppercase hex, call `.upper()` on the result or use f-string formatting: `f"{255:X}"`.

---

### Q3 — Search for palindrome and unique words [5+5 = 10 marks]

Already covered in 2024 Q5. Skip.

---

### Q4 — GUI application in Python — file reading with Tkinter [10 marks]

Skipping GUI as instructed.

---

### Q5 — Connection-oriented server in Python [10 marks]

Skipping socket programming as instructed.

---

### Q6 — List functions + First seven Fibonacci using generator + Trace + Memorize [5+5 = 10 marks]

List functions already covered in 2023 Q6. Fibonacci generator, trace, and memoization already covered in 2024 Q5. Skip.

---

### Q7 — Create list of Tuples (item, price), sort descending by price using operator.itemgetter(). Prove dictionary returned by globals() can manipulate variables [5+5 = 10 marks]

#### Part 1 — List of Tuples, sort by price descending using `operator.itemgetter()`

**Tuples as records:** A tuple can represent a fixed record — here, each tuple is `(item_name, price)`. A list of such tuples is a common pattern for tabular data in Python.

**`operator.itemgetter(index)`:** `operator.itemgetter(n)` returns a callable that, when applied to a sequence, retrieves the element at index `n`. It is used as the `key` argument in `sorted()` or `list.sort()`. It is functionally equivalent to `lambda x: x[n]` but is implemented in C and is therefore faster.

python

```python
from operator import itemgetter

items = [
    ("pen", 15.5),
    ("notebook", 45.0),
    ("eraser", 5.0),
    ("ruler", 20.0)
]

sorted_items = sorted(items, key=itemgetter(1), reverse=True)
# [('notebook', 45.0), ('ruler', 20.0), ('pen', 15.5), ('eraser', 5.0)]
```

`itemgetter(1)` extracts index 1 (the price) from each tuple. `reverse=True` gives descending order.

`itemgetter` can also accept multiple indices — `itemgetter(1, 0)` would sort by price first, then by name — useful for multi-level sorting.

---

#### Part 2 — `globals()` and variable manipulation

**`globals()`** returns a reference to the **current module's global symbol table** as a dictionary. The keys are variable names (as strings), and the values are the objects those variables refer to.

The critical point is that this is not a copy — it is the **actual dictionary** that Python uses internally to resolve global variable names. Modifying it directly modifies the actual global variables.

python

```python
x = 10
print(globals()['x'])    # 10 — reading x through globals()

globals()['x'] = 99      # modifying x through globals()
print(x)                 # 99 — x is actually changed

globals()['y'] = 42      # creating a new variable y
print(y)                 # 42 — y now exists in global scope
```

This proves that `globals()` is not a snapshot — it is a live reference to the actual namespace dictionary Python uses for name resolution. When Python encounters a global variable name, it looks it up in exactly this dictionary.

**`locals()`** works similarly for the local scope, but modifying `locals()` does **not** reliably affect actual local variables — the behavior is implementation-defined. `globals()` modification is reliable and well-defined.

This behavior is a direct consequence of Python's data model: namespaces are dictionaries, and variable access is dictionary lookup.

---

### Q8 — Functional programming + Stack with IndexError [5+5 = 10 marks]

Already covered in 2023 Q8. Skip.

---

That completes 2022. Moving to **2021** now.

---

## 2021 — Part B (Python)

---

### Q8 — Answer any 2 [2 × 3.5 = 7 marks]

#### Q8(a) — Compare List and Tuple

Already covered in 2024 Q1(a). Skip.

#### Q8(b) — File object methods: read(), readline(), readlines(), tell(), seek(), write()

Mostly covered in 2024 Q1(b). One new method:

**`tell()`** Returns the current position of the file pointer as an integer — the number of bytes from the beginning of the file. This is useful when you need to remember a position and return to it later using `seek()`.

python

```python
f = open("file.txt", "r")
f.read(5)       # reads 5 characters
print(f.tell()) # prints 5 — pointer is now at byte 5
f.seek(0)       # move back to beginning
```

`tell()` and `seek()` together enable **random access** in files — you can read a portion, record the position, read more, and jump back to the recorded position at any time.

#### Q8(c) — Generalization vs Encapsulation

Already covered in 2024 Q1(c). Skip.

#### Q8(d) — try, except, finally

Already covered in 2024 Q1(d). Skip.

#### Q8(e) — Read file, tokenize, convert to unique numerical values using dictionary

Already covered in 2024 Q1(e). Skip.

---

### Q9 — Mutable and immutable data types [7 marks]

Already covered in 2024 Q2. Skip.

**OR — Advantages of Python over Java** Already covered in 2023 Q1. Skip.

---

### Q10 — Palindrome and unique words [4+3 = 7 marks]

Already covered in 2024 Q5. Skip.

**OR — Fibonacci generator + trace + memorize + pickling/unpickling** Already covered in 2024 Q5. Skip.

---

### Q11 — Accessibility of class members across classes within and outside package. Multiple inheritance in Python [3+4 = 7 marks]

#### Part 1 — Accessibility of class members

Python does not have a formal package-level access specifier system like Java. Access control is **convention-based**, enforced by naming conventions rather than language-level restrictions.

**Public members:** Any attribute or method without a leading underscore is public. It is accessible from anywhere — within the class, from subclasses, from other classes in the same module, and from external modules.

python

```python
class MyClass:
    def public_method(self):   # accessible everywhere
        pass
```

**Protected members (single underscore `_`):** A single leading underscore is a **convention** signaling that the member is intended for internal use — within the class and its subclasses. Python does **not** enforce this restriction. External code can still access it, but doing so is considered a violation of the class's intended interface.

python

```python
class MyClass:
    def __init__(self):
        self._internal = 42    # accessible but discouraged from outside
```

**Private members (double underscore `__`):** A double leading underscore triggers **name mangling**. Python internally renames `__attr` to `_ClassName__attr`. This makes accidental access from subclasses or external code harder, but does not make it impossible — the mangled name can still be used explicitly.

python

```python
class MyClass:
    def __init__(self):
        self.__secret = 99

obj = MyClass()
obj.__secret              # AttributeError
obj._MyClass__secret      # 99 — accessible via mangled name
```

**Across modules (packages):** When a module is imported, only public names are considered part of its public API. If a module defines `__all__` — a list of strings — then `from module import *` only imports the names listed in `__all__`. Names with a leading underscore are not imported by `import *` even without `__all__`.

There is no true package-level access restriction in Python. A protected or private member of a class in one module can technically be accessed from any other module — the naming convention only signals intent.

---

#### Part 2 — Multiple Inheritance in Python

Multiple inheritance allows a class to inherit from more than one base class simultaneously. The derived class inherits attributes and methods from all base classes.

python

```python
class A:
    def method(self):
        return "A"

class B:
    def method(self):
        return "B"

class C(A, B):    # C inherits from both A and B
    pass

obj = C()
obj.method()      # returns "A" — why?
```

**Method Resolution Order (MRO):** When a method is called, Python needs to determine which class in the inheritance hierarchy to look it up in. This is governed by the MRO, computed using the **C3 linearization algorithm**.

The MRO for a class can be inspected via `ClassName.__mro__` or `ClassName.mro()`.

For `class C(A, B)`, the MRO is: `C → A → B → object`.

Python searches this chain left to right and uses the first class that defines the requested method. This resolves the **diamond problem** — when two base classes share a common ancestor, C3 linearization ensures the ancestor appears only once and in a consistent position.

**Diamond problem example:**

python

```python
class Base:
    def greet(self):
        return "Base"

class Left(Base):
    def greet(self):
        return "Left"

class Right(Base):
    def greet(self):
        return "Right"

class Child(Left, Right):
    pass

Child.__mro__
# (Child, Left, Right, Base, object)

Child().greet()   # "Left" — follows MRO
```

`Base.greet` appears only once in the MRO despite being reachable through both `Left` and `Right`. This is the diamond problem resolved.

**`super()` in multiple inheritance:** `super()` does not simply call the immediate parent — it calls the next class in the MRO. This is important in cooperative multiple inheritance, where each class in the chain calls `super()` to ensure all classes in the MRO get a chance to execute.

---

### Q11 (OR) — Operator Overloading in Python — Complex number class [3+4 = 7 marks]

**Operator overloading** is the mechanism by which a class defines the behavior of Python's built-in operators (`+`, `*`, `>`, etc.) for its instances. This is achieved by implementing special methods called **dunder methods** (double underscore methods), also called **magic methods**.

Python maps each operator to a specific dunder method. When Python evaluates `a + b`, it calls `a.__add__(b)`. If the class defines `__add__`, that definition is used.

**Complex number class:**

python

```python
class Complex:
    def __init__(self, real, imag):
        self.real = real
        self.imag = imag

    def __add__(self, other):       # overloads +
        return Complex(self.real + other.real,
                       self.imag + other.imag)

    def __mul__(self, other):       # overloads *
        # (a+ib)(c+id) = (ac-bd) + (ad+bc)i
        r = self.real * other.real - self.imag * other.imag
        i = self.real * other.imag + self.imag * other.real
        return Complex(r, i)

    def __gt__(self, other):        # overloads >
        return (self.real**2 + self.imag**2) > \
               (other.real**2 + other.imag**2)

    def __str__(self):              # overloads str() / print()
        sign = "+" if self.imag >= 0 else "-"
        return f"{self.real}{sign}{abs(self.imag)}i"
```

**Note on `__lshift__` for `<<`:** The question asks to overload `<<` for display. In Python, `<<` maps to `__lshift__`. However, using `<<` for display is not idiomatic Python — `__str__` is the correct mechanism for string representation, called by `print()` and `str()`. Overloading `<<` for display is a C++ convention (stream insertion). In Python, `__str__` serves this purpose cleanly.

**Key dunder methods for operators:**

|Operator|Dunder method|
|---|---|
|`+`|`__add__`|
|`*`|`__mul__`|
|`>`|`__gt__`|
|`==`|`__eq__`|
|`str()`/`print()`|`__str__`|
|`repr()`|`__repr__`|
|`<`|`__lt__`|

### Q12 — Multiple threads in Python, synchronization, Producer-Consumer problem [7 marks]

#### Thread creation and termination

Already covered in 2023 Q3 (multithreading). The key points: `threading.Thread` with `target` or subclassing, `start()`, `join()`.

A thread ends when its `run()` method returns. There is no forceful termination mechanism in Python's `threading` module — threads are expected to finish naturally. A thread can be made a **daemon thread** by setting `t.daemon = True` before `start()` — daemon threads are killed automatically when the main thread exits, without waiting for them to finish.

---

#### Producer-Consumer problem

**The problem:** A producer thread generates data and places it in a shared buffer. A consumer thread takes data from the buffer and processes it. The constraints are:

- The producer must wait if the buffer is full.
- The consumer must wait if the buffer is empty.
- Access to the buffer must be synchronized to prevent race conditions.

**Solution using `threading.Condition`:**

A `Condition` object wraps a lock and provides `wait()`, `notify()`, and `notify_all()` for thread coordination.

python

```python
import threading
import time

buffer = []
MAX_SIZE = 5
condition = threading.Condition()

class Producer(threading.Thread):
    def run(self):
        for i in range(10):
            with condition:
                while len(buffer) >= MAX_SIZE:
                    condition.wait()       # buffer full, wait
                buffer.append(i)
                print(f"Produced: {i}, Buffer: {buffer}")
                condition.notify_all()    # notify waiting consumers

class Consumer(threading.Thread):
    def run(self):
        for i in range(10):
            with condition:
                while len(buffer) == 0:
                    condition.wait()       # buffer empty, wait
                item = buffer.pop(0)
                print(f"Consumed: {item}, Buffer: {buffer}")
                condition.notify_all()    # notify waiting producers
```

**Why `while` instead of `if` before `wait()`:** After `notify()` wakes a thread, the condition may no longer hold — another thread may have consumed the item first. The `while` loop re-checks the condition after waking, preventing the thread from proceeding incorrectly. This is called **spurious wakeup protection** and is standard practice in concurrent programming.

**Why `notify_all()` over `notify()`:** `notify()` wakes only one waiting thread. If both producers and consumers are waiting (some waiting for space, others for items), `notify()` might wake a producer when a consumer is needed. `notify_all()` wakes all waiting threads, letting each re-check its condition and proceed if appropriate.

---

**OR — Connectionless server (UDP)** Skipping socket programming as instructed.