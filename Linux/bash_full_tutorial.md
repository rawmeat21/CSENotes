# Bash: Full Tutorial for C++ Developers

Goal: every C++ construct you use, mapped to Bash. Run scripts with:
```bash
chmod +x script.sh
./script.sh
# or
bash script.sh
```
Always start scripts with:
```bash
#!/usr/bin/env bash
set -euo pipefail   # exit on error, undefined var error, pipe fail — use like assert-everywhere
```

---

## 1. Variables

```bash
name="Qing"          # NO spaces around =
age=20
readonly PI=3.14159  # const

echo "$name is $age" # use quotes + $var. always quote unless you know why not.
echo "${name}_suffix" # braces to disambiguate

unset age
```

Bash has **no types**. Everything is a string unless you force arithmetic context.
No `int x;` declarations needed — `declare` is optional but useful:
```bash
declare -i n=5     # integer-typed variable (auto arithmetic on assignment)
declare -r K=10     # readonly
declare -a arr      # indexed array
declare -A map      # associative array (hash map)
```

Environment / global export (like extern-ish):
```bash
export PATH="$PATH:/usr/local/bin"
```

---

## 2. Arithmetic (replaces `int`, `+ - * / % ++ --`, math ops)

```bash
a=5
b=2

echo $((a + b))    # 7
echo $((a - b))    # 3
echo $((a * b))    # 10
echo $((a / b))    # 2   -> integer division only!
echo $((a % b))    # 1
echo $((a ** b))   # 25  power

((a++))            # increment
((a--))
((a += 3))
let "a = a + 1"    # alt syntax, avoid, prefer (( ))

# Floating point — bash can't do float math natively, use bc or awk
echo "5 / 2" | bc -l          # 2.50000000000000000000
awk "BEGIN{print 5/2}"        # 2.5
```

Comparison inside arithmetic context:
```bash
if (( a > b )); then echo "a bigger"; fi
```

---

## 3. Conditionals (`if / else if / else`, `switch`)

```bash
if [[ $a -gt $b ]]; then
    echo "a > b"
elif [[ $a -eq $b ]]; then
    echo "equal"
else
    echo "a < b"
fi
```

`[[ ]]` (preferred, bash-only, supports `&&` `||` `=~`) vs `[ ]` (POSIX `test`, use `-a -o`):

Numeric operators (inside `[[ ]]` or `[ ]`):
| C++ | Bash test |
|---|---|
| `==` | `-eq` |
| `!=` | `-ne` |
| `>`  | `-gt` |
| `<`  | `-lt` |
| `>=` | `-ge` |
| `<=` | `-le` |

String operators:
| Meaning | Bash |
|---|---|
| equal | `[[ "$a" == "$b" ]]` |
| not equal | `[[ "$a" != "$b" ]]` |
| empty | `[[ -z "$a" ]]` |
| non-empty | `[[ -n "$a" ]]` |
| regex match | `[[ "$a" =~ ^[0-9]+$ ]]` |
| glob match | `[[ "$a" == foo* ]]` |

Logical: `&&` (and), `||` (or), `!` (not)
```bash
if [[ $a -gt 0 && $b -gt 0 ]]; then echo "both positive"; fi
```

Ternary-like (no real ternary in bash):
```bash
result=$(( a > b ? a : b ))     # works ONLY inside arithmetic context
msg=$([[ $a -gt $b ]] && echo "yes" || echo "no")
```

`switch` -> `case`:
```bash
case "$1" in
    start)
        echo "starting"
        ;;
    stop|halt)
        echo "stopping"
        ;;
    *)
        echo "unknown"
        ;;
esac
```

---

## 4. Loops (`for`, `while`, `do-while`)

C-style for:
```bash
for ((i = 0; i < 10; i++)); do
    echo "$i"
done
```

Range-based / iterate list:
```bash
for x in 1 2 3 4 5; do echo "$x"; done
for x in {1..10}; do echo "$x"; done
for x in {0..20..2}; do echo "$x"; done   # step 2
for f in *.txt; do echo "$f"; done         # iterate files
for word in "${arr[@]}"; do echo "$word"; done  # iterate array
```

while:
```bash
i=0
while (( i < 10 )); do
    echo "$i"
    ((i++))
done
```

do-while (bash has no native do-while; simulate):
```bash
i=0
while true; do
    echo "$i"
    ((i++))
    (( i < 10 )) || break
done
```

`break` and `continue` work exactly like C++. `break 2` breaks out of 2 nested loops.

until (loops while condition is FALSE):
```bash
i=0
until (( i >= 5 )); do
    echo "$i"
    ((i++))
done
```

---

## 5. Functions

```bash
# C++: int add(int a, int b) { return a + b; }
add() {
    local a=$1
    local b=$2
    echo $((a + b))     # "return" a value by printing it, capture with $()
}

result=$(add 3 4)
echo "$result"          # 7
```

Real exit-code return (0-255 only, like C++ `return` from `main`):
```bash
is_even() {
    local n=$1
    if (( n % 2 == 0 )); then
        return 0   # success/true
    else
        return 1   # failure/false
    fi
}

if is_even 4; then echo "even"; fi
```

Variadic args / all args:
```bash
show_all() {
    echo "count: $#"
    echo "all: $@"
    for arg in "$@"; do echo "- $arg"; done
}
show_all a b c
```

`local` = stack-local variable, always use it inside functions to avoid polluting globals.

Recursion works normally:
```bash
factorial() {
    local n=$1
    if (( n <= 1 )); then
        echo 1
    else
        local sub=$(factorial $((n - 1)))
        echo $((n * sub))
    fi
}
factorial 5   # 120
```

Function pointers / callbacks: pass function name as string, call with `"$fn"`:
```bash
apply() {
    local fn=$1
    local val=$2
    "$fn" "$val"
}
double() { echo $(( $1 * 2 )); }
apply double 5   # 10
```

---

## 6. Arrays (replaces `std::vector`, `int arr[]`)

Indexed array:
```bash
arr=(10 20 30 40)
arr[4]=50

echo "${arr[0]}"        # 10
echo "${arr[@]}"        # all elements
echo "${#arr[@]}"        # length -> 5
echo "${arr[@]:1:2}"     # slice: index 1, length 2 -> 20 30

arr+=(60)                # push_back
unset 'arr[0]'            # erase index 0

for v in "${arr[@]}"; do echo "$v"; done
```

Loop with index:
```bash
for i in "${!arr[@]}"; do
    echo "index $i -> ${arr[$i]}"
done
```

Sort a copy (replaces `std::sort`):
```bash
sorted=($(printf '%s\n' "${arr[@]}" | sort -n))
```

Associative array (replaces `std::map` / `std::unordered_map`):
```bash
declare -A m
m["apple"]=3
m["banana"]=5

echo "${m[apple]}"          # 3
echo "${!m[@]}"              # all keys
echo "${m[@]}"                # all values

for k in "${!m[@]}"; do
    echo "$k -> ${m[$k]}"
done

if [[ -v m[apple] ]]; then echo "key exists"; fi
unset 'm[apple]'
```

2D array (bash has none natively) — simulate with a 1D array + index math, or with delimited strings, or associative array keyed "row,col":
```bash
declare -A grid
rows=3; cols=3
for ((r=0; r<rows; r++)); do
    for ((c=0; c<cols; c++)); do
        grid[$r,$c]=0
    done
done
grid[1,1]=5
echo "${grid[1,1]}"
```

---

## 7. Strings (replaces `std::string` methods)

```bash
s="Hello World"

echo "${#s}"              # length -> 11
echo "${s:0:5}"            # substr(0,5) -> Hello
echo "${s:6}"               # substr from 6 -> World
echo "${s^^}"                # uppercase -> HELLO WORLD
echo "${s,,}"                 # lowercase -> hello world
echo "${s/World/Bash}"         # replace first match -> Hello Bash
echo "${s//o/0}"                # replace ALL -> Hell0 W0rld
echo "${s#Hello }"                # strip prefix -> World
echo "${s%World}"                  # strip suffix -> Hello

concat="$s and more"          # concatenation
concat+=" text"                # append (+=)

# split into array (like split() on delimiter)
IFS=',' read -ra parts <<< "a,b,c"
for p in "${parts[@]}"; do echo "$p"; done

# find substring index — no direct built-in, use expr or bash-fu:
str="hello world"
sub="world"
index=$(expr index "$str" "$sub")   # approximate, char-class based, careful

# check contains
if [[ "$str" == *"world"* ]]; then echo "contains"; fi

# reverse a string
echo "$s" | rev

# trim whitespace
trimmed="$(echo "$s" | xargs)"

# string to number and back — implicit, bash auto-coerces in arithmetic context
n="42"
echo $((n + 1))    # 43

# number to string — already a string, just use it
```

Compare strings lexically:
```bash
if [[ "abc" < "abd" ]]; then echo "less"; fi   # inside [[ ]], < and > do lexical compare
```

---

## 8. Input / Output (`cin`, `cout`, `cerr`, file streams)

```bash
echo "plain output"                     # cout <<
echo -n "no newline"                    # cout << without endl
printf "%s is %d\n" "$name" "$age"       # printf, like C printf
echo "error" >&2                          # cerr <<

read -p "Enter name: " name               # cin >> name
read -p "Enter a b: " a b                  # multiple values
read -r line                                # read a whole line safely (raw, no backslash escaping)

# read a number
read -p "num: " n
echo $((n * 2))
```

File I/O (`ifstream` / `ofstream`):
```bash
echo "line1" > file.txt      # write (overwrite/truncate)
echo "line2" >> file.txt      # append

while IFS= read -r line; do    # read file line by line
    echo "$line"
done < file.txt

cat file.txt                    # read whole file to stdout
content=$(<file.txt)             # read whole file into a variable
mapfile -t lines < file.txt       # read all lines into an array
echo "${lines[0]}"
```

Redirection cheat sheet:
| Symbol | Meaning |
|---|---|
| `>` | stdout to file (overwrite) |
| `>>` | stdout to file (append) |
| `<` | read stdin from file |
| `2>` | stderr to file |
| `2>&1` | redirect stderr to same place as stdout |
| `\|` | pipe stdout of one command to stdin of next |
| `<<<` | here-string, feed a variable as stdin |
| `<<EOF ... EOF` | here-doc, multi-line stdin block |

---

## 9. Command-line arguments (`argc`, `argv`, `int main(int argc, char** argv)`)

```bash
echo "$0"      # program name (argv[0])
echo "$1"      # first arg
echo "$2"      # second arg
echo "$#"       # argc-1 (count of args, excludes $0)
echo "$@"        # all args as separate words (use "$@" quoted!)
echo "$*"          # all args as ONE string

shift            # discard $1, shift args left by one
```

Proper flag parsing (replaces `getopt`/manual argv parsing in C++):
```bash
while getopts "n:f:v" opt; do
    case $opt in
        n) name="$OPTARG" ;;
        f) file="$OPTARG" ;;
        v) verbose=true ;;
        *) echo "usage: $0 -n name -f file [-v]"; exit 1 ;;
    esac
done
```
Run as: `./script.sh -n Qing -f data.txt -v`

---

## 10. Exit codes / error handling (`try/catch`, `assert`, `exit()`)

```bash
exit 0        # success
exit 1        # generic failure

echo $?        # last command's exit code (0 = success, else = error, like errno)

command || echo "failed"          # run fallback if command fails (like catch)
command && echo "succeeded"       # run only if success

set -e         # exit script immediately on any error (like unhandled exception crashes)
set -u         # error on undefined variable use
set -o pipefail  # pipe fails if ANY stage fails, not just the last

trap 'echo "error on line $LINENO"' ERR   # crude try/catch: runs on any error
trap 'echo "cleanup"; rm -f tmp.txt' EXIT  # destructor-like cleanup, always runs on exit

# assert-like pattern
[[ -f "config.txt" ]] || { echo "config missing"; exit 1; }
```

---

## 11. Structs / Objects (no OOP in bash — simulate)

Bash has no structs/classes. Common workarounds:

Associative array as a "struct":
```bash
declare -A person
person[name]="Qing"
person[age]=20
echo "${person[name]} is ${person[age]}"
```

Array of "structs" via naming convention (prefix + index):
```bash
p1_name="Qing"; p1_age=20
p2_name="Bob";  p2_age=25
for i in 1 2; do
    n="p${i}_name"; a="p${i}_age"
    echo "${!n} is ${!a}"   # indirect expansion: ${!var} dereferences var-named-var
done
```

For real OOP-like needs, write the logic in Python and call it from bash instead — bash is not the right tool for object-oriented design.

---

## 12. Common algorithms (`std::sort`, `std::find`, `std::max`, `std::min`)

```bash
# max / min of two numbers
max=$(( a > b ? a : b ))
min=$(( a < b ? a : b ))

# max of an array
arr=(5 2 9 1 7)
max=${arr[0]}
for v in "${arr[@]}"; do (( v > max )) && max=$v; done
echo "$max"

# sort array ascending
sorted=($(printf '%s\n' "${arr[@]}" | sort -n))
# sort descending
sorted=($(printf '%s\n' "${arr[@]}" | sort -rn))

# sum of array
sum=0
for v in "${arr[@]}"; do (( sum += v )); done

# find element (linear search)
target=9
found=false
for v in "${arr[@]}"; do
    if [[ "$v" == "$target" ]]; then found=true; break; fi
done

# unique elements
printf '%s\n' "${arr[@]}" | sort -u

# reverse array
reversed=($(printf '%s\n' "${arr[@]}" | tac))
```

---

## 13. Text processing power tools (grep / sed / awk) — bash's real strength over C++ for text

`grep` (search, replaces manual string scanning loops):
```bash
grep "pattern" file.txt              # print matching lines
grep -i "pattern" file.txt            # case-insensitive
grep -v "pattern" file.txt             # invert match
grep -c "pattern" file.txt              # count matches
grep -E "^[0-9]+$" file.txt              # extended regex
grep -r "pattern" ./dir                   # recursive search in directory
```

`sed` (stream editor, find/replace, replaces regex_replace loops):
```bash
sed 's/foo/bar/' file.txt         # replace first occurrence per line
sed 's/foo/bar/g' file.txt         # replace all occurrences
sed -i 's/foo/bar/g' file.txt       # in-place edit
sed -n '2,4p' file.txt                # print lines 2 to 4
sed '/pattern/d' file.txt              # delete matching lines
```

`awk` (field-based text processing, replaces manual tokenizing/parsing loops):
```bash
awk '{print $1}' file.txt              # print 1st column (whitespace-delimited)
awk -F',' '{print $2}' file.txt         # use comma as delimiter
awk '{sum += $1} END {print sum}' file.txt   # sum a column
awk '$3 > 100 {print $0}' file.txt             # filter rows by condition
awk 'BEGIN{print "start"} {print} END{print "done"}' file.txt
```

`cut`, `sort`, `uniq`, `wc`, `tr`:
```bash
cut -d',' -f1,3 file.csv     # extract columns 1 and 3
sort file.txt                 # sort lines
sort -n file.txt                # numeric sort
uniq file.txt                    # remove adjacent duplicate lines
sort file.txt | uniq              # remove all duplicates
wc -l file.txt                     # count lines
wc -w file.txt                      # count words
tr 'a-z' 'A-Z' < file.txt              # translate lowercase to uppercase
```

---

## 14. Processes / system calls (`system()`, `fork()`, `exec()`)

```bash
ls -l                     # run any external program directly, like system("ls -l")
result=$(ls -l)             # capture output
./program arg1 arg2          # run compiled binary directly

command &                     # run in background (like fork + no wait)
wait                            # wait for all background jobs to finish
pid=$!                            # get PID of last background job
kill "$pid"                        # kill a process

command1 | command2                 # pipe, like connecting stdout->stdin between processes
$(command)                           # command substitution, capture output as a string
$(< file.txt)                          # fast way to read a file into a variable
```

---

## 15. Full example: converting a small C++ program

C++:
```cpp
#include <iostream>
#include <vector>
using namespace std;

int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

int main(int argc, char** argv) {
    vector<int> nums = {1, 2, 3, 4, 5};
    int sum = 0;
    for (int x : nums) sum += x;
    cout << "sum=" << sum << endl;
    for (int i = 1; i <= 5; i++)
        cout << i << "! = " << factorial(i) << endl;
    return 0;
}
```

Bash equivalent:
```bash
#!/usr/bin/env bash
set -euo pipefail

factorial() {
    local n=$1
    if (( n <= 1 )); then
        echo 1
    else
        local sub=$(factorial $((n - 1)))
        echo $((n * sub))
    fi
}

main() {
    nums=(1 2 3 4 5)
    sum=0
    for x in "${nums[@]}"; do
        (( sum += x ))
    done
    echo "sum=$sum"

    for ((i = 1; i <= 5; i++)); do
        echo "$i! = $(factorial "$i")"
    done
}

main "$@"
```

---

## 16. Quick reference: C++ -> Bash cheat sheet

| C++ | Bash |
|---|---|
| `int x = 5;` | `x=5` |
| `std::string s = "hi";` | `s="hi"` |
| `std::vector<int> v;` | `declare -a v=()` |
| `std::map<string,int> m;` | `declare -A m=()` |
| `if (a > b)` | `if [[ $a -gt $b ]]; then` |
| `for (int i=0;i<n;i++)` | `for ((i=0;i<n;i++))` |
| `for (auto x : v)` | `for x in "${v[@]}"` |
| `while (cond)` | `while [[ cond ]]` |
| `switch/case` | `case ... in ... esac` |
| `void f(int x)` | `f() { local x=$1; }` |
| `return val;` | `echo "$val"` (capture with `$(f)`), or `return N` for exit codes only |
| `cout << x;` | `echo "$x"` / `printf` |
| `cin >> x;` | `read x` |
| `cerr << x;` | `echo "$x" >&2` |
| `argc, argv` | `$#`, `$@`, `$1 $2 ...` |
| `try/catch` | `set -e` + `trap ... ERR` + `\|\| { }` |
| `assert(cond)` | `[[ cond ]] \|\| exit 1` |
| `nullptr` check | `[[ -z "$x" ]]` |
| `std::sort(v)` | `sort` / `sort -n` |
| `v.push_back(x)` | `v+=("$x")` |
| `v.size()` | `${#v[@]}` |
| `s.substr(a,b)` | `${s:a:b}` |
| `s.length()` | `${#s}` |
| `s.find(x)` | `[[ $s == *"$x"* ]]` |
| `s + t` | `"$s$t"` |
| `system("cmd")` | `cmd` (just run it directly) |

---

## 17. Debugging

```bash
bash -x script.sh        # trace every command as it executes (like a debugger stepping)
set -x                     # turn tracing on mid-script
set +x                      # turn it off
bash -n script.sh             # syntax check only, don't run
```

---

This covers variables, arithmetic, conditionals, loops, functions, arrays, strings, I/O, args, error handling, text processing, and process control — everything needed to port typical competitive-programming-style or utility C++ code into bash. For heavy numeric/float work or real data structures, prefer calling out to `awk`, `bc`, or a Python script from within bash rather than forcing it — that is idiomatic, not a cop-out.
