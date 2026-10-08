
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


