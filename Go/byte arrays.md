`[]byte` is a **slice of bytes**: a resizable view over a sequence of `byte` values. In Go, `byte` is just an alias for `uint8`, so each element is an unsigned 8-bit integer (0 to 255).

### What it stores

Raw binary data, or anything you want to treat as a sequence of bytes: file contents, network packets, encrypted data, hashes, or text in its encoded form.

go

```go
b := []byte{72, 101, 108, 108, 111}
fmt.Println(b)          // [72 101 108 108 111]
fmt.Println(string(b))  // Hello
```

### What a slice actually is

A slice isn't the data itself. It's a small struct (a "slice header") with three fields pointing at an underlying array:

```
slice header                 underlying array
+-----------+
| ptr       |--------------> [72][101][108][108][111][ ][ ]
| len = 5   |                  0    1    2    3    4   5  6
| cap = 7   |
+-----------+
```

- `ptr`: address of the first element
- `len`: how many elements you can currently see
- `cap`: how many fit in the underlying array before Go must allocate a bigger one

So copying a slice variable copies only this 3-field header, not the bytes. Both copies point at the same data:

go

```go
a := []byte("hi")
b := a
b[0] = 'H'
fmt.Println(string(a)) // Hi  (a changed too)
```

To get an independent copy, use `copy`:

go

```go
c := make([]byte, len(a))
copy(c, a)
```

### Strings vs []byte

A Go `string` is an **immutable** sequence of bytes (usually UTF-8). A `[]byte` is **mutable**. Converting between them copies the data:

go

```go
s := "hello"
b := []byte(s)   // copy into a mutable slice
b[0] = 'J'
s2 := string(b)  // copy back: "Jello"
```

One thing to watch: `[]byte` holds bytes, not characters. A non-ASCII character like `é` takes 2 bytes in UTF-8:

go

```go
fmt.Println(len("é"))          // 2
fmt.Println([]byte("é"))       // [195 169]
```

### Common ways to create one

go

```go
var b []byte                 // nil slice, len 0
b = make([]byte, 16)         // 16 zero bytes
b = make([]byte, 0, 64)      // len 0, capacity 64 (good before appending)
b = []byte("text")           // from a string
b = append(b, 'x', 'y')      // grow it
```