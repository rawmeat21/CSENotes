
### Grouped declaration

```go
var (
	ErrNotFound = errors.New("not found")
	ErrAmbiguous = errors.New("cannot identify accounts uniquely with this string")
	ErrDuplicateIdentifier = errors.New("identifier already used by another account for this service")
	ErrDuplicateField = errors.New("duplicate field name in account")
	ErrDuplicateID = errors.New("account ID already exists")
	ErrInvalidAccount = errors.New("invalid account")
)
```

Go lets you group `var`, `const`, `type`, and `import` declarations in parentheses when they're related. You've already seen it with imports:

go

```go
import (
	"errors"
	"fmt"
)
```

Grouping saves repeating the keyword and signals "these belong together". For a set of error values that's a common convention.


### Structs cannot be nil

```go
type Field struct {
	Name  string
	Value string
}
```

```go
func find() Field {
	return nil        // compile error: cannot use nil as Field value
}
```

In Go `nil` only exists for types that can be "nothing": pointers, slices, maps, channels, functions, interfaces. A struct is always a real value with all its fields present, so there is no such thing as a nil `Field` object.

`Field{}` is the **zero value** of the struct: every field set to its own zero value (`""` for strings, `0` for ints, `false` for bools, `nil` for pointers/slices/maps inside it).