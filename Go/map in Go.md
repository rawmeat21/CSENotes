### The syntax

go

```go
map[KeyType]ValueType
```

Read it as "a map from KeyType to ValueType". Your struct has three:

go

```go
accounts  map[string]*Account // string key -> pointer to an Account
byService map[string][]string // string key -> slice of strings
byIdent   map[string][]string // string key -> slice of strings
```

So `byService["github"]` would give you a `[]string` of account IDs, and `accounts["id123"]` gives you a `*Account` (a pointer, so the Account itself isn't copied when you pass it around).

### Declaring vs. creating

Declaring a map doesn't make it usable:

go

```go
var m map[string]int   // nil map: reading works, writing PANICS
m["a"] = 1             // panic: assignment to entry in nil map
```

You must create it with `make` or a literal:

go

```go
m := make(map[string]int)          // empty, ready to use
m := map[string]int{"a": 1, "b": 2} // literal with initial values
```

That matters for your struct. The fields start as nil, so you need a constructor that initializes them:

go

```go
func NewAccountDatabase() *AccountDatabase {
	return &AccountDatabase{
		accounts:  make(map[string]*Account),
		byService: make(map[string][]string),
		byIdent:   make(map[string][]string),
	}
}
```

(This is a common Go pattern: there are no OOP-style constructors, just a plain function named `NewXxx` that returns a ready-to-use value.)

### Using a map

go

```go
m["a"] = 1            // set
v := m["a"]           // get (returns zero value, 0, if missing)
v, ok := m["z"]       // "comma ok": ok is false if key is absent
delete(m, "a")        // remove
len(m)                // number of entries
```

The comma-ok form is how you tell "key missing" apart from "key present with zero value".

### Maps with slice values

`byService` index relies on a handy behavior: a missing key returns a nil slice, and `append` works fine on a nil slice, so you don't need to check first:

go

```go
db.byService["github"] = append(db.byService["github"], "id123")
```

If the key doesn't exist, `db.byService["github"]` is a nil `[]string`, `append` creates a new slice, and you store it back under that key.

### Iterating

go

```go
for key, value := range m {
	fmt.Println(key, value)
}
```

The order changes between runs. If you need sorted output, collect the keys into a slice and sort them:

go

```go
keys := make([]string, 0, len(m))
for k := range m {
	keys = append(keys, k)
}
sort.Strings(keys)
```

### One caveat

Maps are reference-like (like slices, a map variable is a small handle to the real data), so copying a map variable gives you a second handle to the same data, and maps are **not safe for concurrent writes**. If you ever use goroutines, guard access with a `sync.Mutex`.