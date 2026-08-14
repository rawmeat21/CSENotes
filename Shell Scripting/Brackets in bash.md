| Bracket Syntax              | Name                   | Primary Purpose                                 | Output / Return                 |
| --------------------------- | ---------------------- | ----------------------------------------------- | ------------------------------- |
| **`[ ... ]`**               | Single Test Command    | Basic POSIX conditional testing                 | Exit status (`0` or `1`)        |
| **`[[ ... ]]`**             | Double Test Expression | Advanced Bash conditional testing               | Exit status (`0` or `1`)        |
| **`(( ... ))`**             | Arithmetic Evaluation  | Math calculations & C-style math tests          | Exit status (`0` if non-zero)   |
| **`$(( ... ))`**            | Arithmetic Expansion   | Math calculations                               | Returns the evaluated number    |
| **`( ... )`**               | Subshell               | Run commands in an isolated child process       | Exit status of last command     |
| **`$( ... )`**              | Command Substitution   | Run commands and capture standard output        | Returns command output (string) |
| **`<( ... )` / `>( ... )`** | Process Substitution   | Treat command output as a temporary file        | File path (e.g., `/dev/fd/63`)  |
| **`{ ...; }`**              | Command Grouping       | Run commands together in the current shell      | Exit status of last command     |
| **`${ ... }`**              | Parameter Expansion    | Advanced variable manipulation & default values | Returns modified variable value |
| **`{1..5}` / `{a,b}`**      | Brace Expansion        | Generates string sequences/lists                | Expands into multiple strings   |

## 1. Conditional Testing Brackets

### `[ ... ]` (Single Square Brackets)

This is an alias for the standard POSIX `test` command.

- **What's Allowed:**
    
    - File checks (`-f`, `-d`, `-r`).
        
    - String checks (`-z`, `-n`).
        
    - POSIX string comparisons (`=`, `!=`).
        
    - Integer comparators (`-eq`, `-ne`, `-gt`, `-lt`).
        
- **What's NOT Allowed / Pitfalls:**
    
    - **Unquoted variables will break:** If `$var` is empty, `[ $var = "foo" ]` expands to `[ = "foo" ]` (Syntax Error). Always quote variables: `[ "$var" = "foo" ]`.
        
    - **No `<` or `>` without escaping:** `[ $a > $b ]` will silently redirect output to a file named `$b`! You must use `[ "$a" \> "$b" ]`.
        
    - **No `&&` or `||` inside:** You cannot use `[ $a -eq 1 && $b -eq 2 ]`. You must use separate brackets: `[ ... ] && [ ... ]`.
        
- **When to Use:** Only when writing portable scripts intended to run on plain POSIX shells (`/bin/sh`, `dash`, BusyBox).
    

### `[[ ... ]]` (Double Square Brackets)

This is an extended Bash/Zsh keyword that completely upgrades `[ ... ]`.

- **What's Allowed:**
    
    - Safely using unquoted variables (`[[ $var == "foo" ]]` works even if `$var` is empty).
        
    - Native `&&` and `||` logical operators.
        
    - Direct string comparison with `<` and `>` without escaping.
        
    - Pattern matching with wildcard globs (`[[ $str == file* ]]`).
        
    - Regular expressions using `=~` (`[[ $email =~ ^[A-Z] ]]`).
        
- **What's NOT Allowed / Pitfalls:**
    
    - **Not portable:** Will cause a syntax error in standard `/bin/sh`.
        
    - **Not for C-style math:** Don't do addition inside it (`[[ $x + 1 -eq 5 ]]` won't work automatically; use `$((x + 1))` or `(( ... ))`).
    
    - **Cannot run commands inside it
        
- **When to Use:** Always use `[[ ... ]]` over `[ ... ]` for conditional tests in Bash scripts.
    
In general, use this to check conditions. For complex math conditions, prefer `(())`

## 2. Arithmetic Brackets

### `(( ... ))` (Double Parentheses — Evaluation)

Evaluates C-style mathematical expressions and sets an exit status.

- **What's Allowed:**
    
    - Standard math operators (`+`, `-`, `*`, `/`, `%`, `**`).
        
    - Increment/decrement (`++`, `--`, `+=`, `-=`).
        
    - Comparison operators (`>`, `<`, `>=`, `<=`, `==`, `!=`).
        
    - Omitting the `$` on variables: `(( count + 1 > max ))`.
        
- **What's NOT Allowed / Pitfalls:**
    
    - **No Floating-Point Math:** Bash only handles whole integers (`(( 5 / 2 ))` equals `2`).
        
    - **Exit Code Quirks:** If the math result is `0`, the command returns exit code `1` (false). Example: `(( 5 - 5 ))` exits with status `1`!
        
- **When to Use:** Performing math side-effects (`(( count++ ))`) or numerical conditions inside `if` statements (`if (( x > 10 )); then`).
    
Use only for math expressions and checking conditions.
### `$(( ... ))` (Double Parentheses with `$` — Expansion)

Same math rules as `(( ... ))`, but it **returns the numerical result** as a string so you can print or assign it.

- **Example Usage:**
    
    Bash
    
    ```
    result=$(( (5 + 10) * 2 ))
    echo "You have $(( 10 - count )) tries left."
    ```
    
- **When to Use:** Whenever you need to calculate a number and assign it to a variable or pass it to a command.
    

## 3. Subshells & Command Execution

### `( ... )` (Single Parentheses — Subshell)

Runs all commands inside a **new child process (subshell)**.

- **What's Allowed:** Any standard shell commands separated by `;` or newlines.
    
- **What's NOT Allowed / Pitfalls:**
    
    - **Variable Isolation:** Variable assignments inside `( ... )` disappear once the subshell exits.
        
        Bash
        
        ```
        x=10
        ( x=20 )
        echo $x # Prints 10!
        ```
        
- **When to Use:**
    
    - Running temporary state changes without affecting the parent script: `( cd /tmp && rm -rf build/ )`.
        
    - Grouping pipeline output: `( echo "Header"; cat file.txt ) | gzip > output.gz`.
        

### `$( ... )` (Command Substitution)

Runs the command inside a subshell and **replaces itself with the command's standard output**.

- **What's Allowed:** Nesting multiple levels deep easily: `$(dirname $(which node))`.
    
- **What's NOT Allowed / Pitfalls:**
    
    - Do not use the legacy backtick syntax `` `command` ``—it is harder to read and difficult to nest.
        
- **When to Use:** Capturing output into a variable: `today=$(date +%Y-%m-%d)`.
    

### `<( ... )` or `>( ... )` (Process Substitution)

Runs a command and makes its output appear as a **temporary file path** (e.g., `/dev/fd/63`).

- **What's Allowed:** Passing command outputs into programs that _require_ file paths as arguments instead of standard input.
    
- **Examples:**
    
    Bash
    
    ```
    # Compare the output of two commands directly
    diff -u <(ls dir1) <(ls dir2)
    
    # Send output to two commands simultaneously
    tar -czf - /data | tee >(md5sum > data.md5) >(sha256sum > data.sha256)
    ```
    
- **When to Use:** When a command doesn't accept streamed input via pipe `|` and demands a file parameter.
    

## 4. Curly Braces (Grouping & Variables)

### `{ ...; }` (Group Commands — Current Shell)

Executes a group of commands inside the **current shell environment** (no subshell created).

- **What's Allowed:** Variable changes inside the block **persist** after the block finishes.
    
- **STRICT Syntax Rules (What's NOT allowed):**
    
    - You **MUST** have a space after the opening `{`.
        
    - You **MUST** end the last command with a semicolon `;` or a newline before the closing `}`.
        
    - ❌ Incorrect: `{ echo "hi" }`
        
    - ✅ Correct: `{ echo "hi"; }`
        
- **When to Use:** Redirecting output for multiple commands at once without the overhead of spawning a subshell:
    
    Bash
    
    ```
    {
      echo "Log Start"
      date
    } >> app.log
    ```
    

### `${ ... }` (Parameter Expansion)

Explicitly references a variable and allows string manipulation, default assignments, and array operations.

- **Common Use Cases:**
    
    - **Disambiguation:** `echo "${var}_file.txt"`
        
    - **Default Values:** `${var:-"default"}` (use "default" if `$var` is unset)
        
    - **String Length:** `${#var}`
        
    - **Substring:** `${var:0:5}` (first 5 characters)
        
    - **Find & Replace:** `${var/search/replace}`
        
    - **Array Access:** `${array[0]}` or `${array[@]}`
        
- **When to Use:** Any time standard `$var` isn't powerful enough or runs into string boundary ambiguity.
    

### `{a..b}` or `{1,2,3}` (Brace Expansion)

Generates sets or ranges of text **before** any command executes.

- **Examples:**
    
    - Range: `echo {1..5}` → `1 2 3 4 5`
        
    - Zero-padded range: `echo {01..05}` → `01 02 03 04 05`
        
    - Lists: `echo {jpg,png,gif}` → `jpg png gif`
        
    - Combinations: `mkdir -p project/{src,bin,doc}`
        
- **What's NOT Allowed / Pitfalls:**
    
    - **No quotes around braces:** Quoting disables expansion (`"{1..5}"` prints literal `{1..5}`).
        
    - **No variables in standard Bash ranges:** `{1..$max}` will not expand in older Bash versions; use a `for` loop or `seq` instead.


## How to Remember ALL These Rules (The Universal Mental Model)

**4 Fundamental Building Blocks**:

```
              ┌──────────────────────────────────────────────┐
              │          THE 4 BASH BUILDING BLOCKS          │
              └──────────────────────────────────────────────┘

  1. THE SIGIL          2. PARENTHESES         3. BRACKETS          4. BRACES
      [$]                   ( )                    [ ]                 { }
  "Get Output/Value"    "Run Code / Math"     "Evaluate Truth"     "Group / Structure"
```

### The Combination Formula

You can combine **`$` (Value)** with any symbol to transform its behavior:

|Base Symbol|What Base Symbol Does|Add `$` to Front|What it Becomes|
|---|---|---|---|
|**`( cmd )`**|Run in subshell|**`$( cmd )`**|Capture command output as text|
|**`(( math ))`**|Evaluate math condition|**`$(( math ))`**|Calculate math and return the number|
|**`{ var }`**|Parameter boundary|**`${ var }`**|Get variable value / manipulate string|