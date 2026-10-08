A `rune` is Go's name for a single Unicode character (a "code point"). It's an alias for `int32`, so it's just a number: `'A'` is 65, `'é'` is 233, `'9'` is 57. The word is odd, but it exists because "char" would be misleading in Go: a `string` is a sequence of **bytes**, and one character can take **several bytes** in UTF-8.

When you `range` over a string, Go decodes UTF-8 for you and hands you one rune at a time:

go

```go
for i, r := range "aé9" {
	fmt.Println(i, r, string(r))
}
// 0 97  a
// 1 233 é     <- index jumps by 2 because é is 2 bytes
// 3 57  9
```

### Rune literals

Single quotes make a rune, double quotes make a string:

go

```go
'9'   // rune (number 57)
"9"   // string
```

