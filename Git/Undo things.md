https://git-scm.com/book/en/v2/Git-Basics-Undoing-Things

```console
$ git commit --amend
```

what it does: it replaces the last commit with the commit from current staging area

The staging area actually contains more than the modfied changes. Specifically, it holds the files in the last commit.

"Last commit" specifically means **`HEAD`**, the active commit your repository is standing on right now.

The staging area (`.git/index`) always holds the full file tree of **`HEAD`**+ any changes you have explicitly added on top of it via `git add`.


If you commit and then realize you forgot to stage the changes in a file you wanted to add to this commit, you can do something like this:

```console
$ git commit -m 'Initial commit'
$ git add forgotten_file
$ git commit --amend
```

You end up with a single commit, the second commit replaces the results of the first.

The obvious value to amending commits is to make minor improvements to your last commit, without cluttering your repository history with commit messages of the form, “Oops, forgot to add a file” or “Darn, fixing a typo in last commit”.


### Unstaging a Staged File

```console
$ git reset HEAD <filename>
```
`git reset HEAD filename` overwrites the entry for `filename`**inside your Staging Area** (`.git/index`) with the exact version stored in `HEAD` (your current commit), while leaving your Working Directory on disk completely untouched.

So it's like this:

```
---Staging area---

view of commit pointed by HEAD:

...
file.txt
...

view of staged changes to current commit (HEAD):

...
file.txt (modified!)
...
```

On running this command, that modified `file.txt` in staging area is replaced by the `file.txt` in the HEAD commit. What if `file.txt` is not present in the HEAD commit at all? Well, then `file.txt` is just removed from the staging area.

### Unmodifying a Modified File

What if you realize that you don’t want to keep your changes to the `CONTRIBUTING.md` file? How can you easily unmodify it — revert it back to what it looked like when you last committed (or initially cloned, or however you got it into your working directory)?

```console
$ git checkout -- CONTRIBUTING.md
```

`git checkout -- file.txt` copies the version of the file from your **Staging Area (`.git/index`)**, not directly from `HEAD`!

If your Staging Area happens to be identical to `HEAD`, the result looks identical. However, if you have staged changes, the behavior diverges.

Imagine `file.txt` goes through these exact states:

1. **In `HEAD` (Last Commit)**: File contains `Version A`.
    
2. **You edit on disk**: File now contains `Version B`.
    
3. **You stage it (`git add file.txt`)**: The Staging Area (`.git/index`) now holds `Version B`.
    
4. **You edit on disk again**: File in your Working Directory now contains `Version C`.
    

If you now run `git checkout -- file.txt`:

- **What you may think**: The file on disk reverts to `Version A` (from `HEAD`).
    
- **What actually happens**: Git copies from `.git/index`, so the file on disk reverts to **`Version B`** (your staged version).

So, here's something interesting: Say you have just created a file and staged it. Then you try this command. Then, you would see nothing, as `git` replaced the file's version in staging area with the version in your working directory, which will have the same content!

What if you want to replace with `Version A`? - Pass HEAD as source.

```bash
git checkout HEAD -- file.txt
```


**It’s important to understand that `git checkout -- <file>` is a dangerous command. Any local changes you made to that file are gone — Git just replaced that file with the last staged or committed version. Don’t ever use this command unless you absolutely know that you don’t want those unsaved local changes.**


### `git restore` (new alt for git reset)


#### Unstaging a Staged File with git restore

```console
$ git restore --staged CONTRIBUTING.md
```

How is this different from `git reset`?

Without flags, the two commands target opposite locations:

- **`git reset HEAD file.txt`** targets the **Staging Area**. It overwrites `.git/index` with the version from `HEAD` and leaves your working directory untouched.
    
- **`git restore file.txt`** targets the **Working Directory**. It overwrites your local disk file with the version from `.git/index` and leaves your staging area untouched.
    
To make `git restore` match the behavior of `git reset HEAD file.txt`, you must explicitly pass the `--staged` flag: `git restore --staged file.txt`.


`git reset` is fundamentally a pointer-manipulation tool, whereas `git restore` is strictly a file-content tool.

- **`git reset`** can move your branch pointer backward or forward in history. Running `git reset --hard HEAD~1` rewires your current branch pointer to the parent commit, discarding commits.
    
- **`git restore`** can **never** move a branch pointer or alter commit history. It only copies file contents into your working directory or staging area.


#### Unmodifying a Modified File with git restore

```console
$ git restore CONTRIBUTING.md
```








