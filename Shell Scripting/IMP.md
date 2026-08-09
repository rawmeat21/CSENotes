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

|Syntax|Name|Behavior|Example|Output|
|---|---|---|---|---|
|**`"..."`**|**Double Quotes**|**Soft quoting.** Expands variables (`$var`), command substitutions (`$(...)`), and arithmetic (`$((...))`), but preserves spaces and prevents globbing (`*`).|`x=5; echo "Value: $x"`|`Value: 5`|
|**`'...'`**|**Single Quotes**|**Hard quoting.** Treats **everything** literally. No variables, commands, or escape characters are expanded.|`x=5; echo 'Value: $x'`|`Value: $x`|

> **Rule of Thumb:** Always double-quote variable expansions (e.g., `"$var"`) to prevent word splitting and file globbing bugs.

## 2. Parentheses & Braces: Grouping & Scope

|Syntax|Name|Behavior|Common Use Cases|
|---|---|---|---|
|**`{ ... }`**|**Braces**|**1. Parameter Expansion:** Isolates variable names (`${var}_1`).<br><br>  <br><br>**2. Group Commands:** Runs commands in the **current shell context** (modifies current environment).|`${name:-Guest}`<br><br>  <br><br>`{ list; commands; }`|
|**`( ... )`**|**Parentheses**|**1. Subshell:** Runs enclosed commands in a **child shell process** (variable changes inside are lost outside).<br><br>  <br><br>**2. Array Definition:** Instantiates arrays.|`( cd /tmp && ls )`<br><br>  <br><br>`arr=(one two three)`|

## 3. Brackets: Conditional Testing

|Syntax|Name|Portable?|Key Features & Behavior|
|---|---|---|---|
|**`[ ... ]`**|**Single Brackets**|**POSIX standard** (Works everywhere)|Alias for the traditional `test` command. Requires explicit double-quoting of variables (e.g., `[ "$x" = "y" ]`) to avoid syntax errors if a variable is empty or contains spaces.|
|**`[[ ... ]]`**|**Double Brackets**|**Shell Builtin** (Bash/Zsh only)|Modern, safer test construct. Does **not** require quoting variables, supports regex matching (`=~`), and supports native logic ope|