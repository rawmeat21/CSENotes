# Bash Tutorial Part 2: Everything Else (Fundamentals -> Advanced Shell Mechanics)

This fills every gap from Part 1: file basics, permissions, IFS, substitutions/expansions,
globbing, brace expansion, printf/date, regex, special variables, traps/signals, named pipes,
terminal control, shell customization, pitfalls, scheduling (cron/at), and forkbombs.

---

## 1. Terminal basics / file manipulation

```bash
pwd                     # print working directory
ls -la                   # list all files incl. hidden, long format
cd /path/to/dir            # change directory
cd -                        # go to previous directory
cd ~                          # go home
mkdir -p a/b/c                  # make dirs, -p creates parents as needed
touch file.txt                    # create empty file / update timestamp
cp src.txt dst.txt                  # copy
cp -r srcdir dstdir                   # copy recursively
mv old.txt new.txt                      # move / rename
rm file.txt                               # delete file
rm -rf dir/                                 # delete dir recursively, force (DANGEROUS)
```

### Hidden files
Any file/dir starting with `.` is hidden from plain `ls`.
```bash
ls -a          # show hidden files (.bashrc, .git, etc.)
```

### man pages
```bash
man ls           # full manual for a command
ls --help          # short built-in help
type cd              # tells you if something is a builtin, alias, or binary
which python3          # path to the executable
```

### Paging files
```bash
less file.txt      # scrollable pager (q to quit, / to search)
more file.txt        # simpler pager, less common now
head -n 20 file.txt     # first 20 lines
tail -n 20 file.txt       # last 20 lines
tail -f log.txt             # follow file live (like tail -f for logs)
```

### Searching in files
```bash
grep "pattern" file.txt
find . -name "*.cpp"          # find files by name pattern
find . -type f -mtime -1        # files modified in last 1 day
find . -type d                    # find only directories
find . -name "*.o" -delete          # find and delete matches
find . -name "*.txt" -exec cat {} \;  # find and run a command on each match
```

### File permissions (`chmod`, `chown`)
```bash
ls -l file.txt      # shows: -rwxr-xr-x  owner group ...
# r=read(4) w=write(2) x=execute(1), 3 groups: owner/group/others

chmod 755 script.sh     # owner=rwx, group=rx, others=rx
chmod +x script.sh        # add execute permission to all
chmod u+x,g-w file.txt      # symbolic form: user +x, group -w
chown user:group file.txt     # change owner/group (often needs sudo)
```

---

## 2. Variables and expressions (`let`, datatypes)

Bash has exactly ONE real datatype: **strings**. Everything else (integers, arrays) is
a special *attribute* attached to a variable via `declare`, not a separate type system
like C++'s `int`/`float`/`char`.

```bash
declare -i n=5        # integer attribute: auto-evaluates arithmetic on assignment
n="3+4"                  # with -i set, this becomes 7, not the string "3+4"

declare -a arr=(1 2 3)     # indexed array
declare -A map=()            # associative array
declare -r CONST=100           # readonly
declare -x EXPORTED=1            # same as `export EXPORTED=1`
declare -l lower="ABC"             # forces lowercase -> "abc"
declare -u upper="abc"               # forces uppercase -> "ABC"

declare -p n           # print a variable's declaration/attributes (debugging)
```

`let` — alternate arithmetic syntax (older, `(( ))` is preferred):
```bash
let "x = 5 + 3"
let "y++"
let "z = x * y"
```
Rule of thumb: use `(( ))` for arithmetic statements/conditions, `$(( ))` when you need
the numeric *value* substituted into a string or assignment, and avoid `let` in new code.

---

## 3. Mixing commands with code

Bash scripts freely mix control-flow with raw shell commands — there's no boundary
between "code" and "the shell" like there is between C++ and `system()`.

```bash
files=$(ls *.txt)              # command output captured into a variable
count=$(ls *.txt | wc -l)        # pipe chains work inside substitution too

if grep -q "ERROR" log.txt; then     # a COMMAND's exit code IS the condition
    echo "found an error"
fi

for f in $(find . -name "*.cpp"); do   # loop directly over command output
    echo "compiling $f"
    g++ -c "$f"                          # then call a real program
done

result=$(echo "5 + 3" | bc)                # shell out to another language mid-script
```
This "combine builtins + external programs + control flow" style is bash's core idiom —
you are always just gluing programs together, not writing an isolated algorithm.

---

## 4. Recursion (functions calling themselves)

Already shown in Part 1, restated with a second example — recursion works exactly like C++,
bounded by bash's function call stack (default limit ~1000 depth, `ulimit -s` affects it).

```bash
fib() {
    local n=$1
    if (( n <= 1 )); then
        echo "$n"
        return
    fi
    local a=$(fib $((n - 1)))
    local b=$(fib $((n - 2)))
    echo $((a + b))
}
fib 10   # 55
```
Note: every recursive call here spawns a subshell (via `$(...)`), which is slow. For
performance-sensitive recursion, prefer iterative loops in bash, or drop to Python/C++.

---

## 5. IFS (Internal Field Separator)

IFS controls how bash splits words during expansion, `read`, and `for` loops. Default is
space/tab/newline.

```bash
echo "$IFS" | cat -A     # see default IFS (space, tab, newline)

# split a CSV line
line="a,b,c"
IFS=',' read -ra parts <<< "$line"
echo "${parts[1]}"        # b

# temporarily change IFS just for one command (safer, doesn't leak)
IFS=':' read -ra path_parts <<< "$PATH"

# loop over comma-separated values
IFS=','
for item in $line; do echo "$item"; done
unset IFS      # always reset it back, or scope it as above
```

---

## 6. Substitution and Expansion (ALL kinds)

Bash performs expansions in this order: brace -> tilde -> parameter/command/arithmetic ->
word splitting -> pathname (glob).

### 6.1 Command substitution
```bash
now=$(date)          # preferred modern syntax
now=`date`             # old backtick syntax, avoid (can't nest easily)
echo "Today: $(date +%Y-%m-%d)"
```

### 6.2 Arithmetic expansion
```bash
echo "$((3 + 4 * 2))"     # 11
x=$((10 % 3))
```

### 6.3 Parameter expansion (the big one — string ops live here)
```bash
var="hello"
echo "${var}"              # basic
echo "${var:-default}"       # use default if var is unset/empty (doesn't assign)
echo "${var:=default}"         # use default AND assign it to var
echo "${var:+alt}"                # if var IS set, substitute "alt" instead
echo "${var:?error msg}"            # error out with message if var is unset

echo "${#var}"          # length -> 5
echo "${var:1:3}"         # substring -> ell
echo "${var/l/L}"           # replace first "l" -> heLlo
echo "${var//l/L}"            # replace all "l" -> heLLo
echo "${var^}"                  # capitalize first letter -> Hello
echo "${var^^}"                    # all uppercase -> HELLO
echo "${var,,}"                      # all lowercase -> hello
echo "${var#h*l}"                       # strip shortest matching prefix -> lo
echo "${var##h*l}"                        # strip longest matching prefix -> o
echo "${var%l*}"                            # strip shortest matching suffix -> hel
echo "${var%%l*}"                             # strip longest matching suffix -> he

echo "${!var}"          # indirect expansion: dereference the variable NAMED by $var
```

### 6.4 Array expansion
```bash
arr=(a b c d)
echo "${arr[@]}"          # all elements, each a separate word
echo "${arr[*]}"            # all elements, as ONE joined string (uses $IFS)
echo "${arr[@]:1:2}"          # slice starting at index 1, length 2 -> b c
echo "${#arr[@]}"                # number of elements
echo "${!arr[@]}"                  # all indices -> 0 1 2 3
echo "${arr[-1]}"                    # last element (bash 4.3+) -> d
```

### 6.5 Process substitution — treats a command's output as if it were a FILE
```bash
diff <(sort file1.txt) <(sort file2.txt)     # compare two commands' output directly
while read -r line; do echo "$line"; done < <(grep "ERROR" log.txt)
paste <(ls dir1) <(ls dir2)                    # feed two streams side by side
```
`<(...)` = readable "fake file" of a command's stdout. `>(...)` = writable "fake file" that
feeds INTO a command's stdin. Different from `$()` — that captures output as text, this
gives you a file descriptor.

### 6.6 Brace expansion — generates literal strings BEFORE the command even runs
```bash
echo file{1,2,3}.txt        # -> file1.txt file2.txt file3.txt
echo {a,b,c}_test               # -> a_test b_test c_test
mkdir -p project/{src,bin,docs}   # create multiple dirs in one call
echo {1..5}                         # numeric range -> 1 2 3 4 5
echo {5..1}                           # reversed -> 5 4 3 2 1
echo {01..10}                           # zero-padded -> 01 02 ... 10
echo {0..20..5}                           # step-by-5 -> 0 5 10 15 20
echo {a..e}                                 # letter range -> a b c d e
```
Key distinction: brace expansion is pure TEXT GENERATION, it does NOT check whether the
files exist. Globbing (below) DOES check the filesystem.

### 6.7 Tilde expansion
```bash
echo ~          # home dir
echo ~user        # another user's home dir
echo ~+             # current working dir (same as $PWD)
```

---

## 7. Globbing (pattern matching against real files)

### Basic globs
```bash
ls *.txt          # any files ending in .txt
ls file?.txt         # ? = exactly one character
ls [abc]*.txt           # matches files starting with a, b, or c
ls [^abc]*.txt             # NOT starting with a, b, or c
ls file[0-9].txt              # character range
```

### Extended globbing (must enable first)
```bash
shopt -s extglob

ls !(*.txt)          # everything EXCEPT .txt files
ls @(*.c|*.cpp)         # matches EITHER pattern (like an OR)
ls +(*.txt)                # one or more repetitions of pattern
ls *(*.txt)                   # zero or more repetitions
ls ?(*.txt)                      # zero or one repetition
```

### Glob shell options
```bash
shopt -s globstar     # enables ** for recursive matching
ls **/*.cpp              # matches .cpp files in ALL subdirectories recursively

shopt -s nullglob        # unmatched glob expands to nothing instead of literal pattern
shopt -s nocaseglob         # case-insensitive globbing
shopt -s dotglob               # include hidden (dotfiles) in glob matches
```

---

## 8. printf and Date formatting

### printf (much more control than `echo`)
```bash
printf "%s\n" "hello"           # string
printf "%d\n" 42                  # integer
printf "%5d\n" 42                   # right-align width 5 -> "   42"
printf "%-5d|\n" 42                   # left-align width 5 -> "42   |"
printf "%05d\n" 42                      # zero-pad -> 00042
printf "%.2f\n" 3.14159                   # 2 decimal places -> 3.14
printf "%x\n" 255                           # hex -> ff
printf "%o\n" 8                               # octal -> 10
printf "%c\n" 65                                # NOT ascii char by default in bash printf
printf "%s is %d years old\n" "Qing" 20           # multiple args
printf "%s\n" "${arr[@]}"                            # print each array element on own line
```

### Date formatting
```bash
date                              # current date/time, default format
date +%Y-%m-%d                      # 2026-08-07
date +%H:%M:%S                        # 14:30:00
date +"%A, %B %d, %Y"                   # Friday, August 07, 2026
date -d "2 days ago" +%Y-%m-%d            # relative date math (GNU date)
date -d "next monday" +%Y-%m-%d             # next monday's date
date +%s                                      # unix timestamp (seconds since epoch)
date -d @1700000000 +%Y-%m-%d                   # convert timestamp back to date
```

---

## 9. Regular expressions

```bash
if [[ "hello123" =~ ^[a-z]+[0-9]+$ ]]; then
    echo "matched"
fi

# capture groups from a =~ match are stored in BASH_REMATCH
if [[ "2026-08-07" =~ ^([0-9]{4})-([0-9]{2})-([0-9]{2})$ ]]; then
    echo "year: ${BASH_REMATCH[1]}"
    echo "month: ${BASH_REMATCH[2]}"
    echo "day: ${BASH_REMATCH[3]}"
fi

grep -E "^[0-9]+$" file.txt      # extended regex with grep
sed -E 's/([0-9]+)-([0-9]+)/\2-\1/' file.txt   # regex with capture-group backreferences
```

---

## 10. mapfile / readarray

```bash
mapfile -t lines < file.txt        # read all lines of a file into an array, -t strips newlines
echo "${lines[0]}"
echo "${#lines[@]}"                  # number of lines

mapfile -t results < <(ls *.txt)       # combine with process substitution
```

---

## 11. Brackets vs Test (`[[`, `[`, `test`, `((`)

All four evaluate conditions but behave differently:

| Form | Notes |
|---|---|
| `test expr` | POSIX builtin, oldest form |
| `[ expr ]` | identical to `test`, just bracket syntax; needs quoting discipline, supports `-a`/`-o` |
| `[[ expr ]]` | bash keyword (not a command); safer, supports `&&` `\|\|` `=~` `<` `>` directly, no word-splitting surprises |
| `(( expr ))` | arithmetic context; returns exit-status 0 (true) if the numeric result is non-zero |

```bash
[ -z "$var" ] && echo "empty"        # POSIX-safe, portable to sh
[[ -z $var ]] && echo "empty"          # bash-only, allows unquoted var safely
(( 5 > 3 )) && echo "true"               # arithmetic truthiness
```
Prefer `[[ ]]` and `(( ))` in bash-specific scripts; use `[ ]` only if targeting POSIX `sh`.

---

## 12. Special strings / special variables

```bash
$0        # script name
$1..$9    # positional args
$#        # arg count
$@        # all args, separately quoted
$*        # all args, one string
$?        # exit code of last command
$$        # PID of current shell
$!        # PID of last background job
$-        # currently set shell flags (e.g. himBH)
$_        # last argument of previous command
$RANDOM   # random integer 0-32767
$SECONDS  # seconds since shell started
$LINENO   # current line number in script (useful in trap/debug)
$BASH_SOURCE  # path of the script currently being sourced/run
```

---

## 13. Trap signals (full signal handling, not just ERR/EXIT)

```bash
trap 'echo "Ctrl-C caught, exiting"; exit 1' SIGINT   # handle Ctrl+C
trap 'echo "terminated"; cleanup' SIGTERM               # handle kill signal
trap 'echo "exiting, cleaning up"; rm -f /tmp/lock' EXIT  # runs on ANY exit path
trap 'echo "err at line $LINENO"' ERR                       # runs when a command fails
trap '' SIGINT                                                  # ignore Ctrl+C entirely
trap - SIGINT                                                     # restore default behavior

trap 'echo "reload config"' SIGHUP    # common convention: SIGHUP = "reload"
kill -l                                  # list all available signal names
```

---

## 14. Named pipes (FIFOs) — inter-process communication

```bash
mkfifo mypipe                # create a named pipe (special file on disk)

# terminal A (writer):
echo "hello" > mypipe          # blocks until a reader connects

# terminal B (reader):
cat < mypipe                     # receives "hello"

rm mypipe                          # clean up when done
```
Useful for streaming data between two independently-running scripts/processes without
a temp file.

---

## 15. Pipe status, Timing, Sourcing, Subshells

### Pipe status (`$?` only shows the LAST command in a pipe — use PIPESTATUS for all)
```bash
false | true
echo "$?"                # 0 (only reflects `true`, the last command — misleading!)

false | true
echo "${PIPESTATUS[@]}"    # 1 0  -> full exit status of EVERY stage in the pipeline
```

### Timing commands
```bash
time ./script.sh          # real/user/sys time for a whole command
time (for i in {1..1000000}; do :; done)   # time a block

start=$SECONDS
sleep 2
echo "elapsed: $((SECONDS - start))s"     # manual timing using $SECONDS

start_ns=$(date +%s%N)
sleep 1
end_ns=$(date +%s%N)
echo "elapsed ms: $(( (end_ns - start_ns) / 1000000 ))"   # high-precision timing
```

### Sourcing code (like `#include`, but for shell — runs in CURRENT shell, not a subshell)
```bash
# utils.sh
greet() { echo "hi $1"; }

# main.sh
source utils.sh     # or: . utils.sh
greet "Qing"           # now available, because it ran in the SAME shell context
```
Sourcing shares variables/functions both ways with the calling shell; running a script
normally (`./script.sh`) does NOT — it executes in an isolated subshell/process.

### Curlies vs Parens (grouping commands)
```bash
{ echo "a"; echo "b"; }      # braces: run in CURRENT shell (note required spaces + trailing ;)
(echo "a"; echo "b")           # parens: run in a SUBSHELL (variable changes don't leak out)

x=1
( x=2 )              # subshell — this change is local to the subshell
echo "$x"              # still 1

x=1
{ x=2; }             # braces — runs in current shell
echo "$x"               # 2
```

### Return vs Output (the core mental model)
- `return N` (or exiting a script) sets the **exit status**, an integer 0-255, checked via
  `$?` or `if command; then`. This is like a boolean/error code, NOT a return value.
- `echo`/`printf` writes to **stdout**, captured via `$(function_call)`. This is how you
  actually "return a value" from a bash function, since bash functions can't `return` arbitrary
  data types like C++.
```bash
get_status() { return 1; }             # exit code only
get_value()  { echo "42"; }              # actual data, must be captured

get_status; echo "$?"      # 1
val=$(get_value); echo "$val"   # 42
```

---

## 16. Color output, cursor commands, TTY detection

### Color output (ANSI escape codes)
```bash
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m'   # No Color, always reset after
echo -e "${RED}error${NC}: ${GREEN}ok${NC}"

tput setaf 1     # red foreground (more portable than raw escape codes)
tput sgr0          # reset
```

### Cursor commands (via `tput`)
```bash
tput cup 5 10    # move cursor to row 5, col 10
tput civis         # hide cursor
tput cnorm            # show cursor again
tput clear               # clear screen
tput cols                  # terminal width
tput lines                   # terminal height
```

### Is a TTY? (detect interactive terminal vs pipe/redirect)
```bash
if [[ -t 1 ]]; then
    echo "stdout is a terminal (interactive)"
else
    echo "stdout is redirected/piped"
fi
# -t 0 checks stdin, -t 1 checks stdout, -t 2 checks stderr
```
Common use: only print colors when output is a real terminal, plain text otherwise
(since piping to a file shouldn't include escape codes).

---

## 17. Shell customization: PS1, .bashrc, readline shortcuts

### PS1 (prompt string)
```bash
PS1='\u@\h:\w\$ '     # user@host:dir$   — the default-ish style
# \u = username, \h = hostname, \w = full path, \W = basename of path
# \t = time, \d = date, \$ = # if root else $

PS1='\[\033[32m\]\u\[\033[0m\]:\w\$ '   # green username
```

### .bashrc
Runs on every new interactive shell. Put aliases, exports, PS1, functions here.
```bash
# ~/.bashrc
export EDITOR=vim
alias ll='ls -la'
alias gs='git status'
source ~/.bash_aliases   # split into separate files if it grows large
```
Reload without restarting terminal: `source ~/.bashrc`

### Readline shortcuts (built into any bash prompt)
| Shortcut | Action |
|---|---|
| `Ctrl+A` | move to start of line |
| `Ctrl+E` | move to end of line |
| `Ctrl+U` | delete from cursor to start of line |
| `Ctrl+K` | delete from cursor to end of line |
| `Ctrl+W` | delete previous word |
| `Ctrl+R` | reverse search command history |
| `Ctrl+L` | clear screen |
| `Alt+.`  | insert last argument of previous command |
| `Ctrl+_` | undo |

---

## 18. Aliases with arguments (and why plain aliases can't take args)

```bash
alias ll='ls -la'          # simple alias, no arguments possible directly
```
`alias` is pure text substitution — you CAN'T write `alias greet='echo hi $1'` and expect
`$1` to work. Use a function instead when you need arguments:
```bash
greet() { echo "hi $1"; }     # this is the correct replacement for "alias with args"
```

---

## 19. Common pitfalls

### Pitfall: parsing `ls` output
```bash
for f in $(ls); do ...; done      # BREAKS on filenames with spaces/newlines/globs
for f in *; do ...; done             # correct: use globbing directly, not ls
```
Never parse `ls` in scripts — use globs or `find ... -print0` with `read -d ''`.

### Pitfall: string length vs byte length
```bash
s="héllo"
echo "${#s}"                    # character count (locale-dependent)
echo "$s" | wc -c                 # BYTE count, differs for multi-byte UTF-8 chars
```

### Pitfall: unquoted variables
```bash
file="my file.txt"
rm $file       # BREAKS — expands to `rm my file.txt` (two separate args!)
rm "$file"       # correct
```

---

## 20. Forkbomb (why it's dangerous — DO NOT RUN)

A forkbomb is a function that calls itself twice in the background, recursively, with no
base case — each call spawns two more child processes exponentially until the system runs
out of process slots/memory and becomes unresponsive. It is commonly shown in bash courses
purely as a cautionary example of unbounded recursion + `&` backgrounding, never to be
executed on a real machine (including your own, since it can force a hard reboot).
The lesson: always have a base case in recursion, and never background a process inside
its own recursive call without a hard limit.

---

## 21. cron and at (scheduling — not in your syllabus screenshot but commonly paired with it)

### cron — run a command on a recurring schedule
```bash
crontab -e             # edit your personal crontab in $EDITOR
crontab -l                # list current cron jobs
```
Format: `minute hour day-of-month month day-of-week command`
```
0 9 * * *        /home/qing/backup.sh        # every day at 9:00 AM
*/15 * * * *      /home/qing/check.sh           # every 15 minutes
0 0 1 * *          /home/qing/monthly.sh           # midnight, 1st of every month
30 18 * * 1-5        /home/qing/weekday_report.sh       # 6:30 PM, Mon-Fri only
```

### at — run a command ONCE at a specific future time
```bash
echo "./backup.sh" | at 22:00              # run once tonight at 10 PM
at now + 10 minutes <<< "./notify.sh"         # run once, 10 minutes from now
atq                                              # list pending at jobs
atrm 3                                             # cancel job number 3
```
`cron` = recurring schedule (like a repeating timer), `at` = one-shot future execution
(like `setTimeout` for the shell).

---

This, combined with Part 1, covers the entire syllabus: fundamentals, scripting constructs,
arrays, all expansion/substitution types, globbing, text tools, process control, signals,
IPC, terminal/UI control, shell customization, common pitfalls, and scheduling.
