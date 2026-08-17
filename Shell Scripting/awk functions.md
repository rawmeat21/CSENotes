**String functions**

1. `length(s)` — length of string `s` (or `$0` if omitted; number of elements if `s` is an array)
2. `substr(s, m, n)` — substring of `s` starting at position `m`, `n` characters long (n optional = to end)
3. `index(s, t)` — position of first occurrence of `t` in `s`, 0 if not found
4. `split(s, arr, fs)` — splits `s` into `arr` using field separator `fs` (regex or string), returns count
5. `sub(re, repl, target)` — replaces first match of `re` with `repl` in `target` (default `$0`), returns count (0 or 1)
6. `gsub(re, repl, target)` — like `sub` but replaces all matches, returns count
7. `gensub(re, repl, how, target)` — gawk extension; non-destructive version, supports backreferences (`\1`) and "g" for all
8. `match(s, re)` — tests if `re` occurs in `s`, sets `RSTART`/`RLENGTH`, returns position
9. `sprintf(fmt, ...)` — like `printf` but returns a formatted string instead of printing
10. `toupper(s)` / `tolower(s)` — case conversion

**Numeric/math functions**

11. `int(x)` — truncates to integer
12. `sqrt(x)` — square root
13. `exp(x)` / `log(x)` — exponential and natural log
14. `sin(x)` / `cos(x)` / `atan2(y, x)` — trig functions
15. `rand()` / `srand(seed)` — pseudo-random number generator [0,1) and its seeding

**I/O and system functions**

16. `getline` — reads next line of input into `$0` (several variants: `getline var`, `getline < file`, `cmd | getline`)
17. `close(file/cmd)` — closes an open file or pipe, important when reusing filenames/commands or reading return status
18. `system(cmd)` — runs a shell command, returns exit status
19. `fflush([file])` — flushes output buffers (gawk; useful when interleaving with `system()`)

**Type/misc**

20. `typeof(x)` — gawk extension, returns the type ("strnum", "number", "array", etc.)


**String functions**

**1. `length(s)`**

bash

```bash
# Lines longer than 200 chars
awk 'length($0) > 200' file.log

# Average line length
awk '{s+=length($0)} END{print s/NR}' file.txt

# Number of elements in an array
awk '{a[$1]++} END{print length(a)}' data.txt
```

**2. `substr(s, m, n)`**

bash

```bash
# Extract date from fixed-width timestamp
awk '{print substr($0, 1, 10)}' app.log

# Truncate long paths for display
awk '{print substr($0, 1, 40) "..."}' paths.txt

# Get file extension
awk '{print substr($1, length($1)-2)}' filelist.txt
```

**3. `index(s, t)`**

bash

```bash
# Check if line contains a literal string
awk 'index($0, "PermitRootLogin")' sshd_config

# Check if path is absolute
awk 'index($0, "/") == 1' paths.txt

# Split manually using index + substr
awk '{p = index($0, ":"); print substr($0, 1, p-1)}' file.txt
```

**4. `split(s, arr, fs)`**

bash

```bash
# Parse /etc/passwd fields
awk -F: '{split($0, a, ":"); print a[7]}' /etc/passwd

# Tokenize a sentence into words, count them
awk '{n = split($0, words, " "); print n}' sentence.txt

# Re-split PATH variable
echo $PATH | awk '{n = split($0, dirs, ":"); for(i=1;i<=n;i++) print dirs[i]}'
```

**5. `sub(re, repl, target)`**

bash

```bash
# Strip first comment only
awk '{sub(/#.*/, ""); print}' config.conf

# Fix first typo occurrence per line
awk '{sub(/teh/, "the"); print}' draft.txt

# Insert prefix once
awk '{sub(/^/, ">> "); print}' notes.txt
```

**6. `gsub(re, repl, target)`**

bash

```bash
# Strip ANSI color codes
awk '{gsub(/\x1b\[[0-9;]*m/, ""); print}' colored.log

# Normalize whitespace
awk '{gsub(/[ \t]+/, " "); print}' messy.csv

# Count occurrences of a word
awk '{n = gsub(/error/, "error"); total += n} END{print total}' app.log
```

**7. `gensub(re, repl, how, target)`** (gawk only)

bash

```bash
# Swap "Last, First" -> "First Last"
awk '{print gensub(/([A-Za-z]+), ([A-Za-z]+)/, "\\2 \\1", "g")}' names.txt

# Replace only the 2nd occurrence
awk '{print gensub(/foo/, "bar", 2)}' file.txt

# Non-destructive: keep $0 intact, store result separately
awk '{clean = gensub(/[0-9]+/, "N", "g"); print clean}' logs.txt
```

**8. `match(s, re)`**

bash

```bash
# Extract IP address from a log line
awk '{if (match($0, /[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/)) print substr($0, RSTART, RLENGTH)}' auth.log

# Validate email format
awk '{if (match($0, /^[^@]+@[^@]+\.[^@]+$/)) print "valid:", $0}' emails.txt

# Find position of pattern to slice around it
awk '{if (match($0, /ERROR/)) print substr($0, 1, RSTART-1)}' app.log
```

**9. `sprintf(fmt, ...)`**

bash

```bash
# Zero-pad IDs into filenames
awk '{fname = sprintf("out_%03d.txt", NR); print > fname}' data.txt

# Format numbers to fixed decimal precision
awk '{print sprintf("%.2f", $1)}' prices.txt

# Right-align columns
awk '{printf sprintf("%10s: %5d\n", $1, $2)}' report.txt
```

**10. `toupper(s)` / `tolower(s)`**

bash

```bash
# Case-insensitive dedup
awk '{print tolower($1)}' users.txt | sort -u

# Uppercase headers
awk 'NR==1{print toupper($0); next} {print}' data.csv

# Case-fold before sort
awk '{print tolower($0), $0}' words.txt | sort | cut -d' ' -f2-
```

**Numeric/math functions**

**11. `int(x)`**

bash

```bash
# Truncate fractional seconds
awk '{print int($3)}' timings.txt

# Bucket values into bins of 10
awk '{bin = int($1/10)*10; count[bin]++} END{for(b in count) print b, count[b]}' scores.txt

# Integer division manually
awk '{print int($1/$2), $1 - int($1/$2)*$2}' pairs.txt
```

**12. `sqrt(x)`**

bash

```bash
# Standard deviation
awk '{sum+=$1; sumsq+=$1*$1; n++} END{mean=sum/n; print sqrt(sumsq/n - mean*mean)}' data.txt

# Distance between two coordinates
awk '{print sqrt(($1-$3)^2 + ($2-$4)^2)}' coords.txt

# RMS of a signal column
awk '{s+=$1*$1} END{print sqrt(s/NR)}' signal.txt
```

**13. `exp(x)` / `log(x)`**

bash

```bash
# Convert log-scale sensor data back to linear
awk '{print exp($1)}' logscale.dat

# Natural log of a column (e.g., for entropy calc)
awk '{print log($1)}' counts.txt

# Exponential decay smoothing
awk 'NR==1{s=$1; print s; next} {s = 0.3*$1 + 0.7*s; print s}' readings.txt
```

**14. `sin(x)` / `cos(x)` / `atan2(y, x)`**

bash

```bash
# Generate a sine lookup table
awk 'BEGIN{for(i=0;i<360;i++) print i, sin(i*3.14159/180)}'

# Compute bearing angle between two points
awk '{print atan2($4-$2, $3-$1) * 180/3.14159}' points.txt

# Simple waveform for plotting
awk 'BEGIN{for(i=0;i<100;i++) print i, cos(i*0.1)}' > wave.dat
```

**15. `rand()` / `srand(seed)`**

bash

```bash
# Generate randomized stress-test input
awk 'BEGIN{srand(); for(i=0;i<10;i++) print int(rand()*100)}'

# Shuffle lines from a file
awk 'BEGIN{srand()} {print rand(), $0}' file.txt | sort -n | cut -d' ' -f2-

# Reproducible sequence with fixed seed
awk 'BEGIN{srand(42); for(i=0;i<5;i++) print rand()}'
```

**I/O and system functions**

**16. `getline`**

bash

```bash
# Merge header + data pair into one line
awk '{getline nextline; print $0, nextline}' pairs.txt

# Pull content from a second file mid-processing
awk '{while ((getline line < "extra.txt") > 0) print line; close("extra.txt")}' main.txt

# Capture command output into a variable
awk 'BEGIN{"date" | getline d; print d}'
```

**17. `close(file/cmd)`**

bash

```bash
# Split log by hour, reset file each time it's reused
awk '{f=$1".log"; print > f; close(f)}' access.log

# Rerun same shell command multiple times (must close pipe first)
awk '{"date" | getline d; print d; close("date")}' triggers.txt

# Get exit status of a piped command
awk 'BEGIN{cmd="grep foo file.txt"; while((cmd | getline line) > 0) print line; status = close(cmd); print "exit:", status}'
```

**18. `system(cmd)`**

bash

```bash
# Trigger notification on pattern match
awk '/OOM-killer/{system("notify-send \"OOM detected\"")}' /var/log/kern.log

# Restart a service on match
awk '/CRITICAL/{system("systemctl restart myservice")}' app.log

# Create a directory as a side effect
awk '{d="output_" $1; system("mkdir -p " d); print > (d "/" NR ".txt")}' data.txt
```

**19. `fflush([file])`**

bash

```bash
# Real-time output into tee
awk '{print; fflush()}' bigfile.txt | tee live.log

# Flush before invoking system()
awk '{print "processing:", $0; fflush(); system("sleep 1")}' tasks.txt

# Flush a specific output file
awk '{print > "out.log"; fflush("out.log")}' input.txt
```

**Type/misc**

**20. `typeof(x)`** (gawk only)

bash

```bash
# Debug why sorting isn't numeric
awk '{print typeof($2)}' data.csv

# Distinguish uninitialized vs empty string
awk 'BEGIN{print typeof(x); x=""; print typeof(x)}'

# Validate array vs scalar before use
awk '{a[$1]=$2} END{print typeof(a)}' file.txt
```

