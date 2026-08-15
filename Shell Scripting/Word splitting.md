
Word splitting occurs **only** when an **unquoted expansion** (Parameter, Command, or Arithmetic) is evaluated in a context expecting a list of words or arguments.

Only three expansion constructs trigger word splitting when left unquoted:

- **Parameter Expansion:** `$var`, `${var}`
    
- **Command Substitution:** `$(cmd)`, `` `cmd` ``
    
- **Arithmetic Expansion:** `$((expr))`
    

## 1. Contexts Where Word Splitting DOES Happen

### Command Position Arguments

Unquoted expansions passed to utilities, functions, or shell builtins are split by `$IFS`.

Bash

```bash
var="one two three"
printf "[%s]\n" $var
# Output:
# [one]
# [two]
# [three]
```

### Loop Iteration Lists (`for ... in`)

The word list following `in` is split to create loop iteration elements.

Bash

```bash
for item in $(echo "a b c"); do
    echo "Item: $item"
done
```

### Compound Array Initializers

Values placed inside `arr=( ... )` undergo word splitting to populate array elements.

Bash

```bash
str="apple banana cherry"
arr=( $str )
# arr[0]="apple", arr[1]="banana", arr[2]="cherry"
```

### Unquoted Array Expansions

- **`"${arr[@]}"` (Quoted):** NO splitting. Preserves exact array elements.
    
- **`${arr[@]}` (Unquoted):** Expands each element, then **splits each element string** on `$IFS`.
    
- **`${arr[*]}` (Unquoted):** Concatenates all elements using the first character of `$IFS`, then **splits the resulting string** on `$IFS`.
    

Bash

```bash
arr=("hello world" "foo bar")
printf "[%s]\n" ${arr[@]}
# Output: [hello], [world], [foo], [bar]
```

### Unquoted Redirection Targets

Target file paths undergo word splitting. If the expansion resolves to multiple words, Bash raises an error.

Bash

```
file="my log.txt"
echo "data" > $file
# Result: bash: $file: ambiguous redirect
```

### Argument Lists for Variable-Declaring Builtins

Passing unquoted variable values as positional arguments to `export`, `declare`, `local`, or `readonly` forces word splitting across arguments.

Bash

```
opts="FOO=1 BAR=2"
export $opts
# Executes: export FOO=1 BAR=2
```

### Positional Arguments in Single Bracket Tests (`[ ... ]`)

The legacy `test` / `[` command evaluates arguments via standard utility rules, subjecting unquoted expansions to word splitting.

Bash

```
var="a b"
[ $var = "a b" ]
# Evaluates to: [ a b = "a b" ] -> Syntax error: too many arguments
```

## 2. Contexts Where Word Splitting DOES NOT Happen

### Double-Quoted Expansions

Enclosing any expansion in `""` suppresses word splitting completely across all contexts.

Bash

```
var="one two three"
printf "[%s]\n" "$var"
# Output: [one two three]
```

### Scalar Variable Assignments (`var=...`)

The right-hand side of a scalar assignment is evaluated strictly as a literal string context.

Bash

```
# NO splitting occurs during assignment
data=$(cat file.txt)
backup=$data
```

### Assignment Arguments in `export` / `declare` / `local`

The right-hand side of an explicit `key=value` assignment inside declaration builtins is protected from word splitting in modern Bash.

Bash

```
export PATH_VAR=$(cat path.txt)  # Safe: Treated as scalar assignment
```

### Extended Test Construct (`[[ ... ]]`)

The conditional test operator `[[ ... ]]` suppresses word splitting and globbing on all contained expansions.

Bash

```
var="a b"
if [[ $var == "a b" ]]; then
    echo "Match" # Safe: Evaluated without syntax errors
fi
```

### `case` Statement Targets and Patterns

Expressions inside `case ... in` blocks are evaluated as single fields.

Bash

```
var="hello world"
case $var in
    "hello world") echo "Matches" ;;
esac
```

### Array Subscripts / Index Numbers

Expansions inside array indices (`arr[$index]`) do not undergo word splitting.

Bash

```
idx="1 + 1"
arr[$idx]="value"  # Evaluates $idx as a single index expression
```

### Here-Strings (`<<<`)

The expression following `<<<` is treated as a single word stream.

Bash

```
var="line one\nline two"
cat <<< $var  # Passes the entire contents of $var as a single stdin stream
```

### Arithmetic Evaluation Contexts (`(( ... ))`)

Variables expanded inside arithmetic blocks are evaluated numerically rather than split into command line words.

Bash

```
a="5"
b="10"
(( result = $a + $b ))
```

## Summary Matrix

|Context|Syntax|Word Splitting Occurs?|
|---|---|---|
|**Command Argument**|`cmd $var`|**Yes**|
|**Command Argument (Quoted)**|`cmd "$var"`|**No**|
|**Array Initializer**|`arr=( $var )`|**Yes**|
|**Loop List**|`for x in $var`|**Yes**|
|**Single Bracket Test**|`[ $var = "x" ]`|**Yes** (Triggers syntax errors)|
|**Scalar Assignment**|`var=$val`|**No**|
|**Double Bracket Test**|`[[ $var == "x" ]]`|**No**|
|**Case Statement**|`case $var in`|**No**|
|**Here-String**|`cmd <<< $var`|**No**|
|**Array Subscript**|`arr[$var]=1`|**No**|

## Why Word Splitting Doesn't Happen in Process substitution

Word splitting only occurs when the shell parses **text printed directly onto the command line** from an unquoted parameter expansion (`$var`) or command substitution (`$(cmd)`).

Process substitution works completely differently:

1. **`$(cmd)` (Command Substitution):** Captures stdout and dumps the raw text directly into the command line buffer. The shell parser sees unquoted text, so it runs **Word Splitting** and **Globbing** on it.
    
2. **`<(cmd)` (Process Substitution):** Leaves the output inside a dynamic pipe stream and replaces `<(cmd)` with a file path string (e.g., `/dev/fd/63`). The shell parser **never sees the text inside the stream**—it only sees the string `/dev/fd/63`.
    

Because the receiving program reads bytes directly from the pipe file descriptor, the shell's `$IFS` word splitter is never invoked on that stream.

## Direct Comparison

Suppose `list.txt` contains filenames with spaces:

Plaintext

```
my file.txt
another file.txt
```

### 1. Command Substitution `$(...)` → Splits Data

Bash

```
# UNQUOTED $(...): Text is dumped to command line and split on spaces
for f in $(cat list.txt); do
    echo "File: $f"
done

# Output (BROKEN - 4 iterations):
# File: my
# File: file.txt
# File: another
# File: file.txt
```

### 2. Process Substitution `<(...)` → No Splitting

Bash

```
# <(...): Data stays in pipe, read line-by-line via read
while IFS= read -r f; do
    echo "File: $f"
done < <(cat list.txt)

# Output (SAFE - 2 iterations):
# File: my file.txt
# File: another file.txt
```