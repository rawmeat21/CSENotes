In Bash, **script arguments** and **function arguments** work using the exact same system of **positional parameters**.

Here is the complete guide on how to inspect, count, manipulate, save, and parse arguments in Bash.

## 1. Quick-Reference Variables

When a script or function is executed, Bash automatically populates these special variables:

| Variable             | Meaning                            | Details / Notes                                                                                    |
| -------------------- | ---------------------------------- | -------------------------------------------------------------------------------------------------- |
| **`$1` ... `$9`**    | Argument 1 through 9               | The individual arguments passed in order.                                                          |
| **`${10}`**          | Argument 10 and higher             | **Curly braces required!** `$10` will expand to `$1` followed by a literal `0`.                    |
| **`$#`**             | Number of arguments                | Total count of arguments passed.                                                                   |
| **`"$@"`**           | All arguments (as separate words)  | **Best practice.** Preserves quotes and exact spacing.                                             |
| **`"$*"`**           | All arguments (as a single string) | Joins all arguments into one string separated by the first character of `$IFS` (usually space).    |
| **`$0`**             | Script name                        | Stores the name of the running script. _(Note: Inside a function, `$0` is still the script name!)_ |
| **`${FUNCNAME[0]}`** | Function name                      | Use this inside a function to get its actual name.                                                 |

## 2. Functions vs. Scripts: How They Interact

Inside a function, positional parameters are **locally scoped**. The function gets its own set of `$1`, `$2`, `$#`, and `"$@"`, completely hiding the script's global arguments while inside that function block.

Bash

```
#!/bin/bash

my_function() {
    echo "Inside function: $1 (Count: $#)"
    echo "Function name: ${FUNCNAME[0]}"
}

# Running script with arguments
echo "Inside script: $1 (Count: $#)"
my_function "apple" "banana"  # Overrides $1, $2, and $# inside my_function
```

## 3. Handling Argument Counts & Defaults

### Checking for Required Arguments

Bash

```
# Ensure at least 2 arguments are provided
if [[ $# -lt 2 ]]; then
    echo "Error: Expected at least 2 arguments, got $#." >&2
    echo "Usage: $0 <input_file> <output_file>" >&2
    exit 1
fi
```

### Providing Default Values for Optional Arguments

Use parameter expansion `${1:-default}` to assign fallback values if an argument isn't provided:

Bash

```
filename="${1:-"data.txt"}"  # If $1 is empty, use "data.txt"
port="${2:-8080}"             # If $2 is empty, use 8080
```

## 4. `"$@"` vs `"$*"` (The Most Important Difference)

Always prefer **`"$@"`** when passing or iterating over arguments.

Bash

```
# Suppose arguments are: "hello world" "foo"

# ❌ BAD: "$*" merges everything into one single argument
for arg in "$*"; do
    echo "$arg"
done
# Output:
# hello world foo   <-- Processed as 1 item!

# ✅ GOOD: "$@" keeps each argument distinct
for arg in "$@"; do
    echo "$arg"
done
# Output:
# hello world       <-- Item 1 (spaces preserved!)
# foo               <-- Item 2
```

## 5. Saving Arguments to an Array

To store all arguments into an array for later processing without losing spacing:

Bash

```
# Save all arguments into an array
args=("$@")

# Save arguments starting from the 2nd argument onward
sub_args=("${@:2}")

# Access elements
echo "First saved arg: ${args[0]}"
echo "Total saved args: ${#args[@]}"
```

## 6. Shifting Arguments (`shift`)

The `shift` command pops the first argument (`$1`) off the list and moves all remaining arguments down by one (`$2` becomes `$1`, `$3` becomes `$2`, and `$#` decreases by 1).

Bash

```
#!/bin/bash

# Grab the first argument as a command/action
action="$1"
shift  # Remove $1 so "$@" now contains only the remaining parameters

case "$action" in
    start)
        echo "Starting service with options: "$@""
        ;;
    stop)
        echo "Stopping service..."
        ;;
    *)
        echo "Unknown action: $action"
        ;;
esac
```

You can also shift multiple positions at once: `shift 3` drops the first 3 arguments.

## 7. Advanced Option Parsing (Flags & Named Args)

### Method A: Manual Parsing with `while`, `case`, and `shift`

_Best for scripts that need to support both short flags (`-v`) and long flags (`--file`)._

Bash

```
#!/bin/bash

verbose=false
output_file="default.out"

while [[ $# -gt 0 ]]; do
    case "$1" in
        -v|--verbose)
            verbose=true
            shift # Move past argument
            ;;
        -o|--output)
            output_file="$2"
            shift 2 # Move past flag AND its value
            ;;
        -h|--help)
            echo "Usage: $0 [-v|--verbose] [-o|--output FILE]"
            exit 0
            ;;
        *)
            echo "Unknown option: $1" >&2
            exit 1
            ;;
    esac
done

echo "Verbose: $verbose"
echo "Output file: $output_file"
```

### Method B: Native `getopts`

_Best for standard single-character flags (`-a`, `-b value`). Built directly into Bash._

Bash

```
#!/bin/bash

# The colon after 'o' means -o requires a value (e.g. -o output.txt)
while getopts "vho:" opt; do
    case "$opt" in
        v) verbose=true ;;
        o) output_file="$OPTARG" ;;
        h) 
           echo "Usage: $0 [-v] [-o output_file]"
           exit 0
           ;;
        \?) exit 1 ;;
    esac
done

# Remove parsed flags from the positional parameters
shift $((OPTIND - 1))

# Remaining non-flag positional arguments are now in $1, $2, etc.
echo "Remaining non-flag arguments: $@"
```