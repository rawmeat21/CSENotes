
## Iterator and itertools

```python
import itertools

class numiterator:
    def __init__(self, limit):
        self.curr = 1
        self.limit = limit

    def __iter__(self):
        return self

    def __next__(self):
        if self.curr > self.limit:
            raise StopIteration
        v = self.curr
        self.curr += 1
        return v

numgen = numiterator(20)
nums = list(numgen)  # get list from numgen iterator
print(nums)

# masks
oddmask = [n % 2 != 0 for n in nums]
print(oddmask)

evenmask = [n % 2 == 0 for n in nums]
print(evenmask)

odds = list(itertools.compress(nums, oddmask))
evens = list(itertools.compress(nums, evenmask))
print(odds)
print(evens)
```

## Flatten a json

```python
import json

d = {}

def flatten(x, path):
    if not isinstance(x, dict):
        d[".".join(path)] = x
        return

    for k, v in x.items():
        path.append(str(k))
        flatten(v, path)
        path.pop()

test_data = {}
with open("myfile.json", "r") as f:
    test_data = json.load(f)

flatten(test_data, [])
print(d)
```

