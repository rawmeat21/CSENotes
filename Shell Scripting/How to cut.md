`cut` is a lightweight, fast, and essential command-line utility used to **extract specific columns, fields, or characters** from each line of text or standard input.

While `sed` is used for searching and replacing text, `cut` is designed for slicing data vertically (like taking a single column out of a spreadsheet).

## The Fundamental Syntax

Bash

```
cut OPTION... [FILE]...
```

By default, `cut` reads from standard input or a specified file, slices out the requested parts of each line, and prints the result to standard output.

## 1. Extracting by Character (`-c`)

Use `-c` when you know the exact character position(s) you want to pull from every line.

### Specifying Character Ranges

|Syntax|Meaning|Example|Input: `HelloWorld`|Output|
|---|---|---|---|---|
|**`cut -c N`**|Nth character|`cut -c 3`|`HelloWorld`|`l`|
|**`cut -c N,M`**|Specific positions|`cut -c 1,5,10`|`HelloWorld`|`Hod`|
|**`cut -c N-M`**|Range from N to M|`cut -c 1-5`|`HelloWorld`|`Hello`|
|**`cut -c N-`**|Nth char to end|`cut -c 6-`|`HelloWorld`|`World`|
|**`cut -c -M`**|Start of line to Mth char|`cut -c -5`|`HelloWorld`|`Hello`|

## 2. Extracting by Delimited Fields (`-d` and `-f`)

This is `cut`'s most powerful feature. When working with tabular data (like CSV files or log files), you can split lines by a **delimiter** (`-d`) and pick specific **fields** (`-f`).

- **`-d 'char'`**: Sets the delimiter character (default is `TAB`).
    
- **`-f N`**: Specifies which field column(s) to output.
    

### Examples

#### Example A: Extracting Usernames from `/etc/passwd`

The system file `/etc/passwd` uses colons (`:`) to separate fields, where the 1st field is the username:

Bash

```
cut -d ':' -f 1 /etc/passwd
# Output:
# root
# daemon
# bin
```

#### Example B: Parsing CSV Files

Extract the 1st and 3rd columns from a comma-separated file:

Bash

```
# Data: Alice,25,Engineer
cut -d ',' -f 1,3 employees.csv
# Output: Alice,Engineer
```

#### Example C: Extracting Field Ranges

Bash

```
# Extract fields 2 through 4
cut -d ',' -f 2-4 data.csv

# Extract everything from field 3 to the end of the line
cut -d ',' -f 3- data.csv
```

## 3. Useful Flags & Options

### A. The `--complement` Flag (Invert Selection)

The `--complement` option tells `cut` to print **everything EXCEPT** the specified characters or fields.

Bash

```
# Print every field EXCEPT field 2:
cut -d ',' --complement -f 2 data.csv

# Print every character EXCEPT the first 3:
cut -c 1-3 --complement file.txt
```

### B. The `-s` Flag (Suppress Lines Without Delimiter)

By default, if a line does not contain the delimiter you specified, `cut` prints the entire line as-is. Use `-s` (`--only-delimited`) to ignore lines that don't have the delimiter.

Bash

```
# Only print lines that actually contain a comma
cut -d ',' -f 1 -s file.txt
```

## 4. Common Pitfall: Multiple Spaces

A classic trap when using `cut` is trying to delimit by spaces on commands with variable spacing (like `ps aux` or `ls -l`):

Bash

```
# ❌ THIS FAILS OR GIVES UNEXPECTED RESULTS:
ls -l | cut -d ' ' -f 5
```

**Why it fails:** `cut` treats _every single space_ as a delimiter boundary. If there are 4 spaces between columns, `cut` sees 4 empty fields!

### The Fix

Either compress multiple spaces into a single space first using `tr -s ' '`:

Bash

```
# ✅ Squish multiple spaces into one space first
ls -l | tr -s ' ' | cut -d ' ' -f 5
```

Or use `awk` instead, which handles variable spacing automatically:

Bash

```
# ✅ Awk automatically handles multiple spaces
ls -l | awk '{print $5}'
```

## Quick-Reference Cheat Sheet

| Command                       | What it does                                        |
| ----------------------------- | --------------------------------------------------- |
| `cut -c3 file.txt`            | Print the 3rd character of every line.              |
| `cut -c1-10 file.txt`         | Print the first 10 characters of every line.        |
| `cut -d',' -f1 file.csv`      | Extract column 1 from a CSV file.                   |
| `cut -d':' -f1,7 /etc/passwd` | Extract fields 1 and 7 from a colon-delimited file. |
| `cut -d' ' -f2 --complement`  | Print all fields _except_ field 2.                  |
| `cut -d' ' -f1 -s file.txt`   | Extract field 1, ignoring lines without spaces.     |