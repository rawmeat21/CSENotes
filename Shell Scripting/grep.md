## Pattern Engines & Variants

|Mode|Equivalent Command|POSIX Standard|Description|
|---|---|---|---|
|**BRE**|`grep`|Standard BRE|Basic Regular Expressions. Operators like `?`, `+`, `{`, `|
|**ERE**|`grep -E` (`egrep`)|POSIX ERE|Extended Regular Expressions. Metacharacters `?`, `+`, `{`, `|
|**Fixed**|`grep -F` (`fgrep`)|POSIX|Treats patterns as literal fixed strings (disables regex interpretation completely for high speed).|
|**PCRE**|`grep -P`|Non-POSIX (GNU)|Uses the PCRE2 library. Supports advanced features like lookaheads `(?=...)`, lookbehinds `(?<=...)`, and non-greedy quantifiers `*?`.|

## Option Flag Reference

|Flag|Long Option|Functional Purpose|
|---|---|---|
|`-i`|`--ignore-case`|Perform case-insensitive matching.|
|`-w`|`--word-regexp`|Match pattern only when bounded by non-word characters.|
|`-x`|`--line-regexp`|Match pattern only if it spans the entire line.|
|`-v`|`--invert-match`|Select non-matching lines.|
|`-n`|`--line-number`|Prefix output with 1-based line numbers.|
|`-c`|`--count`|Output total count of matching lines per file.|
|`-l`|`--files-with-matches`|Print only names of files containing matches.|
|`-L`|`--files-without-match`|Print only names of files containing NO matches.|
|`-o`|`--only-matching`|Print only the exact matched substring, not the whole line.|
|`-r`|`--recursive`|Recurse directories (skip symlinks).|
|`-R`|`--dereference-recursive`|Recurse directories (follow all symlinks).|
|`-A N`|`--after-context=N`|Print N lines after match.|
|`-B N`|`--before-context=N`|Print N lines before match.|
|`-C N`|`--context=N`|Print N lines before and after match.|
|`-m N`|`--max-count=N`|Stop reading a file after N matching lines.|
|`-q`|`--quiet`, `--silent`|Exit immediately with status 0 on first match; suppress output.|
|`-b`|`--byte-offset`|Print 0-based byte offset before each matching line/substring.|
|`-e PAT`|`--regexp=PAT`|Explicitly specify a pattern (allows multiple `-e` conditions).|
|`-f FILE`|`--file=FILE`|Read patterns from a file (one pattern per line).|
|`-I`|Direct flag|Ignore binary files (treat binary files as having no matches).|
|`-z`|`--null-data`|Treat input as a set of lines separated by NUL bytes instead of newlines.|
|`-Z`|`--null`|Output a NUL byte after file names instead of a newline.|

## 25 Practical Use Cases

### 1. Basic Matching & Case Sensitivity

#### 1. Case-Insensitive Matching (`-i`)

Find `error`, `ERROR`, or `Error` in system logs:

Bash

```
grep -i "error" /var/log/syslog
```

#### 2. Exact Word Boundaries (`-w`)

Match `port` as an isolated word, ignoring `portable`, `viewport`, or `sport`:

Bash

```
grep -w "port" /etc/services
```

#### 3. Full-Line Match (`-x`)

Filter for lines that consist _exclusively_ of the string `root`:

Bash

```
grep -x "root" /etc/users
```

#### 4. Invert Matching (`-v`)

Filter out comment lines starting with `#` and empty lines:

Bash

```
grep -v -E '^\s*(#|$)' /etc/ssh/sshd_config
```

#### 5. Output Matched Substrings Only (`-o`)

Extract all IPv4 addresses from an engine log using `grep -o`:

Bash

```
grep -E -o '([0-9]{1,3}\.){3}[0-9]{1,3}' /var/log/nginx/access.log
```

### 2. Output Formatting & Counters

#### 6. Display Line Numbers (`-n`)

Show matching line numbers alongside the matched string:

Bash

```
grep -n "main" src/server.c
```

#### 7. Count Total Matching Lines (`-c`)

Count the number of failed login attempts without printing each line:

Bash

```
grep -c "Failed password" /var/log/auth.log
```

#### 8. Print Byte Offsets (`-b`)

Locate exact byte offsets of matching headers inside raw binary or structured data streams:

Bash

```
grep -b -o "ELF" /bin/ls
```

#### 9. Limit Total Match Count (`-m`)

Stop processing after finding the first 3 occurrences of a memory allocation failure:

Bash

```
grep -m 3 "out of memory" /var/log/kern.log
```

### 3. File Discovery & Multfile Handling

#### 10. List Only Files Containing Matches (`-l`)

Find all C source files that invoke `malloc`:

Bash

```
grep -l "malloc" src/*.c
```

#### 11. List Only Files WITHOUT Matches (`-L`)

Identify C source files that do not include `config.h`:

Bash

```
grep -L '#include "config.h"' src/*.c
```

#### 12. Safe Filename Output via NUL Separator (`-Z`)

Pass files containing spaces safely into `xargs`:

Bash

```
grep -lZ "TODO" ./*.py | xargs -0 rm
```

### 4. Context Inspection

#### 13. Print Lines After Match (`-A`)

Show 5 lines of stack trace directly _after_ an exception:

Bash

```
grep -A 5 "NullPointerException" app.log
```

#### 14. Print Lines Before Match (`-B`)

Show 3 lines of configuration state directly _before_ an error line:

Bash

```
grep -B 3 "CRITICAL" /var/log/app.log
```

#### 15. Print Surrounding Context (`-C`)

Show 2 lines before and 2 lines after a function declaration:

Bash

```
grep -C 2 "int init_module(void)" driver.c
```

### 5. Recursive Search & File Exclusions

#### 16. Basic Recursive Search (`-r`)

Search all files under the current tree for a symbol:

Bash

```
grep -r "sys_call_table" /usr/src/linux/
```

#### 17. Include Specific File Masks (`--include`)

Search strictly within `.c` and `.h` files:

Bash

```
grep -r --include="*.[ch]" "PAGE_SIZE" ./kernel/
```

#### 18. Exclude Specific Files and Directories (`--exclude`, `--exclude-dir`)

Ignore `node_modules`, `.git`, and compiled `.min.js` files during search:

Bash

```
grep -r --exclude-dir={.git,node_modules,build} --exclude="*.min.js" "API_KEY" .
```

### 6. Advanced Regex & Engine Modes

#### 19. Extended Regex Logic (`-E`)

Match alternate expressions or apply non-escaped grouping quantifiers:

Bash

```
grep -E "ERR(OR)?|WARN(ING)?" application.log
```

#### 20. High-Performance Fixed String Matching (`-F`)

Search for literal strings containing special characters without escaping:

Bash

```
grep -F "https://example.com/api?v=1.0&status=active" access.log
```

#### 21. PCRE Lookaheads & Lookbehinds (`-P`)

Extract key values from a key-value pair using a positive lookbehind:

Bash

```
# Extracts value after 'DB_PASSWORD=' without including the key
grep -P -o '(?<=DB_PASSWORD=)\w+' .env
```

#### 22. Multiple Patterns via Flag or File (`-e`, `-f`)

Combine multiple search patterns via `-e` or pull them from an external pattern file:

Bash

```
# Via multiple -e flags
grep -e "sys_open" -e "sys_read" -e "sys_write" kernel_symbols.txt

# Via pattern file (one regex per line)
grep -f patterns.txt target_file.log
```

### 7. Scripting, Systems & Binary Control

#### 23. Quiet Execution for Conditional Shell Logic (`-q`)

Check if a user exists in `/etc/passwd` silently in a shell script:

Bash

```
if grep -q "^docker:" /etc/passwd; then
    echo "Docker group user exists"
fi
```

#### 24. Process Only Text Files / Skip Binaries (`-I`)

Avoid scanning compiled binaries during broad text searches:

Bash

```
grep -I -r "DEBUG_LEVEL" ./
```

#### 25. Multiline / NUL Data Processing (`-z`)

Process multiline blocks separated by NUL characters or search across raw data buffers:

Bash

```
grep -z -P 'function_start\n\s+call_subroutine' binary_dump.bin
```