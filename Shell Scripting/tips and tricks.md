
```
cd "$(dirname "$(readlink -f "$0")")"
```

This command runs 3 commands:
1. readlink -f "$0" determines the path to the current script ($0)
2. dirname converts the path to script to the path to its directory
3. cd changes the current work directory to the directory it receives from dirname


### useful ls options:

-c -> Sort ﬁles by change time
-d, --directory -> List directory entries
-r, --reverse -> Show contents in reverse order
-s, --size -> Print size of each ﬁle in blocks
-S -> Sort by ﬁle size
--sort=WORD -> Sort contents by a word. (i.e size, version, status)
-t -> Sort by modiﬁcation time
-u -> Sort by last access time
-v -> Sort by version


### List Files Without Using `ls`

```
# display the files and directories that are in the current directory
printf "%s\n" *
# display only the directories in the current directory
printf "%s\n" */
# display only (some) image files
printf "%s\n" *.{gif,jpg,png}
```

```
files=( * )
# iterate over them
for file in "${files[@]}"; do
echo "$file"
done
```


### Concatenate ﬁles

```
cat file1 file2 file3 > file_all
```
```
cat file1 file2 file3 | grep foo
```

In case the content needs to be listed backwards from its end the command tac can be used:
```
tac file.txt
```
If you want to print the contents with line numbers, then use -n with cat:
```
cat -n file.txt
```


```
cat <<END >file
Hello, World.
END
```
The token after the << redirection symbol is an arbitrary string which needs to occur alone on a line (with no leading or trailing whitespace) to indicate the end of the here document. You can add quoting to prevent the shell from performing command substitution and variable interpolation:

```
cat <<'fnord'
Nothing in `here` will be $changed
fnord
```





