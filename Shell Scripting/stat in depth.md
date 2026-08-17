### Basic syntax

bash

```bash
stat [OPTIONS] FILE...
```

By default, `stat` shows detailed metadata: size, blocks, device, inode, permissions, links, UID/GID, and the three (or four) timestamps.

bash

```bash
stat file.txt
```

```
  File: file.txt
  Size: 1024        Blocks: 8          IO Block: 4096   regular file
Device: 802h/2050d   Inode: 1234567     Links: 1
Access: (0644/-rw-r--r--)  Uid: ( 1000/rawmeat)   Gid: ( 1000/rawmeat)
Access: 2026-08-17 10:12:03.000000000 +0530
Modify: 2026-08-16 22:04:11.000000000 +0530
Change: 2026-08-16 22:04:11.000000000 +0530
 Birth: 2026-08-10 09:00:00.000000000 +0530
```

### Understanding the three (or four) timestamps

This trips people up constantly, so it's worth being precise:

- **Access (atime)** — last time file _contents_ were read
- **Modify (mtime)** — last time file _contents_ were changed
- **Change (ctime)** — last time file _metadata_ (permissions, ownership, links) changed — **not** "creation time"
- **Birth (crtime)** — actual creation time, only available on filesystems that support it (ext4, btrfs, xfs — not all older ext2/3 setups)

### Key options

|Option|Purpose|
|---|---|
|`-f`|Show filesystem status instead of file status|
|`-c FORMAT`|Custom output using format sequences|
|`-t`|Terse (single-line, script-friendly) output|
|`-L`|Follow symlinks (dereference) instead of showing the link itself|
|`--printf=FORMAT`|Like `-c` but interprets backslash escapes (`\n`, `\t`)|

#### Useful format sequences for `-c`/`--printf`

|Sequence|Meaning|
|---|---|
|`%n`|filename|
|`%s`|size in bytes|
|`%a`|permission bits (octal)|
|`%A`|permission bits (human-readable, `-rw-r--r--`)|
|`%U` / `%G`|owner / group name|
|`%u` / `%g`|owner / group UID/GID|
|`%i`|inode number|
|`%h`|number of hard links|
|`%F`|file type (regular file, directory, symlink...)|
|`%x` / `%y` / `%z`|access / modify / change time (human-readable)|
|`%X` / `%Y` / `%Z`|access / modify / change time (Unix epoch)|
|`%W`|birth time (epoch), 0 if unknown|
|`%d` / `%D`|device number (decimal / hex)|
|`%m`|mount point|

### Use cases

#### 1. Get just the file size (for scripting)

bash

```bash
stat -c %s file.txt
```

Handy for bash conditionals like checking if a log file exceeds a size threshold before rotating it:

bash

```bash
size=$(stat -c %s app.log)
[ "$size" -gt 10485760 ] && mv app.log app.log.old
```

#### 2. Get modification time as epoch (for comparisons)

bash

```bash
stat -c %Y file.txt
```

Compare two files' freshness without `find -newer`:

bash

```bash
[ "$(stat -c %Y a.txt)" -gt "$(stat -c %Y b.txt)" ] && echo "a.txt is newer"
```

#### 3. Get permissions in both octal and symbolic form

bash

```bash
stat -c "%a %A" file.sh
```

```
755 -rwxr-xr-x
```

Useful in a config-audit script checking that SSH keys are `600`:

bash

```bash
perm=$(stat -c %a ~/.ssh/id_ed25519)
[ "$perm" != "600" ] && echo "WARNING: bad permissions on private key"
```

#### 4. Check filesystem info instead of a file

bash

```bash
stat -f /
```

```
  File: "/"
    ID: abcd1234        Namelen: 255     Type: ext4/ext2
Block size: 4096       Fundamental block size: 4096
Blocks: Total: 12345678   Free: 3456789   Available: 3200000
Inodes: Total: 1000000    Free: 800000
```

Great for a quick disk-space/inode-exhaustion check without parsing `df`:

bash

```bash
stat -f -c "%b blocks total, %f free" /
```

#### 5. Terse/script-friendly single-line output

bash

```bash
stat -t file.txt
```

Outputs everything space-separated on one line — good when you need to `awk`/`cut` multiple fields at once instead of chaining several `stat -c` calls:

bash

```bash
stat -t file.txt | awk '{print "size:", $2, "mtime:", $13}'
```

#### 6. Compare symlink vs target

bash

```bash
stat /usr/bin/python      # shows the symlink's own metadata
stat -L /usr/bin/python   # follows the link, shows target's metadata
```

Useful when debugging broken symlinks or verifying which binary a symlink actually resolves to (e.g. checking your `ly`/display-manager symlinks, or Python venv symlinks).

#### 7. Detect if a file was actually created recently (not just modified)

bash

```bash
stat -c %w file.txt      # human-readable birth time
stat -c %W file.txt      # epoch (0 if filesystem doesn't support it)
```

Useful for forensics-style checks — e.g. distinguishing a config that was freshly created vs one that was just touched/edited.

#### 8. Check file type generically

bash

```bash
stat -c %F file
```

Returns things like `regular file`, `directory`, `symbolic link`, `socket`, `character special file` — handy in a script that needs to branch behavior based on what a path actually is, without separate `-f`/`-d`/`-L` test flags:

bash

```bash
case $(stat -c %F "$path") in
  "regular file") echo "processing as file";;
  "directory") echo "recursing into dir";;
  "symbolic link") echo "resolving symlink";;
esac
```

#### 9. Batch stat multiple files with a custom report

bash

```bash
stat -c "%n | %s bytes | %A | modified %y" *.conf
```

```
sshd_config | 4200 bytes | -rw-r--r-- | modified 2026-08-10 09:00:00
```

Good for a quick audit table across a whole directory of config files.

#### 10. Find hard-linked files

bash

```bash
stat -c "%i %h %n" *
```

Files sharing the same inode number (`%i`) are hard links to the same data; `%h` shows the link count. Useful when cleaning up duplicated dotfiles in a rice setup where you're not sure if something's a copy or a hardlink.

#### 11. Use in `find` for advanced filtering (combined usage)

`stat` alone doesn't recurse, but pairs well with `find -exec`:

bash

```bash
find . -type f -exec stat -c "%Y %n" {} \; | sort -n
```

Lists all files sorted by modification time (oldest first) — useful for finding stale files before a cleanup pass.

#### 12. Compare stat output between two files directly

bash

```bash
diff <(stat file1.txt) <(stat file2.txt)
```

Quick way to spot metadata drift (permission changes, ownership changes) between what should be identical files — e.g. comparing a dotfile in your repo vs the deployed version in `~/.config`.

### Quick reference: `stat` vs alternatives

- Use `stat` when you need **metadata** (permissions, timestamps, inode, size) precisely and script-parseably.
- Use `ls -l` for a quick human glance, not scripting (its output format isn't stable across systems).
- Use `file` when you need **content-based type detection** (`stat -c %F` only reports the filesystem-level type, not e.g. "ELF binary" or "ASCII text").
- Use `find -newer` for relative time comparisons instead of manually diffing two `stat -c %Y` epochs, when you don't need the raw value.