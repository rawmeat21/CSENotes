## Regex in Python

---

### What is Regex

A regular expression is a **pattern** — a mini-language for describing what a string should look like. You write a pattern, and the regex engine checks strings against it — finding matches, extracting parts, replacing text.

Python's regex module is `re`:

python

```python
import re
```

---

### How the Engine Works

The engine scans the string left to right, trying to match the pattern starting at each position. When it finds a match it either returns it or continues looking for more, depending on which function you called.

This mental model matters — the engine is positional and greedy by default (covered later).

---

### The `re` Functions

These are what you actually call:

python

```python
re.match(pattern, string)      # match only at the START of the string
re.search(pattern, string)     # match ANYWHERE in the string, returns first match
re.findall(pattern, string)    # returns list of all matches
re.finditer(pattern, string)   # returns iterator of match objects
re.sub(pattern, repl, string)  # replace matches with repl
re.split(pattern, string)      # split string by pattern
re.fullmatch(pattern, string)  # match must cover the ENTIRE string
```

python

```python
re.match(r"\d+", "123abc")     # matches — starts at position 0
re.match(r"\d+", "abc123")     # None — no match at position 0
re.search(r"\d+", "abc123")    # matches — finds 123 anywhere
re.fullmatch(r"\d+", "123")    # matches — entire string is digits
re.fullmatch(r"\d+", "123abc") # None — not entire string
```

All functions return a **match object** on success, `None` on failure (except `findall` and `split` which return lists).

#### Match object

python

```python
m = re.search(r"\d+", "abc123def")

m.group()     # "123"  — the matched string
m.start()     # 3      — start index
m.end()       # 6      — end index (exclusive)
m.span()      # (3, 6) — tuple of (start, end)
```

Always check for `None` before using a match object:

python

```python
m = re.search(pattern, string)
if m:
    print(m.group())
```

---

### Raw Strings

Always write patterns as **raw strings** — prefix with `r`:

python

```python
r"\d+"    # raw string — backslash is literal, not an escape
"\d+"     # regular string — \d is not a standard escape, works here but fragile
```

In a regular string, `\n` is a newline, `\t` is a tab, etc. In a raw string, backslashes are always literal. Since regex uses `\d`, `\w`, `\s` etc., raw strings prevent Python from interpreting the backslash before the regex engine sees it.

Rule: **always use `r"..."` for regex patterns**.

---

### Pattern Syntax — Building Blocks

#### Literal characters

python

```python
re.search(r"cat", "the cat sat")   # matches "cat"
```

Most characters match themselves literally.

---

#### `.` — Any character except newline

python

```python
re.search(r"c.t", "cat")   # matches "cat"
re.search(r"c.t", "cot")   # matches "cot"
re.search(r"c.t", "ct")    # None — . requires exactly one char
```

`.` matches any single character except `\n`. To include newlines, use the `re.DOTALL` flag.

---

#### Character Classes `[...]`

Matches **one character** that is any of the listed characters:

python

```python
r"[aeiou]"      # one vowel
r"[abc]"        # one of a, b, or c
r"[0-9]"        # one digit — range syntax
r"[a-z]"        # one lowercase letter
r"[A-Z]"        # one uppercase letter
r"[a-zA-Z]"     # one letter, any case
r"[a-zA-Z0-9]"  # one alphanumeric character
```

**Negated class** — `^` inside `[...]` means NOT:

python

```python
r"[^aeiou]"    # one character that is NOT a vowel
r"[^0-9]"      # one character that is NOT a digit
```

**Special characters lose most meaning inside `[...]`**:

python

```python
r"[.]"    # literal dot — not "any character"
r"[-]"    # literal hyphen — put it first or last to avoid range interpretation
r"[^]"    # not valid — ^ only negates at the start
```

---

#### Shorthand Character Classes

These are abbreviations for common character classes:

|Pattern|Meaning|Equivalent|
|---|---|---|
|`\d`|digit|`[0-9]`|
|`\D`|non-digit|`[^0-9]`|
|`\w`|word character|`[a-zA-Z0-9_]`|
|`\W`|non-word character|`[^a-zA-Z0-9_]`|
|`\s`|whitespace|`[ \t\n\r\f\v]`|
|`\S`|non-whitespace|`[^ \t\n\r\f\v]`|

python

```python
re.search(r"\d", "abc3def")    # matches "3"
re.search(r"\w+", "hello!")    # matches "hello"
re.search(r"\s", "a b")        # matches the space
```

---

#### Anchors — Position, Not Characters

Anchors match a **position** in the string, not a character. They consume no characters.

|Anchor|Matches position|
|---|---|
|`^`|start of string (or start of line with `re.MULTILINE`)|
|`$`|end of string (or end of line with `re.MULTILINE`)|
|`\b`|word boundary — between `\w` and `\W`|
|`\B`|non-word boundary|
|`\A`|absolute start of string (unaffected by `re.MULTILINE`)|
|`\Z`|absolute end of string|

python

```python
re.search(r"^\d+", "123abc")    # matches "123" — starts at beginning
re.search(r"^\d+", "abc123")    # None — doesn't start with digits
re.search(r"\d+$", "abc123")    # matches "123" — ends with digits
re.search(r"\bcat\b", "the cat sat")   # matches "cat" — whole word
re.search(r"\bcat\b", "concatenate")   # None — cat not at word boundary
```

---

#### Quantifiers — How Many Times

Quantifiers apply to the **preceding element**:

|Quantifier|Meaning|
|---|---|
|`*`|0 or more|
|`+`|1 or more|
|`?`|0 or 1 (optional)|
|`{n}`|exactly n times|
|`{n,}`|n or more times|
|`{n,m}`|between n and m times (inclusive)|

python

```python
re.search(r"\d*", "abc")       # matches "" — 0 digits is valid
re.search(r"\d+", "abc")       # None — needs at least 1 digit
re.search(r"\d+", "abc123")    # matches "123"
re.search(r"colou?r", "color")    # matches — u is optional
re.search(r"colou?r", "colour")   # matches
re.search(r"\d{3}", "12345")   # matches "123" — exactly 3
re.search(r"\d{3,5}", "12345") # matches "12345"
re.search(r"\d{3,5}", "12")    # None — less than 3
```

---

#### Greedy vs Lazy

By default quantifiers are **greedy** — they match as much as possible:

python

```python
re.search(r"<.+>", "<a>text<b>")
# matches "<a>text<b>" — grabbed everything between first < and last >
```

Add `?` after a quantifier to make it **lazy** — match as little as possible:

python

```python
re.search(r"<.+?>", "<a>text<b>")
# matches "<a>" — stopped at first >
```

|Greedy|Lazy|
|---|---|
|`*`|`*?`|
|`+`|`+?`|
|`?`|`??`|
|`{n,m}`|`{n,m}?`|

Greedy: expand as far as possible, then backtrack if needed. Lazy: expand as little as possible, then expand if needed.

---

#### Alternation — `|`

Matches either the left or right expression:

python

```python
re.search(r"cat|dog", "I have a dog")   # matches "dog"
re.search(r"cat|dog", "I have a cat")   # matches "cat"
re.search(r"cat|dog", "no pets")        # None
```

`|` has the lowest precedence — `cat|dog` means `(cat)|(dog)`, not `ca(t|d)og`. Use groups to control scope.

---

#### Groups `(...)`

Parentheses group part of a pattern. Two uses:

**1. Apply quantifiers to a group:**

python

```python
re.search(r"(ab)+", "ababab")   # matches "ababab" — (ab) repeated
```

**2. Capture — extract the matched part:**

python

```python
m = re.search(r"(\d{4})-(\d{2})-(\d{2})", "date: 2024-01-15")

m.group(0)   # "2024-01-15" — entire match
m.group(1)   # "2024"       — first capture group
m.group(2)   # "01"         — second capture group
m.group(3)   # "15"         — third capture group
```

Groups are numbered left to right by their opening `(`. `group(0)` is always the entire match.

**Named groups** — `(?P<name>...)`:

python

```python
m = re.search(r"(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})", "2024-01-15")

m.group("year")    # "2024"
m.group("month")   # "01"
m.group("day")     # "15"
```

Named groups make patterns self-documenting.

---

#### Non-Capturing Groups `(?:...)`

Groups the pattern but does not capture — no entry in `m.group()`:

python

```python
re.search(r"(?:ab)+cd", "ababcd")   # matches "ababcd", no capture group
```

Use when you need grouping for quantifiers or alternation but don't need to extract the content.

---

#### `findall` with Groups

When the pattern has groups, `findall` returns the group contents instead of the full match:

python

```python
re.findall(r"\d+", "a1b22c333")
# ['1', '22', '333']  — no groups, returns full matches

re.findall(r"(\d+)", "a1b22c333")
# ['1', '22', '333']  — one group, returns list of group 1 contents

re.findall(r"(\w+)=(\w+)", "x=1 y=2 z=3")
# [('x','1'), ('y','2'), ('z','3')]  — two groups, returns list of tuples
```

---

#### Lookahead and Lookbehind

These match a position based on what is **around** it, without consuming characters.

**Positive lookahead** `(?=...)` — match if followed by:

python

```python
re.findall(r"\d+(?= dollars)", "100 dollars and 50 euros")
# ['100']  — only digits followed by " dollars"
```

**Negative lookahead** `(?!...)` — match if NOT followed by:

python

```python
re.findall(r"\d+(?! dollars)", "100 dollars and 50 euros")
# ['50']
```

**Positive lookbehind** `(?<=...)` — match if preceded by:

python

```python
re.findall(r"(?<=\$)\d+", "price $100 and $200")
# ['100', '200']  — digits preceded by $
```

**Negative lookbehind** `(?<!...)` — match if NOT preceded by:

python

```python
re.findall(r"(?<!\$)\d+", "price $100 and 200")
# ['200']
```

Lookarounds assert context without including it in the match. The string consumed is only the part outside the lookaround.

---

#### `re.sub` — Substitution

python

```python
re.sub(pattern, replacement, string, count=0)
```

python

```python
re.sub(r"\d+", "NUM", "abc123def456")
# "abcNUMdefNUM"

re.sub(r"\s+", " ", "too   many    spaces")
# "too many spaces"
```

`count` limits how many substitutions:

python

```python
re.sub(r"\d+", "NUM", "1 2 3", count=2)
# "NUM NUM 3"
```

**Using group references in replacement:**

`\1`, `\2` in the replacement string refer to captured groups:

python

```python
re.sub(r"(\w+) (\w+)", r"\2 \1", "hello world")
# "world hello" — swapped
```

**Using a function as replacement:**

python

```python
def double_digit(m):
    return str(int(m.group()) * 2)

re.sub(r"\d+", double_digit, "a1b2c3")
# "a2b4c6"
```

When replacement is a function, it receives the match object and must return a string.

---

#### `re.split`

python

```python
re.split(r"\s+", "one  two   three")
# ['one', 'two', 'three']

re.split(r"[,;]\s*", "a, b;c,  d")
# ['a', 'b', 'c', 'd']
```

If the pattern has a group, the matched separators are included in the result:

python

```python
re.split(r"(\s+)", "one two three")
# ['one', ' ', 'two', ' ', 'three']
```

---

#### Flags

Flags modify matching behavior. Pass as the `flags` argument:

python

```python
re.search(pattern, string, flags=re.IGNORECASE)
```

|Flag|Short|Effect|
|---|---|---|
|`re.IGNORECASE`|`re.I`|case-insensitive matching|
|`re.MULTILINE`|`re.M`|`^` and `$` match start/end of each line|
|`re.DOTALL`|`re.S`|`.` matches newline too|
|`re.VERBOSE`|`re.X`|allows whitespace and comments in pattern|

python

```python
re.search(r"hello", "HELLO", re.I)    # matches

re.findall(r"^\d+", "123\n456", re.M) # ['123', '456'] — ^ matches each line start
```

`re.VERBOSE` lets you write readable multi-line patterns:

python

```python
pattern = re.compile(r"""
    (\d{4})   # year
    -
    (\d{2})   # month
    -
    (\d{2})   # day
""", re.VERBOSE)
```

Whitespace in the pattern is ignored, `#` starts a comment. Only the actual regex tokens matter.

---

#### Compiled Patterns

If you use a pattern repeatedly, compile it once:

python

```python
pattern = re.compile(r"\d+")

pattern.search("abc123")
pattern.findall("1a2b3c")
pattern.sub("NUM", "a1b2")
```

`re.compile` returns a pattern object with the same methods as the `re` module functions, but without the pattern argument. Saves recompilation overhead.

---

### Building Patterns — The Thinking Process

When writing a regex from scratch:

**1. Identify what you are matching** — email, date, number, word, etc.

**2. Break it into parts** — what are the components and their order

**3. Define each part** — what characters, how many, optional or required

**4. Assemble and handle edge cases**

Example — match an email address:

```
user@domain.com

parts:
  user     → one or more word chars or dots or hyphens: [\w.\-]+
  @        → literal @
  domain   → one or more word chars or hyphens: [\w\-]+
  .        → literal dot: \.
  com/org  → two or more letters: [a-zA-Z]{2,}
```

python

```python
pattern = re.compile(r"[\w.\-]+@[\w\-]+\.[a-zA-Z]{2,}")
```

---

### Quick Reference

```
.          any char except \n
\d \D      digit / non-digit
\w \W      word char / non-word char
\s \S      whitespace / non-whitespace

^  $       start / end of string
\b \B      word boundary / non-boundary
\A \Z      absolute start / end

[abc]      character class
[^abc]     negated class
[a-z]      range

*  +  ?         0+, 1+, 0 or 1  (greedy)
*? +? ??        lazy versions
{n} {n,} {n,m} exact, min, range

(...)      capturing group
(?:...)    non-capturing group
(?P<x>...) named group
|          alternation

(?=...)    positive lookahead
(?!...)    negative lookahead
(?<=...)   positive lookbehind
(?<!...)   negative lookbehind
```

---

What's next?