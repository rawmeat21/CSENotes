## Part 1: The `find` Command

While `ls` lists what’s inside a directory, `find` recursively searches through directory trees based on attributes like name, size, modification date, and permissions.

### Fundamental Syntax

Bash

```
find [path] [filters/conditions] [actions]
```

### 1. Essential Filters

#### A. Search by Name

Bash

```
# Exact match
find /path/to/dir -name "file.txt"

# Case-insensitive match (-iname)
find . -iname "*.JPG"

# Match files NOT named "config.json"
find . ! -name "config.json"
```

#### B. Search by File Type (`-type`)

Bash

```
find . -type f   # Regular files
find . -type d   # Directories
find . -type l   # Symbolic links
```

#### C. Search by Size (`-size`)

- `k` = Kilobytes, `M` = Megabytes, `G` = Gigabytes
    

Bash

```
find . -size +100M   # Files LARGER than 100 Megabytes
find . -size -10k    # Files SMALLER than 10 Kilobytes
find . -size 0b      # Completely empty files
```

#### D. Search by Time (`-mtime`, `-atime`, `-ctime`)

- **`-mtime`**: Modification time (file content changed).
    
- **`-atime`**: Access time (file was read).
    
- **`-ctime`**: Status change time (permissions, owner, or content changed).
    

Bash

```
find . -mtime -7    # Modified within the LAST 7 days
find . -mtime +30   # Modified MORE than 30 days ago
find . -mtime 0     # Modified in the last 24 hours
```

#### E. Controlling Depth (`-maxdepth` & `-mindepth`)

By default, `find` searches all subdirectories recursively. Use depth limits to restrict it:

Bash

```
# Search ONLY the current directory (do not enter subfolders)
find . -maxdepth 1 -type f

# Skip the top-level directory itself and search subfolders
find . -mindepth 2
```

### 2. Actions: Doing Something with Found Files

#### A. Execute Commands (`-exec`)

Instead of piping output, `-exec` runs a command directly on every matched file.

- **`{}`**: Placeholder replaced by the current file path.
    
- **`\;`**: Runs the command **once per file** (slower, but necessary for line-by-line tasks).
    
- **`+`**: Bundles files together to run the command **once on all files** (much faster).
    

Bash

```
# Change permissions on all directories found
find /var/www -type d -exec chmod 755 {} +

# Search for a string inside all matched .sh files
find . -name "*.sh" -exec grep -H "TODO" {} \;
```

#### B. Deleting Matches (`-delete`)

Bash

```
# Safely test first!
find . -name "*.tmp" -type f

# Then delete
find . -name "*.tmp" -type f -delete
```

#### C. Safe Stream Handling (`-print0`)

Filenames with spaces or newlines can break standard loops. `-print0` separates file names using a null character (`\0`) instead of a newline, making it 100% safe to pipe into `xargs -0` or `readarray -d ''`.

Bash

```
find . -type f -print0 | xargs -0 rm -f
```

## Part 2: The `stat` Command

While `ls -l` shows basic information, `stat` displays **all raw metadata** stored in a file's inode (inode number, permissions, exact timestamp down to the nanosecond, block size, owner UID/GID).

### Fundamental Syntax

Bash

```
stat [options] filename
```

### Basic Output Explained

Plaintext

```
  File: report.pdf
  Size: 1048576   	Blocks: 2048       IO Block: 4096   regular file
Device: 801h/2049d	Inode: 1234567     Links: 1
Access: (0644/-rw-r--r--)  Uid: ( 1000/  alice)   Gid: ( 1000/  alice)
Access: 2026-08-10 14:20:00.000000000 +0000
Modify: 2026-08-09 09:15:30.000000000 +0000
Change: 2026-08-09 09:15:30.000000000 +0000
 Birth: 2026-08-01 10:00:00.000000000 +0000
```

- **Access (atime):** Last time the file was read/opened.
    
- **Modify (mtime):** Last time the file's **contents** were modified.
    
- **Change (ctime):** Last time the file's **metadata** or content changed (e.g., `chmod`, `chown`).
    
- **Birth (btime):** File creation date (supported on ext4, btrfs, macOS).
    

### Custom Formatting (`-c` or `--format`)

Instead of parsing full text output, use `-c` to extract specific attributes in custom formats.

#### Essential Format Tokens

|Token|Details|Example Output|
|---|---|---|
|**`%n`**|File name|`data.csv`|
|**`%s`**|Total size in bytes|`2048`|
|**`%F`**|File type|`regular file` or `directory`|
|**`%a`**|Permissions in octal|`755`|
|**`%A`**|Permissions in human-readable format|`-rw-r--r--`|
|**`%U`**|User name of owner|`alice`|
|**`%y`**|Human-readable Modification time (mtime)|`2026-08-14 10:30:00`|
|**`%Y`**|Epoch timestamp of Modification time (seconds since 1970)|`1786617000`|

#### Custom Formatting Examples

Bash

```
# Print only file size in bytes
stat -c %s file.txt

# Print permissions in octal and owner name
stat -c "%a %U" script.sh
# Output: 755 alice

# Print filename, size, and formatted modification date
stat -c "%n -> Size: %s bytes | Modified: %y" document.pdf
```

## Part 3: Combining `find` and `stat`

Combining both commands allows you to locate specific files and extract specific metadata without clutter.

### Example 1: Print File Name and Size for All Files in Top Level

Bash

```
find . -maxdepth 1 -type f -exec stat -c "%n: %s bytes" {} +
```

### Example 2: Calculate File Sizes Inside a Loop

Bash

```
while IFS= read -r -d '' file; do
    size=$(stat -c %s "$file")
    echo "Processing $file ($size bytes)..."
done < <(find . -type f -name "*.log" -print0)
```

## Quick-Reference Cheat Sheet

| Task                                 | Command                     |
| ------------------------------------ | --------------------------- |
| **Find file by name**                | `find . -name "*.txt"`      |
| **Find directories only**            | `find . -type d`            |
| **Find files modified in last 24h**  | `find . -type f -mtime 0`   |
| **Find files > 50MB and delete**     | `find . -size +50M -delete` |
| **Get size of a file in bytes**      | `stat -c %s filename`       |
| **Get octal permissions (e.g. 755)** | `stat -c %a filename`       |
| **Get file modification timestamp**  | `stat -c %y filename`       |