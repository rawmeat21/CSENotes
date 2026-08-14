## 1. File Operators

|Operator|Details|
|---|---|
|`-e "$file"`|Returns true if the file/path exists.|
|`-d "$file"`|Returns true if the file exists and is a directory.|
|`-f "$file"`|Returns true if the file exists and is a regular file.|
|`-h "$file"`|Returns true if the file exists and is a symbolic link (also `-L`).|

###  Bonus File Operators

|Operator|Details|
|---|---|
|`-r "$file"`|True if the file exists and is **readable**.|
|`-w "$file"`|True if the file exists and is **writable**.|
|`-x "$file"`|True if the file exists and is **executable**.|
|`-s "$file"`|True if the file exists and has a **size greater than 0** (not empty).|
|`"$f1" -nt "$f2"`|True if file `$f1` is **newer than** `$f2`.|

## 2. String Comparators

|Operator|Details|
|---|---|
|`-z "$str"`|True if length of string is **zero** (empty).|
|`-n "$str"`|True if length of string is **non-zero** (not empty).|
|`"$str" = "$str2"`|True if string `$str` is equal to `$str2` _(Note: in `[[ ]]`, `==` is preferred)_.|
|`"$str" != "$str2"`|True if the strings are **not equal**.|

### Bonus String Operators (Inside `[[ ]]`)

|Operator|Details|
|---|---|
|`"$str" =~ $regex`|True if string matches the specified **regular expression**.|
|`"$str" == pattern*`|True if string matches a **glob pattern** (wildcard).|

## 3. Integer Comparators

|Operator|Details|
|---|---|
|`"$int1" -eq "$int2"`|True if the integers are **equal** (=).|
|`"$int1" -ne "$int2"`|True if the integers are **not equal** (=).|
|`"$int1" -gt "$int2"`|True if `$int1` is **greater than** `$int2` (>).|
|`"$int1" -ge "$int2"`|True if `$int1` is **greater than or equal to** `$int2` (≥).|
|`"$int1" -lt "$int2"`|True if `$int1` is **less than** `$int2` (<).|
|`"$int1" -le "$int2"`|True if `$int1` is **less than or equal to** `$int2` (≤).|

## Practical Usage Examples

### Example 1: Validating Directory & Files

Bash

```bash
#!/bin/bash

log_dir="/var/log/myapp"
config_file="/etc/myapp.conf"

# Check if directory exists; if not, create it
if [[ ! -d "$log_dir" ]]; then
  echo "Directory missing. Creating $log_dir..."
  mkdir -p "$log_dir"
fi

# Check if file exists AND is readable
if [[ -f "$config_file" && -r "$config_file" ]]; then
  echo "Config file ready to read."
else
  echo "Error: Config file is missing or unreadable!" >&2
fi
```

### Example 2: String & Regex Validation

Bash

```bash
#!/bin/bash

read -p "Enter your email: " email

# Check if empty (-z)
if [[ -z "$email" ]]; then
  echo "Error: Input cannot be empty."
# Check regex matching (=~)
elif [[ "$email" =~ ^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$ ]]; then
  echo "Valid email address!"
else
  echo "Invalid email format."
fi
```

### Example 3: Numerical Range Checking

Bash

```bash
#!/bin/bash

age=21

# Combine integer comparators using logical operators
if [[ "$age" -ge 18 && "$age" -le 65 ]]; then
  echo "Eligible for working age bracket."
fi
```


### Problem

If you write `if [[ $1 + 5 > 91 ]]`, Bash will not throw a syntax error, but **it will give wrong and buggy results**.

### Why `[[ $1 + 5 > 91 ]]` Fails

1. **`>` means String Comparison in `[[ ]]`**
    
    - Inside `(( ... ))`, `>` compares **numbers** (15>10).
        
    - Inside `[[ ... ]]`, `>` compares **strings alphabetically** ("9">"10" is `true` because '9' comes after '1' in ASCII).
        
2. **No Automatic Math in `[[ ]]`**
    
    - Inside `(( ... ))`, arithmetic expressions are automatically calculated.
        
    - Inside `[[ ... ]]`, `$1 + 5` is not evaluated as addition; Bash treats it as a literal string containing a plus sign (e.g., `"10 + 5"`).
        
3. **Different Operators for Numbers**
    
    - To compare numbers in `[[ ]]`, you must use integer operators like `-gt`, `-lt`, `-eq`.
        

### How to rewrite it using `[[ ]]` (If you really had to)

To make it work inside `[[ ... ]]`, you would need to use **arithmetic expansion** `$(( ... ))` for the addition and `-gt` for the comparison:

Bash

```
if [[ $(( $1 + 5 )) -gt 91 ]]; then
  echo "$1 is greater than 86"
fi
```

### Summary: Use the Right Tool for the Job

- **Use `(( ... ))` for Math:** It natively supports C-style arithmetic (`+`, `-`, `*`, `/`), variables without `$`, and standard math operators (`>`, `<`, `>=`, `==`).
    
- **Use `[[ ... ]]` for Conditions:** Best for string matching, file checks, regex, and standard logical tests.

