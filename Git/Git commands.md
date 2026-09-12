https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository

```bash
$ git status

On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed: (this means it's in staging area)
  (use "git reset HEAD <file>..." to unstage)

    new file:   README

Changes not staged for commit: (this means it's modified)
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

Suppose you remember one little change that you want to make in `CONTRIBUTING.md` before you commit it.

```console
$ git status

On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md <--- staged version (this is old)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md <--- this is for the newly modified file
```

### Viewing Your Staged and Unstaged Changes

To see what you’ve changed **but not yet staged**, type `git diff` with no other arguments:

```console
$ git diff
diff --git a/CONTRIBUTING.md b/CONTRIBUTING.md
index 8ebb991..643e24f 100644
--- a/CONTRIBUTING.md
+++ b/CONTRIBUTING.md
@@ -65,7 +65,8 @@ branch directly, things can get messy.
 Please include a nice description of your changes when you submit your PR;
 if we have to read the whole diff to figure out why you're contributing
 in the first place, you're less likely to get feedback and have your change
-merged in.
+merged in. Also, split your changes into comprehensive chunks if your patch is
+longer than a dozen lines.

 If you are starting to work on a particular area, feel free to submit a PR
 that highlights your work in progress (and note in the PR title that it's
```

**`git diff` will not show you staged changes! Only shows modified files.**

If you want to see what you’ve staged that will go into your next commit, you can use `git diff --staged`. This command compares your staged changes to your last commit.

### Removing Files

To remove a file from Git, you have to remove it from your tracked files (more accurately, remove it from your staging area) and then commit. The `git rm` command does that, and also removes the file from your working directory so you don’t see it as an untracked file the next time around.

You can follow these steps:

```bash
$ rm file.txt
$ git add file.txt (stage its deletion)
$ git commit -m "deleted file.txt"
```

`git rm file.txt` does the first 2 steps directly.


If you simply remove the file (say `CONTRIBUTING.md) from your working directory, it shows up under the “Changes not staged for commit” (that is, _unstaged_) area of your `git status` output:

```console
$ rm PROJECTS.md
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    PROJECTS.md

no changes added to commit (use "git add" and/or "git commit -a")
```

Then, if you run `git rm`, it stages the file’s removal:

```console
$ git rm PROJECTS.md
rm 'PROJECTS.md'
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    deleted:    PROJECTS.md
```

The next time you commit, the file will be gone and no longer tracked. If you modified the file or had already added it to the staging area, you must force the removal with the `-f` option. This is a safety feature to prevent accidental removal of data that hasn’t yet been recorded in a snapshot and that can’t be recovered from Git.


### Keep the file in your working tree but remove it from your staging area


```console
git rm --cached <filename>
```
Only use for **untracking files that should never have been committed in the first place**. 

##### What can go wrong: 

Say `file.txt` was **already saved in your previous commit** (e.g., `main.cpp` or `index.html`).

**Step 1: `git add file.txt`**

- You staged your local changes to `file.txt`.
    
**Step 2: `git rm --cached file.txt`**

- This completely **wipes `file.txt` out of the Staging Area**.
    
- The Staging Area now contains **zero reference** to `file.txt`.
    
**Step 3: `git commit -m "file.txt removed"`**

- Git takes whatever is in the Staging Area and saves it as the **New Commit**.
    
- Because the Staging Area had no `file.txt`, the **New Commit snapshot does not contain `file.txt`**.


Git calculates history by comparing **Commit A (Previous)** vs **Commit B (New)**:

- **Commit A**: Contains `file.txt`
    
- **Commit B**: Does **not** contain `file.txt`
    
- **Difference between A and B**: `file.txt` was deleted.
    

That difference is what gets sent when you push to GitHub or share code with a teammate.

**What goes wrong next**:

**On your teammate's computer** When your teammate runs `git pull`, Git compares Commit A with Commit B. Seeing that `file.txt` is missing in Commit B, Git executes the change and **deletes `file.txt` off their local disk entirely**.

**Using `git restore --staged file.txt`**

1. **`git add file.txt`**
    
    The Staging Area gets your **new edited version** of `file.txt`.
    
2. **`git restore --staged file.txt`**
    
    Git looks at your last commit, copies the **previous version** of `file.txt`, and replaces your edited version in the Staging Area with it.
    
    _(The Staging Area still contains `file.txt`! )_
    
3. **`git commit -m "some commit"`**
    
    Git saves the Staging Area into the new commit. Because `file.txt` is still in the Staging Area, **`file.txt` remains in your repository**. Your new local edits are not deleted; they just sit in your folder waiting for later.
    

**Direct Comparison of the Staging Area at Commit Time**

- **`git restore --staged file.txt`** → Staging Area contains `file.txt` (the clean version from your last commit).
    
- **`git rm --cached file.txt`** → Staging Area contains **no record** of `file.txt` (staged for deletion).



You can pass files, directories, and file-glob patterns to the `git rm` command. That means you can do things such as:

```console
$ git rm log/\*.log
```

Note the backslash (`\`) in front of the `*`. This is necessary because Git does its own filename expansion in addition to your shell’s filename expansion.

### Moving / Renaming Files

Unlike many other VCSs, Git doesn’t explicitly track file movement. If you rename a file in Git, no metadata is stored in Git that tells it you renamed the file.

If you want to rename a file in Git, you can run something like:

```console
$ git mv file_from file_to
```

```console
$ git mv README.md README
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed: (in staging area)
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
```

However, this is equivalent to running something like this:

```console
$ mv README.md README
$ git rm README.md
$ git add README
```

### Viewing the Commit History  - `git log`

```console
$ git log
commit ca82a6dff817ec66f44342007202690a93763949 <--- SHA1 checksum of the commit
Author: Scott Chacon <schacon@gee-mail.com> <--- who made the commit
Date:   Mon Mar 17 21:52:11 2008 -0700 <--- what time

    Change version number <--- commit message

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    Remove unnecessary test

commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 10:31:28 2008 -0700

    Initial commit
```

Useful options:

`-p` (patch): shows the difference (the _patch_ output) introduced in each commit.

```console
$ git log -p -2 <--- limit number of commits
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    Change version number

diff --git a/Rakefile b/Rakefile
index a874b73..8f94139 100644
--- a/Rakefile
+++ b/Rakefile
@@ -5,7 +5,7 @@ require 'rake/gempackagetask'
 spec = Gem::Specification.new do |s|
     s.platform  =   Gem::Platform::RUBY
     s.name      =   "simplegit"
-    s.version   =   "0.1.0"
+    s.version   =   "0.1.1"
     s.author    =   "Scott Chacon"
     s.email     =   "schacon@gee-mail.com"
     s.summary   =   "A simple gem for using Git in Ruby code."
     
...
```

`--stat` - See shorter stats
`--pretty` - changes the log output to formats other than the default.

```console
$ git log --pretty=oneline
ca82a6dff817ec66f44342007202690a93763949 Change version number
085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7 Remove unnecessary test
a11bef06a3f659402fe7563abf99ad00de2209e6 Initial commit
```

read more: https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History







