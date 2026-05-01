
# Strings- Just the important stuff


## 1. Slicing

Syntax: `s[start : stop : step]`

- `start` — index to begin at (inclusive)
- `stop` — index to end at (**exclusive**)
- `step` — how many indices to advance each time (default 1)

## 2. IMP methods
```python
s.title()       # "Hello World" — first char of each word uppercased
s.find("world")      # 6  — returns starting index, or -1 if not found
s.find("xyz")        # -1
s.index("world")     # 6  — same as find, but raises ValueError if not found
s.count("l")         # 3  — how many times "l" appears
s.startswith("hel")  # True
s.endswith("rld")    # True
```
`find()` vs `index()`: When string is not found, find() returns -1 while index gives a ValueError

Strip whitespace:
```python
s = "   hello   "

s.strip()     # "hello"   — removes leading and trailing whitespace
s.lstrip()    # "hello   " — removes only leading
s.rstrip()    # "   hello" — removes only trailing
```
#### Replace

```python
"hello world".replace("world", "python")   # "hello python"
"aabbaa".replace("a", "x")                # "xxbbxx" — replaces all by default
"aabbaa".replace("a", "x", 1)             # "xabbaa" — third arg limits replacements
```
#### Split and Join

**Split** breaks a string into a list of substrings:

```python
"a,b,c".split(",")       # ['a', 'b', 'c']
"hello world".split()    # ['hello', 'world'] — splits on any whitespace if no arg
"a,,b".split(",")        # ['a', '', 'b'] — empty string between consecutive delimiters
```

**Join** is the inverse — it takes a list of strings and joins them with a separator:
```python
",".join(['a', 'b', 'c'])     # "a,b,c"
" ".join(['hello', 'world'])  # "hello world"
```
The separator string is the object you call `.join()` on. The argument is the list. 

#### Check type of content

```python
"123".isdigit()    # True  — all characters are digits
"abc".isalpha()    # True  — all characters are alphabetic
"abc123".isalnum() # True  — all characters are alphanumeric
"   ".isspace()    # True  — all characters are whitespace
"ABC".isupper()    # True
"abc".islower()    # True
```

### `ord()` and `chr()`

These convert between characters and their Unicode code points:

```python
ord('A')    # 65
ord('a')    # 97
chr(65)     # 'A'
```

### String Interning (under the hood)

Python **interns** (caches and reuses) certain string objects — typically short strings that look like identifiers. This means two variables holding `"hello"` may actually point to the same object in memory.

python

```python
a = "hello"
b = "hello"
a is b    # True — same object (interned)

a = "hello world"
b = "hello world"
a is b    # may be False — longer strings may not be interned
```

`is` checks **identity** (same object in memory). `==` checks **equality** (same value). For strings, always use `==` for comparison. `is` is an implementation detail here.


