### Basic syntax

bash

```bash
grep [OPTIONS] PATTERN [FILE...]
```

By default, `grep` searches for `PATTERN` (a **basic regular expression** by default) in each `FILE`, printing matching lines.

bash

```bash
grep "error" app.log
```

### Key options

| Option         | Purpose                                                                    |
| -------------- | -------------------------------------------------------------------------- |
| `-i`           | Case-insensitive match                                                     |
| `-v`           | Invert match (show non-matching lines)                                     |
| `-r` / `-R`    | Recursive search through directories (`-R` follows symlinks, `-r` doesn't) |
| `-n`           | Show line numbers                                                          |
| `-c`           | Count matching lines instead of printing them                              |
| `-l`           | Print only filenames that contain a match                                  |
| `-L`           | Print only filenames that do **not** contain a match                       |
| `-o`           | Print only the matched portion, not the whole line                         |
| `-w`           | Match whole words only                                                     |
| `-x`           | Match whole lines only                                                     |
| `-E`           | Extended regex (ERE) — same as `egrep`                                     |
| `-F`           | Fixed string, no regex interpretation — same as `fgrep`                    |
| `-P`           | Perl-compatible regex (PCRE) — supports lookahead/lookbehind               |
| `-A N`         | Show N lines **after** each match                                          |
| `-B N`         | Show N lines **before** each match                                         |
| `-C N`         | Show N lines of context on both sides                                      |
| `-e PATTERN`   | Specify multiple patterns (OR logic)                                       |
| `-f FILE`      | Read patterns from a file, one per line                                    |
| `-q`           | Quiet — no output, only exit status (for scripting)                        |
| `-Z`           | Output NUL-separated filenames (pairs with `xargs -0`)                     |
| `--color=auto` | Highlight matches in output                                                |
| `-m N`         | Stop after N matches                                                       |
| `-a`           | Treat binary files as text                                                 |
| `-s`           | Suppress error messages (e.g. missing files)                               |

When you work with files, `grep` will always specify the file name. If you want to work with all files in a directory, use `-r` and specify a directory file. 

### Regex flavor note

Plain `grep` uses **BRE** (Basic Regular Expressions) — `+`, `?`, `|`, `()` need to be escaped (`\+`, `\?`, `\|`, `\(\)`). `grep -E` switches to **ERE**, where those are unescaped, matching what you'd write in `awk`. `grep -P` gives you full PCRE — lookaheads, non-greedy, `\d`, `\s`, etc.

bash

```bash
grep 'colou\?r' file.txt      # BRE: ? must be escaped
grep -E 'colou?r' file.txt    # ERE: ? is native
```

### Use cases

#### 1. Basic search with line numbers

bash

```bash
grep -n "TODO" main.cpp
```

```
42:// TODO: handle edge case for empty input
```

Standard first move when scanning your own code for leftover markers before a submission.

#### 2. Case-insensitive search

bash

```bash
grep -i "error" app.log
```

Catches `Error`, `ERROR`, `error` all at once — useful since log severity casing isn't always consistent across tools.

#### 3. Invert match — filter out noise

bash

```bash
grep -v "^#" config.conf
```

Strips comment lines from a config file, showing only active settings. Combine with another `grep -v "^$"` to also drop blank lines:

bash

```bash
grep -v "^#" config.conf | grep -v "^$"
```

#### 4. Recursive search across a project

bash

```bash
grep -rn "TODO" ~/projects/myapp/
```

Finds every TODO across an entire codebase with file+line references, e.g. auditing your Subject Allocation System or the SRE-agent project before a cleanup pass.

#### 5. Count matches per file

bash

```bash
grep -c "ERROR" *.log
```

```
access.log:0
auth.log:12
kern.log:3
```

Quick triage across multiple log files to see which one needs attention first.

#### 6. List only filenames with a match (or without)

bash

```bash
grep -rl "import torch" ~/projects/          # files that use torch
grep -rL "import torch" ~/projects/          # files that DON'T use torch
```

Useful for scoping a refactor — e.g. finding every file that imports a library you're about to deprecate.

#### 7. Extract only the matched text with `-o`

bash

```bash
grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' auth.log
```

Pulls out just IP addresses instead of full lines — pairs with `sort | uniq -c` for a quick frequency count of who's hitting your SSH log:

bash

```bash
grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' auth.log | sort | uniq -c | sort -rn
```

#### 8. Whole-word matching (avoid partial matches)

bash

```bash
grep -w "cat" file.txt
```

Matches `cat` but not `category` or `concatenate` — important when grepping for short variable/function names in code:

bash

```bash
grep -rnw "n" *.cpp    # find lone variable 'n', not part of longer identifiers
```

#### 9. Multiple patterns at once (OR logic)

bash

```bash
grep -E "ERROR|WARN|CRITICAL" app.log
# or equivalently:
grep -e "ERROR" -e "WARN" -e "CRITICAL" app.log
```

Catches any of several severity levels in one pass instead of chaining multiple greps.

#### 10. Search patterns from a file

bash

```bash
grep -f banned_words.txt comments.txt
```

Handy when the pattern list is long or dynamically generated — e.g. checking submitted code against a list of banned function names for a lab (`system`, `exec`, `gets`).

#### 11. Show context around a match

bash

```bash
grep -A 3 -B 1 "Segmentation fault" crash.log
```

Shows 1 line before and 3 lines after each match — critical for actually understanding a crash log instead of just seeing the fault line in isolation.

#### 12. Fixed-string search (no regex interpretation)

bash

```bash
grep -F "a.b*c[1]" weird_filenames.txt
```

Without `-F`, characters like `.`, `*`, `[]` would be interpreted as regex metacharacters. Use `-F` whenever the search term itself contains regex-special characters you want treated literally (paths, version strings like `1.2.3`).

#### 13. PCRE lookahead/lookbehind

bash

```bash
grep -P '(?<=user=)\w+' access.log
```

Extracts the username following `user=` without including `user=` itself — something BRE/ERE can't do natively. Useful for parsing structured log formats.

#### 14. Silent mode for scripting (exit-status only)

bash

```bash
if grep -q "^root:" /etc/passwd; then
    echo "root user exists"
fi
```

No output printed — `grep -q` is purely for the `$?` exit status (0 = found, 1 = not found), the standard pattern for conditional logic in bash scripts.

#### 15. Whole-line matching

bash

```bash
grep -x "192.168.1.1" ip_list.txt
```

Matches only if the _entire line_ equals the pattern — avoids matching `192.168.1.100` when searching for `192.168.1.1`.

#### 16. Binary files as text

bash

```bash
grep -a "GET /admin" access.log
```

Forces grep to treat a file as text even if it looks binary (e.g. contains non-UTF8 bytes from a corrupted log or mixed encoding) — otherwise grep just prints `binary file matches` without showing the line.

#### 17. Combine with `find` for filtered recursive search

bash

```bash
find . -name "*.cpp" -exec grep -l "TODO" {} \;
```

More precise than `grep -r` when you only want to search specific file types — though `grep -r --include="*.cpp"` does the same thing more efficiently:

bash

```bash
grep -rn --include="*.cpp" "TODO" .
```

#### 18. Stop after N matches

bash

```bash
grep -m 5 "ERROR" huge_log.txt
```

Stops scanning after 5 matches — useful on very large files where you just need a sample, not an exhaustive scan.

#### 19. NUL-separated output for safe piping

bash

```bash
grep -rlZ "deprecated_function" . | xargs -0 sed -i 's/deprecated_function/new_function/g'
```

`-Z` + `xargs -0` handles filenames with spaces/newlines safely — the standard "find files matching X, then act on them" pattern without breaking on weird filenames.

#### 20. Highlight matches for readability

bash

```bash
grep --color=auto -n "warning" build.log
```

Most distros alias `grep` to include this by default, but explicit is better when scripting or piping through something that might strip color otherwise.


Print the lines in a file which contain a word:

```bash
grep -w "the"
```


