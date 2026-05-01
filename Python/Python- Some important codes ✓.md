
### Switch Case → `match` statement

```python
command = "quit"

match command:
    case "quit":
        print("quitting")
    case "start":
        print("starting")
    case "pause":
        print("pausing")
    case _:
        print("unknown command")
```
- `case _:` is the default case — same as `default:` in C++
- No `break` needed — cases do **not** fall through
- No fall-through at all, by design

#### Matching multiple values in one case
```python
match command:
    case "quit" | "exit" | "q":
        print("quitting")
    case "start":
        print("starting")
```

#### Matching with a condition — **guards**

```python
match x:
    case n if n < 0:
        print("negative")
    case n if n == 0:
        print("zero")
    case n:
        print("positive")
```

#### Matching tuples / structure

`match` in Python is actually a **structural pattern matcher** — it can destructure objects:
```python
point = (1, 0)

match point:
    case (0, 0):
        print("origin")
    case (x, 0):
        print(f"on x-axis at {x}")
    case (0, y):
        print(f"on y-axis at {y}")
    case (x, y):
        print(f"at {x}, {y}")
```


#### Iterating over a list with index — `enumerate()`
```python
fruits = ["apple", "banana", "cherry"]

for i, fruit in enumerate(fruits):
    print(i, fruit)
# 0 apple
# 1 banana
# 2 cherry
```

`enumerate()` wraps an iterable and yields `(index, value)` pairs. The `i, fruit` syntax is **tuple unpacking** — Python assigns the pair to two variables simultaneously.

#### Iterating over two sequences — `zip()`

```python
names = ["Alice", "Bob", "Charlie"]
scores = [95, 87, 92]

for name, score in zip(names, scores):
    print(f"{name}: {score}")
# Alice: 95
# Bob: 87
# Charlie: 92
```
