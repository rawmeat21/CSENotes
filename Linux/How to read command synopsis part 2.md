## Options vs. Arguments: Formal Definitions

In systems programming and command-line parsing (specifically how functions like POSIX `getopt()` parse the `argv` array), elements are strictly categorized into three types:

| Component                         | Technical Definition                                                                                                                                 | Example                    |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| **Option** (Flag)                 | A string starting with `-` or `--` that modifies the utility's behavior. They are generally unordered.                                               | `-v`, `--bare`             |
| **Option-argument**               | A mandatory value structurally bound to a specific option.                                                                                           | `<path>` in `-C <path>`    |
| **Operand** (Positional Argument) | Target data the command operates on. They do not start with a hyphen (unless preceded by `--` to terminate option parsing) and are strictly ordered. | `destination`, `[FILE]...` |

## Deconstructing the Synopses

### 1. `git`

Plaintext

```
git [-v | --version] [-h | --help] [-C <path>] [-c <name>=<value>]
    [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
    [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--no-lazy-fetch]
    [--no-optional-locks] [--no-advice] [--bare] [--git-dir=<path>]
    [--work-tree=<path>] [--namespace=<name>] [--config-env=<name>=<envvar>]
    <command> [<args>]
```

**Parsing Mechanics:**

- **Global vs. Subcommand separation:** Every option listed _before_ `<command>` is a global `git` option. The `[<args>]` at the end belong specifically to the `<command>` you invoke.
    
- **Nested optionality:** `[--exec-path[=<path>]]` means you can pass `--exec-path` alone (which returns the current path), or pass it with a value like `--exec-path=/usr/lib/git-core` to override it.
    
- **Key Execution Rule:** You cannot place global options after the subcommand.
    
    - _Valid:_ `git -c user.name=Romit commit -m "init"`
        
    - _Invalid:_ `git commit -m "init" -c user.name=Romit` (The parser treats `-c` as an argument to `commit`, which fails).


### 2. `docker`

Plaintext

```
docker [OPTIONS] COMMAND [ARG...]
docker [--help|-v|--version]
```

**Parsing Mechanics:**

- **Documentation Macros:** `[OPTIONS]` is a macro. Instead of printing 50 global flags on line one, the man page groups them under `[OPTIONS]` and details them further down.
    
- **Mutually Exclusive Modes:**
    
    - Line 1: Operational mode. Execute a `COMMAND` (like `run`, `build`), optionally passing `ARG...` (like image names or container commands).
        
    - Line 2: Informational mode. You pass `--help` or `--version`, and no `COMMAND` is allowed.

### 3. `find`

Plaintext

```
find [-H] [-L] [-P] [-D debugopts] [-Olevel] [starting-point...] [expression]
```

**Parsing Mechanics:**

- **Space vs. No Space:** Notice `[-D debugopts]` vs. `[-Olevel]`.
    
    - `-D` requires a space before its argument (e.g., `-D search`).
        
    - `-O` mandates immediate concatenation (e.g., `-O3`). The parser checks the characters immediately following the `O` in the same `argv` index.
        
- **Trailing Lists:** Both `[starting-point...]` (paths to search) and `[expression]` (predicates like `-name "*.txt"`) are optional in GNU `find`. If omitted, it defaults to the current directory (`.`) and prints everything (`-print`).


### 4. `grep`

Plaintext

```
grep [OPTION]... PATTERNS [FILE]...
grep [OPTION]... -e PATTERNS ... [FILE]...
grep [OPTION]... -f PATTERN_FILE ... [FILE]...
```

**Parsing Mechanics:**

- **Repeatability:** `[OPTION]...` means you can stack multiple options (`-i -v -n` or `-ivn`).
    
- **Execution Modes (Pattern Provision):**
    
    - Line 1: The first non-option operand is unconditionally treated as the `PATTERNS`.
        
    - Line 2: Explicit pattern definition using `-e`. Because of the `...`, you can pass multiple patterns: `grep -e "error" -e "warn" file.log`.
        
    - Line 3: File-based pattern provision. `-f` expects a file containing patterns.
        
- **`[FILE]...`:** Optional, repeatable operands. If omitted, `grep` reads from standard input (`stdin`).

### 5. `mkdir`

Plaintext

```
mkdir [OPTION]... DIRECTORY...
```

**Parsing Mechanics:**

- **Mandatory Operands:** Notice that `DIRECTORY...` has no square brackets `[ ]`. You must provide at least one directory name.
    
	- **Repeatable Operands:** The `...` means you can pass multiple directories in one invocation (`mkdir -p dir1 dir2 dir3`). The `argv` array is parsed until a null pointer is reached.