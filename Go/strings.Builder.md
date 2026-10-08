Go strings are immutable, so a `strings.Builder` isn't a modifiable string. It's a growable **byte buffer** that you append to, and when you're done, `b.String()` gives you the final string.


```go
result := ""
for _, r := range s {
	result += string(r)   // allocates a brand new string every time
}
```

Each `+=` copies the whole string so far, which gets slow for long inputs. The builder appends into one buffer and only produces a string at the end::

go

```go
result := ""
for _, r := range s {
	result.WriteRune(r)   // allocates a brand new string every time
}
```


go

```go
var b strings.Builder     // zero value is ready to use, no make() needed
b.WriteRune('5')          // append a rune (encoded as UTF-8)
b.WriteString("abc")      // append a string
b.WriteByte('x')          // append a byte
s := b.String()           // "5abcx"
```

Note that `var b strings.Builder` works without any setup, unlike maps. Many Go types are designed so their zero value is usable, and `strings.Builder` is one of them.