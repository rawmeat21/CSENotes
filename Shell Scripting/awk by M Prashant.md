![[Pasted image 20260816122650.png]]

![[Pasted image 20260816123703.png]]

![[Pasted image 20260816123724.png]]


Search multiple words:

```bash
awk '/Raju|Romit|nigga/ {print $0}' file.txt 
```

```bash
awk 'BEGIN{IGNORECASE=1} /Raju|Romit|nigga/ {print $0}' file.txt 
```

Get all names having character 'a':

```bash
awk '$2 ~ /a/ {print $2}' file.txt
```
(consider 2nd column is name)

![[Pasted image 20260816130415.png]]

### How to read logs in a range of time:

(How to work with time)

![[Pasted image 20260816130736.png]]

Just put the time under double quotes.


![[Pasted image 20260816130909.png]]

For substitution, use gsub.

`gsub(word_to_replace, word_to_replace_with)`



Ex: Replace all occurences of a word "Santa" to "Satan"
```bash
awk '{gsub("Santa","Satan");print $0} file.txt'
```

Ex- Get length of column values using `length()`:

```bash
awk '{print length($2)}' file.txt
```

To print the name too:

```bash
awk -F, 'NR>1 {print "Name: " $2 ", Number of chars: " length($2)}' myFile0.csv
```

Search for word (get position):

```bash
head -n 10 myFile0.csv | awk -F, -v word="mail"  'NR>1 {print "Position of " word " at line number " NR ": "index($0, word)}'
```
To convert to lowercase:

```bash
awk '{print tolower($2)}' file.txt
```

For uppercase, use `toupper()`.


![[Pasted image 20260816132928.png]]


```bash
awk 'BEGIN{} pattern{} END{}' file.txt
```

Variables in awk:

```bash
awk 'BEGIN{sum=0} {print sum}' file.txt
```

Find average age:

```bash
❯ awk -F, 'NR>1 {sum+=$6;cnt++} END{print "Average age: " sum/cnt}' myFile0.csv
Average age: 49.95
```
Note that you can avoid initializing the variables in BEGIN if you want.

A bit more safe, so we don't count empty rows:

```bash
❯ awk -F, 'NR>1 {sum+=$6;if(NF>0) cnt++} END{print "Average age: " sum/cnt}' myFile0.csv
Average age: 49.95
```

How to print number of lines:

```bash
awk '{} END{print NR}' file.txt
```

Just print NR in END.

Get length of longest line:

```bash
awk '{if(length($0) > mx) mx = length($0)} END{print "Length of longest line is: " mx}'  file.txt
```

Print OLD for age >21 else YOUNG:


```bash
awk -F, 'NR>1 {if($6>21) {print "OLD";old++} else {print "YOUNG";young++;}} END{print "Number of old people: " old ", Number of young people: " young}'
```


This is messy. Lets use files for conditions:

cond.awk:

```
'NR>1 {if($6>21) {print "OLD";old++} else {print "YOUNG";young++;}} END{print "Number of old people: " old ", Number of young people: " young}'
```

```bash
awk -f cond.awk file.txt
```


To write Awk code without constantly checking documentation, remember one core rule: **Awk syntax is C syntax.** Awk was created by the same engineers who developed C and Unix, so its loops, conditionals, and variables follow C patterns rather than Bash or Python conventions.

### Quick Comparison

|Language Feature|Awk (C-style)|Bash|Python|
|---|---|---|---|
|**Variables**|`x = 5`|`x=5` (read as `$x`)|`x = 5`|
|**Field / Column**|`$1` (1st column)|`cut -d' ' -f1`|`row[0]`|
|**If Statement**|`if (x > 5) { ... }`|`if [ $x -gt 5 ]; then ... fi`|`if x > 5:`|
|**For Loop**|`for (i=1; i<=N; i++)`|`for ((i=1; i<=N; i++))`|`for i in range(1, N+1):`|
|**Associative Loop**|`for (k in arr)`|`for k in "${!arr[@]}"`|`for k in dict:`|

### The 3 Rules to Never Forget Awk Syntax

1. **`$` means column, NOT variable:** In Bash, you write `$x` to read a variable. In Awk, `x` is a variable, and `$x` means "the field number stored in `x`". (`$1` is column 1, `$NF` is the last column).
    
2. **Use C-style braces `{}` and parentheses `()`:** No `then`, `fi`, `do`, or `done` like Bash; no indentation-based blocks like Python.
    
3. **The main loop is implicit:** Awk automatically reads input line-by-line. You only write loops _inside_ a line's execution block.
    

### Awk Syntax Guide

**Variables & Built-ins** Variables auto-initialize (`0` for numbers, `""` for strings).

- `NR`: Current line/record number.
    
- `NF`: Total number of fields/columns in current line.
    

Awk

```bash
count++                 # Increment variable
total += $2             # Add 2nd column to total
print NR, $1, total     # Print line number, column 1, running total
```

**Conditionals** Uses standard C relational operators (`==`, `!=`, `>`, `<`, `&&`, `||`) and `~` for regex matching.

Awk

```bash
if ($3 >= 50 && $1 ~ /ERROR/) {
    print "Critical line:", NR
} else {
    print "OK"
}
```

**Loops**

- **Standard For Loop:**
    
    Awk
    
    ```bash
    # Print every column on the current line individually
    for (i = 1; i <= NF; i++) {
        print "Column", i, ":", $i
    }
    ```
    
- **Associative Array Loop (Key-Value):**
    
    Awk
    
    ```bash
    # Count occurrences of values in column 1
    { counts[$1]++ }
    
    END {
        for (ip in counts) {
            print ip, "appeared", counts[ip], "times"
        }
    }
    ```
    
- **While Loop:**
    
    Awk
    
    ```bash
    i = 1
    while (i <= NF) {
        print $i
        i++
    }
    ```



### -v` Flags

Repeat `-v` for every variable you want to introduce. Always quote the shell variables in your command line to handle spaces properly.

Bash

```bash
USER="alice"
STATUS="active"
MIN_ID=1000

awk -v target_user="$USER" -v target_status="$STATUS" -v min_id="$MIN_ID" '
BEGIN { 
    print "Filtering for:", target_user, target_status 
}
$1 == target_user && $2 == target_status && $3 >= min_id { 
    print $0 
}
' data.txt
```

