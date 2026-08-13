## 1. Symbol Legend

| Symbol / Syntax        | Rule             | Meaning                                                  | Example                |
| ---------------------- | ---------------- | -------------------------------------------------------- | ---------------------- |
| **`[ ]`**              | **Optional**     | Everything inside square brackets may be omitted.        | `[-v]`                 |
| **`< >`** or _Italics_ | **Placeholder**  | A positional variable you must replace with actual data. | `<path>`, `<name>`     |
| **`\|`**               | **Exclusive OR** | Pick **exactly one** of the pipe-separated choices.      | `-p \| --paginate`     |
| **`...`**              | **Repeatable**   | The preceding element can be repeated one or more times. | `[argument ...]`       |
| **No Brackets**        | **Mandatory**    | Argument/placeholder **must** be provided.               | `destination` in `ssh` |

## 2. Option & Flag Conventions

### Single-Letter Flags (POSIX Short Options)

- **Combined Options `[-46AaCf...]`**: Single-character flags prefixed with a single hyphen `-` can be passed individually (`-4 -A`) or aggregated together under a single hyphen (`-4A`).
    
- **Options taking arguments `[-i identity_file]`**: The flag requires a value. Depending on the tool's parser, space separator (`-i id_rsa`) or immediate concatenation (`-iid_rsa`) may be allowed.


### GNU Long Options

- **Double Hyphens `--option`**: Long options use two hyphens (e.g., `--version`, `--help`).
    
- **Explicit Assignment `[--exec-path[=<path>]]`**:
    
    - Outer `[ ]` means `--exec-path` itself is optional.
        
    - Inner `[=<path>]` means the value `=<path>` is optional even if `--exec-path` is passed.
        
    - Contrast with `--git-dir=<path>`: The outer brackets are missing around `<path>`, meaning if `--git-dir` is passed, supplying `<path>` is **mandatory**.


## 3. Structural Parsing Rules

### Nested Optionality

Nested brackets indicate dependency relationships:

Format: `destination [command [argument ...]]`

1. `destination` is mandatory.
    
2. `[command ...]` is optional.
    
3. `[argument ...]` can **only** be supplied if `command` is present.
    
4. `...` allows multiple trailing arguments (`arg1 arg2 arg3`).


### Separate Synopsis Lines

When a man page lists multiple separate lines under `SYNOPSIS`, it signifies **mutually exclusive operational modes**.

For example, `ssh` lists:

1. `ssh [...] destination [command [argument ...]]` — Normal connection/execution mode.
    
2. `ssh [-Q query_option]` — Query mode (e.g., `ssh -Q cipher`).
    

You cannot mix options from mode #2 into an invocation targeting mode #1.


## 4. Converting Synopsis to Concrete Invocation

Given this subset of the `ssh` synopsis:

Plaintext

```
ssh [-4] [-i identity_file] [-p port] destination [command [argument ...]]
```

### Valid Command Forms:

- **Minimal required invocation:**
    
    Bash
    
    ```
    ssh user@example.com
    ```
    
- **Adding optional flags:**
    
    Bash
    
    ```
    ssh -4 -i ~/.ssh/id_ed25519 user@example.com
    ```
    
- **Adding mandatory argument + optional trailing command/args:**
    
    Bash
    
    ```
    ssh user@example.com uname -a
    ```
    

## 5. Standard Compliance & Deviations

**Does every utility follow these rules?**

Most standard Unix tools, Linux utilities, and modern CLIs follow POSIX/GNU guidelines, but exceptions exist:

1. **Strict POSIX/GNU tools** (`ls`, `git`, `ssh`, `grep`, `find`*): Adhere to standard `-` / `--` / positional conventions.
    
2. **Legacy Unix utilities**:
    
    - `dd`: Uses key-value pairs directly without hyphens (`dd if=/dev/urandom of=file.bin bs=1M count=10`).
        
    - `tar`: Historically allowed flag aggregation without hyphens (`tar xvf archive.tar.gz`).
        
3. **Non-standard subcommands / toolchains**:
    
    - `go`: Uses single hyphens for multi-character options (`go build -work`).
        
    - `java`: Uses single hyphens for long flags (`java -version`, `java -classpath .`).
        
    - `find`: Uses predicate-style flags starting with a single hyphen (`find . -name "*.txt" -type f`).
        

While syntax parsing rules (`[ ]`, `< >`, `|`, `...`) remain identical across almost all documentation, always check the tool's parser style if options use unusual hyphenation patterns.


