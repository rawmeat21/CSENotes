In Bash/Zsh, `if` statements do **not** look at text output (`stdout`). They only check the **exit code** of the command that follows them:

- **Exit code `0`** = **SUCCESS** (evaluates to `true`)
    
- **Exit code `1` (or any non-zero)** = **FAILURE** (evaluates to `false`)

```bash
is_numeric() {
    [[ "$1" == "integer" || "$1" == "real" ]]
}
```

- It executes the double-bracket expression `[[ ... ]]`.
    
- `[[ ... ]]` is a builtin command. If the condition is true, it exits with status **`0`**. If false, it exits with status **`1`**.
    
- Because `[[ ... ]]` is the **last command** inside your function, the function automatically adopts that same exit status!

```bash
if is_numeric "$t1" && is_numeric "$t2"; then
```

Here's the step-by-step under the hood:

1. Shell executes `is_numeric "$t1"`.
    
    - If `$t1` is `"integer"`, `[[ ... ]]` succeeds → exits with **0**.
        
2. Because of `&&` (AND), the shell moves to `is_numeric "$t2"`.
    
    - If `$t2` is `"real"`, `[[ ... ]]` succeeds → exits with **0**.
        
3. Both returned **0 (Success)**, so the `if` block executes!


In shell functions, there is **no explicit `return` keyword needed** to output text or data.
Instead, shell functions work like mini-terminal commands: **they print their output to `stdout` (standard output)**.

### How "Returning" Values Works in Shell

When you want to capture the "returned" value of a shell function, you use **command substitution** `$( ... )` to capture whatever the function printed to stdout.


```bash
# 1. Define the function (with fixed closing quote and parenthesis)
char_to_ascii() {
    printf '%d' "'$1"
}

# 2. Call the function inside $( ) to capture its printed output:
result=$(char_to_ascii "A")

# 3. Use the captured value:
echo $result  # Output: 65
```



The difference comes down to **Command Substitution** vs. **Arithmetic Expansion**:

|Syntax|Name|What it does|Example|
|---|---|---|---|
|**`$( ... )`**|**Command Substitution**|Runs a **shell command** (`echo`, `bc`, `awk`, `grep`) and captures its text output.|`result=$(echo "4 * 3" \| bc -l)`|
|**`$(( ... ))`**|**Arithmetic Expansion**|Evaluates a **native shell math expression** directly inside the shell (no external commands).|`result=$(( x * y ))`|


## 1. Quotes: Handling Expansion & Literals

| Syntax      | Name              | Behavior                                                                                                                                                       | Example                 | Output      |
| ----------- | ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- | ----------- |
| **`"..."`** | **Double Quotes** | **Soft quoting.** Expands variables (`$var`), command substitutions (`$(...)`), and arithmetic (`$((...))`), but preserves spaces and prevents globbing (`*`). | `x=5; echo "Value: $x"` | `Value: 5`  |
| **`'...'`** | **Single Quotes** | **Hard quoting.** Treats **everything** literally. No variables, commands, or escape characters are expanded.                                                  | `x=5; echo 'Value: $x'` | `Value: $x` |

> **Rule of Thumb:** Always double-quote variable expansions (e.g., `"$var"`) to prevent word splitting and file globbing bugs.

## 2. Parentheses & Braces: Grouping & Scope

| Syntax        | Name            | Behavior                                                                                                                                                                                                                                                                     | Common Use Cases                                           |
| ------------- | --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| **`{ ... }`** | **Braces**      | **1. Parameter Expansion:** Isolates variable names (`${var}_1`). Use this when you want to append something to the variable value while printing it.<br><br>  <br><br>**2. Group Commands:** Runs commands in the **current shell context** (modifies current environment). | `${name:-Guest}`<br><br>  <br><br>`{ list; commands; }`    |
| **`( ... )`** | **Parentheses** | **1. Subshell:** Runs enclosed commands in a **child shell process** (variable changes inside are lost outside).<br><br>  <br><br>**2. Array Definition:** Instantiates arrays.                                                                                              | `( cd /tmp && ls )`<br><br>  <br><br>`arr=(one two three)` |

## 3. Brackets: Conditional Testing

| Syntax          | Name                | Portable?                             | Key Features & Behavior                                                                                                                                                            |
| --------------- | ------------------- | ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`[ ... ]`**   | **Single Brackets** | **POSIX standard** (Works everywhere) | Alias for the traditional `test` command. Requires explicit double-quoting of variables (e.g., `[ "$x" = "y" ]`) to avoid syntax errors if a variable is empty or contains spaces. |
| **`[[ ... ]]`** | **Double Brackets** | **Shell Builtin** (Bash/Zsh only)     | Modern, safer test construct. Does **not** require quoting variables, supports regex matching (`=~`), and supports native logic ope                                                |
|                 |                     |                                       |                                                                                                                                                                                    |

when to use `[[]]` in bash?

**`[[`**: An extended test keyword added by Bash, Zsh, and KornShell that improves upon `[` with regex matching (`=~`), pattern matching (`==`), and better variable handling without quote issues. Basically, improved version of `[`.

`if` checks status code. `[[]]` is a command

| Scenario               | Use `[[ ... ]]`? | Example                       | Why?                                                               |
| ---------------------- | ---------------- | ----------------------------- | ------------------------------------------------------------------ |
| **Command / Function** | **NO**           | `if grep -q "text" file.txt`  | You are testing if the command succeeds (exit status `0`).         |
| **String Comparison**  | **YES**          | `if [[ "$t1" == "char" ]]`    | `==` is an operator, not a command. `[[` is needed to evaluate it. |
| **File Checks**        | **YES**          | `if [[ -f "/path/to/file" ]]` | `-f` is an operator provided by the test construct.                |
| **Regex Matching**     | **YES**          | `if [[ "$var" =~ ^[0-9]+$ ]]` | `=~` is a Bash operator evaluated inside `[[`.                     |
| **Numeric Comparison** | **YES**          | `if [[ "$num" -gt 10 ]]`      | `-gt` is a comparison operator.                                    |


### Common Pitfall to Avoid

Do **not** wrap your function calls in `[[ ... ]]` or `[ ... ]` like this:

Bash

```
# ❌ INCORRECT / WRONG
if [[ is_num "$t1" ]]; then ... fi
```

Inside `[[ ... ]]`, Bash treats `is_num` as a literal string rather than executing it as a command, which will cause a syntax error or give unexpected results!



### Wait a fucking minute,

```bash
echo "the sum of '$userv1' and '$userv2' is $result"
```

single quotes will work??

It looks contradictory at first glance, but here is what's actually happening: **the single quotes are inside double quotes.**

In shell scripts, **outer quotes always win**. Because the whole string is wrapped in **double quotes** (`"..."`), the shell parses the entire sentence in double-quote mode.


### 1. Handling Strings & Variables

|What you want to do|What to use|Example|Why|
|---|---|---|---|
|**Normal text with variables**|Double Quotes `""`|`echo "Hello, $user"`|Expands variables, but preserves spaces and prevents file globbing. **Default choice for 90% of code.**|
|**Raw text / Regex / Literal strings**|Single Quotes `''`|`grep 'pattern.*' file`|Stops _all_ shell expansion. What you type is literally what gets used.|
|**Attach text to a variable name**|Braces `${var}`|`file="${name}_v2.txt"`|Tells the shell where the variable name ends so it doesn't look for `$name_v2`.|
|**Set defaults or edit strings on the fly**|Braces `${var}`|`${port:-8080}`|Triggers built-in shell manipulations without calling external tools.|

### 2. If Statements & Conditions

|What you want to do|What to use|Example|Why|
|---|---|---|---|
|**Check conditions in Bash/Zsh**|Double Brackets `[[ ]]`|`if [[ $x == "a" && $y -gt 5 ]]; then`|**Always default to this.** Supports `&&`, `|
|**Write standard POSIX (`/bin/sh`) scripts**|Single Brackets `[ ]`|`if [ "$x" = "a" ]; then`|Only use this if your script **must** run on minimal Linux distributions or embedded systems without Bash/Zsh.|

### 3. Math & Shell Calculations

|What you want to do|What to use|Example|Why|
|---|---|---|---|
|**Store integer math in a variable**|Arithmetic Expansion `$(( ))`|`res=$(( x + y ))`|Evaluates native integer math inside the shell and returns the value.|
|**Do math or test numbers without storing**|Arithmetic Statements `(( ))`|`(( count++ ))`<br><br>  <br><br>`if (( x > 5 )); then`|Runs math natively without needing the `$` sign. Cleanest way to increment counters or test numbers.|
|**Floating point / Decimal math**|Command Substitution `$( )`|`res=$(echo "$x / $y" \| bc -l)`|Standard `$(( ))` can't do decimals—you have to run an external command like `bc` or `awk` inside `$( )`.|

### 4. Running Commands & Grouping

| What you want to do                         | What to use                 | Example                                | Why                                                                                                                                       |
| ------------------------------------------- | --------------------------- | -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| **Capture output of a command**             | Command Substitution `$( )` | `today=$(date +%Y-%m-%d)`              | Executes the command and substitutes the printed text right into your script.                                                             |
| **Run commands in an isolated environment** | Subshell Parentheses `( )`  | `(cd /tmp && rm -rf *)`                | Runs in a separate child shell. Changes to working directories (`cd`) or variables inside **do not affect** the rest of your main script. |
| **Group commands to share an output/pipe**  | Braces `{ ...; }`           | `{ echo "Start"; run_job; } > log.txt` | Groups multiple commands together in the **current shell context** so you can redirect or pipe all their out                              |

