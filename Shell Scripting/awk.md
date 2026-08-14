`awk` is a complete pattern-scanning and text-processing language built right into the terminal.

While `cut` is great for simple character/field splitting and `sed` is great for line editing, `awk` excels at **structured, columnar data processing, math calculations, and report generation**.

## The Fundamental Syntax

Bash

```
awk 'pattern { action }' filename
```

- **Pattern**: Tells `awk` **which lines** to target (e.g., lines matching a regex or numeric condition). If omitted, `awk` targets _all_ lines.
    
- **Action**: Code wrapped in `{ ... }` that tells `awk` **what to do** with targeted lines. If omitted, `awk` simply prints the line.
    

## 1. Essential Built-In Variables

`awk` automatically breaks every line into fields (columns) separated by whitespace (spaces or tabs) by default.

|Variable|Meaning|Example|
|---|---|---|
|**`$0`**|The entire current line|`awk '{print $0}'`|
|**`$1`, `$2`, `$3`...**|The 1st, 2nd, 3rd field/column|`awk '{print $1, $3}'`|
|**`$NF`**|The **last** field on the line|`awk '{print $NF}'`|
|**`NF`**|**N**umber of **F**ields (total column count)|`awk '{print NF}'`|
|**`NR`**|**N**umber of **R**ecords (current line number)|`awk '{print NR, $0}'`|
|**`FS`**|**F**ield **S**eparator (default: whitespace)|Set via `-F` flag|
|**`OFS`**|**O**utput **F**ield **S**eparator|Default is a single space|

## 2. Printing & Field Extraction

### A. Print Specific Columns

Bash

```
# Print username ($1) and default shell ($7) from /etc/passwd
awk -F ':' '{print $1, $7}' /etc/passwd
```

### B. Changing the Field Separator (`-F`)

Unlike `cut`, `awk` can handle multi-character delimiters or treat consecutive spaces as a single delimiter automatically.

Bash

```
# Custom delimiter comma (CSV)
awk -F ',' '{print $1}' data.csv

# Custom delimiter colon (:)
awk -F ':' '{print $1}' /etc/passwd
```

### C. Formatting Output

Commas between fields in `print` insert the `OFS` (a space by default). Concatenating without a comma joins them directly:

Bash

```
# Output: User: alice | Shell: /bin/bash
awk -F ':' '{print "User: " $1 " | Shell: " $7}' /etc/passwd
```

## 3. Filtering with Patterns & Conditions

You can place conditions **before** the `{ action }` block to target specific lines.

### A. Numeric Comparisons

Bash

```
# Print lines where column 3 is greater than 100
awk '$3 > 100 {print $1, $3}' data.txt

# Print lines where column 2 equals 0
awk '$2 == 0 {print $0}' data.txt
```

### B. Text & Regular Expression Matching

Use `/pattern/` to target lines matching a pattern:

Bash

```
# Print lines containing "ERROR"
awk '/ERROR/ {print $0}' app.log

# Print lines where field 1 STARTS with "admin"
awk '$1 ~ /^admin/ {print $0}' users.txt

# Print lines where field 1 DOES NOT match "guest"
awk '$1 !~ /guest/ {print $0}' users.txt
```

### C. Range Filtering by Line Numbers (`NR`)

Bash

```
# Print lines 5 through 10 (like sed -n '5,10p')
awk 'NR >= 5 && NR <= 10' file.txt
```

## 4. `BEGIN` and `END` Blocks

`awk` gives you two special execution blocks:

- **`BEGIN { ... }`**: Runs **once before** processing any input lines (great for setting headers or variables).
    
- **`END { ... }`**: Runs **once after** all input lines have been processed (great for calculating totals or averages).
    

```
         ┌──────────────────────────────────────┐
         │              BEGIN { }               │  <- Runs 1x at start
         └──────────────────┬───────────────────┘
                            │
                            ▼
    ┌────────────────────────────────────────────────┐
    │  for each line in file:                        │
    │      pattern { action }                       │  <- Runs for EVERY line
    └───────────────────────┬────────────────────────┘
                            │
                            ▼
         ┌──────────────────────────────────────┐
         │               END { }                │  <- Runs 1x at end
         └──────────────────────────────────────┘
```

### Example: Column Sum & Average Calculation

Bash

```
# Calculate the total and average size of files in a directory
ls -l | awk '
BEGIN {
    print "--- Starting Calculation ---"
}
NR > 1 {
    sum += $5   # $5 is the file size column in `ls -l`
    count++
}
END {
    print "Total Files: " count
    print "Total Bytes: " sum
    print "Average Size: " sum / count " bytes"
}'
```

## 5. Useful Built-in String & Math Functions

Inside `awk` action blocks, you can use built-in functions just like in programming languages:

Bash

```
# length(): Get string length
awk '{print $1, length($1)}' file.txt

# tolower() & toupper(): Change case
awk '{print toupper($1)}' file.txt

# substr(string, start, length): Extract substring (1-based index)
awk '{print substr($1, 1, 3)}' file.txt   # Print first 3 chars of field 1
```

## Quick-Reference Cheat Sheet

|Task|Command|
|---|---|
|**Print 1st column**|`awk '{print $1}' file.txt`|
|**Print last column**|`awk '{print $NF}' file.txt`|
|**Print line numbers with text**|`awk '{print NR, $0}' file.txt`|
|**Change delimiter to comma**|`awk -F ',' '{print $1, $2}' file.csv`|
|**Filter by text match**|`awk '/pattern/ {print $0}' file.txt`|
|**Filter by number comparison**|`awk '$3 >= 50 {print $1}' file.txt`|
|**Count non-empty lines**|`awk 'NF > 0 {count++} END {print count}' file.txt`|
|**Sum numbers in column 1**|`awk '{sum += $1} END {print sum}' numbers.txt`|
