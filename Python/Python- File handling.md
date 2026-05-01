
## File Handling in Python

---

### 1) Normal File IO

#### Opening a File

python

```python
f = open("file.txt", "r")
```

`open(path, mode)` returns a **file object**. The mode string controls what you can do:

|Mode|Meaning|
|---|---|
|`"r"`|read (default) — file must exist|
|`"w"`|write — creates file if not exist, **truncates** if exists|
|`"a"`|append — creates if not exist, writes at end if exists|
|`"x"`|exclusive create — fails if file already exists|
|`"r+"`|read and write — file must exist|
|`"w+"`|read and write — truncates|
|`"b"`|binary mode — combine with others: `"rb"`, `"wb"`|
|`"t"`|text mode (default) — combine with others: `"rt"`, `"wt"`|

Text mode handles newline translation and encoding. Binary mode reads/writes raw bytes.

---

#### The Right Way — `with` Statement

Always use `with` for file IO. It guarantees the file is closed when the block exits — even if an exception occurs:

python

```python
with open("file.txt", "r") as f:
    content = f.read()
# file is closed here automatically
```

Without `with`, you must call `f.close()` manually. If an exception occurs before `close()`, the file stays open — resource leak.

---

#### Reading

python

```python
with open("file.txt", "r") as f:

    content = f.read()          # reads entire file as one string

    f.seek(0)                   # move cursor back to start

    line = f.readline()         # reads one line including \n

    f.seek(0)

    lines = f.readlines()       # reads all lines into a list of strings
```

`f.read()`, `f.readline()`, `f.readlines()` all move the file cursor forward. Once you've read to the end, reading again returns empty string. `f.seek(0)` resets the cursor to the beginning.

#### Iterating line by line — most memory efficient

python

```python
with open("file.txt", "r") as f:
    for line in f:              # file object is iterable — yields one line at a time
        print(line.strip())     # strip() removes the trailing \n
```

This is lazy — it does not load the entire file into memory. For large files, this is the correct approach.

---

#### Writing

python

```python
with open("output.txt", "w") as f:
    f.write("hello\n")          # write a string — \n not added automatically
    f.write("world\n")

with open("output.txt", "w") as f:
    f.writelines(["line1\n", "line2\n", "line3\n"])   # write a list of strings
```

`writelines` does not add newlines between items — you include them yourself.

---

#### Appending

python

```python
with open("log.txt", "a") as f:
    f.write("new log entry\n")   # adds to end, does not truncate
```

---

#### Binary IO

python

```python
with open("image.png", "rb") as f:
    data = f.read()             # bytes object, not str

with open("copy.png", "wb") as f:
    f.write(data)
```

In binary mode, `read()` returns `bytes`, `write()` expects `bytes`.

---

#### File Cursor Methods

python

```python
f.tell()         # returns current cursor position as byte offset from start
f.seek(offset)   # move cursor to byte offset from start
f.seek(0, 0)     # seek to offset 0 from start  (same as seek(0))
f.seek(0, 1)     # seek to offset 0 from current position
f.seek(0, 2)     # seek to offset 0 from end — i.e. move to end
```

---

#### Encoding

Text mode uses the system default encoding (usually UTF-8 on modern systems). Always specify explicitly for portability:

python

```python
with open("file.txt", "r", encoding="utf-8") as f:
    content = f.read()

with open("file.txt", "w", encoding="utf-8") as f:
    f.write("some text")
```

---

#### Opening Multiple Files at Once

python

```python
with open("input.txt", "r") as fin, open("output.txt", "w") as fout:
    for line in fin:
        fout.write(line.upper())
```

Multiple context managers in one `with` statement separated by commas.

---

### 2) Writing Class Objects to File

A plain `open()` and `write()` can only write strings or bytes. To write objects, you need **serialization** — converting an object to a storable format. Python has two main approaches: `pickle` and `json`.

---

#### `pickle` — Binary Serialization

`pickle` converts any Python object to bytes and back. It preserves the full object — type, attributes, nested objects, everything.

python

```python
import pickle

class Student:
    def __init__(self, name, age, grades):
        self.name = name
        self.age = age
        self.grades = grades

    def __repr__(self):
        return f"Student({self.name}, {self.age}, {self.grades})"
```

**Writing an object:**

python

```python
s = Student("Romit", 20, [90, 85, 92])

with open("student.pkl", "wb") as f:   # binary write mode
    pickle.dump(s, f)
```

`pickle.dump(obj, file)` — serializes `obj` and writes bytes to `file`.

**Reading it back:**

python

```python
with open("student.pkl", "rb") as f:   # binary read mode
    s = pickle.load(f)

print(s)        # Student(Romit, 20, [90, 85, 92])
print(s.name)   # Romit
```

`pickle.load(file)` — reads bytes and reconstructs the object. The class definition must be available (imported) when loading — pickle stores the object's data, not the class code itself.

**Multiple objects:**

python

```python
students = [
    Student("Romit", 20, [90, 85, 92]),
    Student("Alice", 21, [88, 79, 95]),
    Student("Bob",   19, [70, 80, 75]),
]

# write all
with open("students.pkl", "wb") as f:
    pickle.dump(students, f)     # dump the entire list as one object

# read all
with open("students.pkl", "rb") as f:
    students = pickle.load(f)
```

Or dump objects one by one and load them one by one:

python

```python
with open("students.pkl", "wb") as f:
    for s in students:
        pickle.dump(s, f)

with open("students.pkl", "rb") as f:
    while True:
        try:
            s = pickle.load(f)
            print(s)
        except EOFError:
            break              # no more objects in file
```

**`pickle.dumps` / `pickle.loads`** — same but to/from bytes in memory instead of a file:

python

```python
data = pickle.dumps(s)    # returns bytes
s2   = pickle.loads(data) # reconstructs from bytes
```

**What pickle can handle:**

- All built-in types — int, str, list, dict, tuple, set
- Class instances
- Nested objects
- Functions and lambdas (with limitations)
- Most standard library objects

**What pickle cannot handle:**

- File handles
- Database connections
- Some C extension objects

**Security warning:** Never unpickle data from an untrusted source. `pickle.load` can execute arbitrary code during deserialization — it is a known attack vector.

---

#### Controlling Pickle Behavior — `__getstate__` and `__setstate__`

By default, pickle serializes `obj.__dict__`. You can override this:

python

```python
class Student:
    def __init__(self, name, age, grades):
        self.name = name
        self.age = age
        self.grades = grades
        self._cache = {}       # we don't want to pickle this

    def __getstate__(self):
        state = self.__dict__.copy()
        del state['_cache']    # exclude _cache from serialization
        return state

    def __setstate__(self, state):
        self.__dict__.update(state)
        self._cache = {}       # reinitialize _cache on load
```

`__getstate__` returns what gets pickled. `__setstate__` receives it on load and restores the object.

---

#### `json` — Text Serialization

`json` serializes to a human-readable text format. The tradeoff: it only handles a limited set of types natively.

**Types json handles natively:**

|Python|JSON|
|---|---|
|`dict`|object `{}`|
|`list`, `tuple`|array `[]`|
|`str`|string|
|`int`, `float`|number|
|`True`, `False`|`true`, `false`|
|`None`|`null`|

Classes are **not** natively supported. You must convert them to dicts first.

**Simple approach — convert manually:**

python

```python
import json

class Student:
    def __init__(self, name, age, grades):
        self.name = name
        self.age = age
        self.grades = grades

s = Student("Romit", 20, [90, 85, 92])

# writing
with open("student.json", "w") as f:
    json.dump(s.__dict__, f, indent=4)   # __dict__ is already a plain dict
```

Output in `student.json`:

json

```json
{
    "name": "Romit",
    "age": 20,
    "grades": [90, 85, 92]
}
```

Reading back:

python

```python
with open("student.json", "r") as f:
    data = json.load(f)          # gives you a dict, not a Student

s = Student(**data)              # reconstruct using unpacking
```

**Custom encoder — cleaner approach:**

python

```python
class StudentEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Student):
            return {
                "__type__": "Student",    # tag to identify type on load
                "name": obj.name,
                "age": obj.age,
                "grades": obj.grades
            }
        return super().default(obj)
```

python

```python
with open("student.json", "w") as f:
    json.dump(s, f, cls=StudentEncoder, indent=4)
```

**Custom decoder:**

python

```python
def student_decoder(data):
    if data.get("__type__") == "Student":
        return Student(data["name"], data["age"], data["grades"])
    return data

with open("student.json", "r") as f:
    s = json.load(f, object_hook=student_decoder)

print(type(s))    # <class 'Student'>
print(s.name)     # Romit
```

`object_hook` is called for every JSON object decoded — if it recognises the `__type__` tag, it reconstructs the class instance.

**`json.dumps` / `json.loads`** — same but to/from string in memory:

python

```python
s_json = json.dumps(s.__dict__)   # string
data   = json.loads(s_json)       # dict
```

---