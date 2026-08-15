`sed` (short for **S**tream **Ed**itor) is one of the most powerful text-processing tools in Unix/Linux.

Unlike interactive text editors (like `nano` or `vim`), `sed` parses text **line-by-line**, applies transformation rules, and outputs the result immediately to standard output.

## The Fundamental Syntax

Bash

```
sed [options] 'command' filename
```

By default, `sed` prints the entire file along with any modifications to the terminal, **without altering the original file**.

## 1. Search and Replace (The `s` Command)

This is `sed`'s most common use case.

### Basic Substitution

Bash

```
sed 's/cat/dog/' file.txt
```

- **`s`**: Stands for substitute.
    
- **`cat`**: The target pattern to find.
    
- **`dog`**: The replacement string.
    
- ⚠️ **Note:** By default, this only replaces the **first occurrence** on each line!
    

### Essential Flags for Substitution

|Flag|Purpose|Example|Result|
|---|---|---|---|
|**`g`**|**Global:** Replace _all_ occurrences on each line|`sed 's/cat/dog/g' file.txt`|Replaces every "cat" on every line.|
|**`i`**|**Ignore Case:** Match pattern case-insensitively|`sed 's/cat/dog/gi' file.txt`|Matches "Cat", "CAT", "cAt", etc.|
|**`2`**|**Nth Occurrence:** Replace only the Nth match on a line|`sed 's/cat/dog/2' file.txt`|Replaces only the 2nd "cat" per line.|

### Using Custom Delimiters

If you are replacing file paths containing slashes (`/`), using `/` as a delimiter creates readable clutter:

Bash

```
# Hard to read ("Leaning Toothpick Syndrome"):
sed 's/\/usr\/local\/bin/\/usr\/bin/g' file.txt

# Clean & readable (using '#' or '@' instead):
sed 's#/usr/local/bin#/usr/bin#g' file.txt
```

## 2. In-Place File Editing (`-i`)

To save changes directly to the original file instead of just printing them to the screen:

Bash

```
# Modify file.txt directly
sed -i 's/old_text/new_text/g' file.txt

# Modify file.txt directly AND create a backup named file.txt.bak
sed -i.bak 's/old_text/new_text/g' file.txt
```

## 3. Printing Specific Lines (`-n` and `p`)

The `-n` flag suppresses automatic output, allowing you to print only specific lines using `p`.

Bash

```
# Print only line 5
sed -n '5p' file.txt

# Print a range of lines (lines 10 through 20)
sed -n '10,20p' file.txt

# Print lines matching a specific pattern (like grep)
sed -n '/ERROR/p' file.txt
```

## 4. Deleting Lines (`d`)

Bash

```
# Delete line 1 (e.g., remove a CSV header)
sed '1d' file.txt

# Delete lines 5 through 10
sed '5,10d' file.txt

# Delete the LAST line of a file
sed '$d' file.txt

# Delete all empty lines
sed '/^$/d' file.txt

# Delete lines containing a specific keyword
sed '/DEBUG/d' file.txt
```

## 5. Advanced: Extended Regex & Backreferences (`-E`)

The `-E` flag enables Extended Regular Expressions, allowing capture groups with `()` without needing clunky backslashes.

### Reordering Words (Capture Groups)

Use `\1`, `\2`, etc., to reference matched groups:

Bash

```
# Input:  "John Smith"
# Output: "Smith, John"

sed -E 's/([A-Za-z]+) ([A-Za-z]+)/\2, \1/' names.txt
```

### Whole Word Boundaries (`\b`)

To avoid replacing partial matches (e.g., replacing "cat" inside "category"):

Bash

```
# ❌ Replaces "category" -> "dogegory"
sed 's/cat/dog/g' file.txt

# ✅ Replaces ONLY the standalone word "cat"
sed -E 's/\bcat\b/dog/g' file.txt
```

## Quick-Reference Cheat Sheet

|Command|Action|
|---|---|
|`sed 's/foo/bar/g' f.txt`|Replace all `foo` with `bar` (Output to terminal).|
|`sed -i 's/foo/bar/g' f.txt`|Replace all `foo` with `bar` directly in `f.txt`.|
|`sed -n '3,7p' f.txt`|Print only lines 3 through 7.|
|`sed '/pattern/d' f.txt`|Delete all lines matching `pattern`.|
|`sed '/^$/d' f.txt`|Remove all blank lines.|
|`sed -E 's/\bword\b/new/g'`|Replace whole words only (ignoring partial matches).|