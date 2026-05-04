```python
class myList:
    def __init__(self, n):
        self.arr = [0] * n

    def add(self, x):
        self.arr.append(x)

    def __iter__(self):
        return myList.myIterator(self)

    class myIterator:
        def __init__(self, li):
            self.li = li
            self.i = 0

        def __iter__(self):
            return self

        def __next__(self):
            if self.i == len(self.li.arr):
                raise StopIteration
            x = self.li.arr[self.i]
            self.i += 1
            return x

lis = myList(5)
lis.add(3)
lis.add(1)
lis.add(2)

for x in lis:
    print(x)
```

