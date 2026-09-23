https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging

You are a SDE working on a project. Right now the project history looks like:

![Pasted image 20260913194434](../assets/Pasted%20image%2020260913194434.png)


You want to work on issue 53. So you create a new branch:

```console
$ git checkout -b iss53
```

![Pasted image 20260913194525](../assets/Pasted%20image%2020260913194525.png)

You work on your website and do some commits. Doing so moves the `iss53` branch forward, because you have it checked out (that is, your `HEAD` is pointing to it).

![Pasted image 20260913194611](../assets/Pasted%20image%2020260913194611.png)


Now you get the call that there is an issue with the website, and you need to fix it immediately.

So you switch back to the `master` branch.

**Note that if your working directory or staging area has uncommitted changes that conflict with the branch you’re checking out, Git won’t let you switch branches. It’s best to have a clean working state when you switch branches.**

```console
$ git checkout master
```

At this point, your project working directory is exactly the way it was before you started working on issue #53, and you can concentrate on your hotfix. 

When you switch branches, Git **resets your working directory** to look like it did the last time you committed on that branch. It adds, removes, and modifies files automatically to make sure your working copy is what the branch looked like on your last commit to it.

Now:

```console
$ git checkout -b hotfix
```

You made some changes, now commit:

```console
$ git commit -a -m 'Fix broken email address'
```


![Pasted image 20260913195001](../assets/Pasted%20image%2020260913195001.png)


Now, you merge your changes to `master` branch to deploy to prod.

```console
$ git checkout master
$ git merge hotfix

Updating f42c576..3a0874c
Fast-forward <--- what is this??
 index.html | 2 ++
 1 file changed, 2 insertions(+)

```

Because the commit `C4` pointed to by the branch `hotfix` you merged in was directly ahead of the commit `C2` you’re on, Git simply moves the pointer forward. 

When you try to merge one commit x with a commit y that can be reached by following the first commit’s (x) history, Git simplifies things by moving the pointer forward because there is no divergent work to merge together — this is called a “fast-forward.”

![Pasted image 20260913195443](../assets/Pasted%20image%2020260913195443.png)

Now you can delete the `hotfix`branch:

```console
$ git branch -d hotfix
Deleted branch hotfix (3a0874c).
```

Now you can switch back to your work-in-progress branch on issue #53 and continue working on it.


```console
$ git checkout iss53
Switched to branch "iss53"
$ vim index.html
$ git commit -a -m 'Finish the new footer [issue 53]'
[iss53 ad82d7a] Finish the new footer [issue 53]
1 file changed, 1 insertion(+)
```

![Pasted image 20260913195550](../assets/Pasted%20image%2020260913195550.png)


Suppose you’ve decided that your issue #53 work is complete and ready to be merged into your `master` branch. 

In order to do that, you’ll merge your `iss53` branch into `master`:

```console
$ git checkout master
Switched to branch 'master'
$ git merge iss53
Merge made by the 'recursive' strategy.
index.html |    1 +
1 file changed, 1 insertion(+)
```

Before:

![Pasted image 20260913195833](../assets/Pasted%20image%2020260913195833.png)

After:

![Pasted image 20260913195841](../assets/Pasted%20image%2020260913195841.png)


Instead of just moving the branch pointer forward, Git creates a new snapshot that results from this three-way merge and automatically creates a new commit that points to it. This is referred to as a **merge commit**, and is special in that it has more than one parent.


### Merge conflict

If you changed the same part of the same file differently in the two branches you’re merging, Git won’t be able to merge them cleanly. If your fix for issue #53 modified the same part of a file as the `hotfix` branch, you’ll get a merge conflict that looks something like this:

```console
$ git merge iss53
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

Git hasn’t automatically created a new merge commit. It has paused the process while you resolve the conflict. 

If you want to see which files are unmerged at any point after a merge conflict, you can run `git status`:

```console
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)

    both modified:      index.html

no changes added to commit (use "git add" and/or "git commit -a")
```

Anything that has merge conflicts and hasn’t been resolved is listed as unmerged. Git adds standard conflict-resolution markers to the files that have conflicts, so you can open them manually and resolve those conflicts.

After you’ve resolved each of these sections in each conflicted file, run `git add` on each file to mark it as resolved. **Staging the file marks it as resolved in Git.**


You can use `git mergetool` (GUI) to resolve the changes too. 

After you exit the merge tool, Git asks you if the merge was successful. If you tell the script that it was, it stages the file to mark it as resolved for you. You can run `git status` again to verify that all conflicts have been resolved:

```console
$ git status
On branch master
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:

    modified:   index.html
```

If you’re happy with that, and you verify that everything that had conflicts has been staged, you can type `git commit` to finalize the merge commit. The commit message by default looks something like this:

```console
Merge branch 'iss53'

Conflicts:
    index.html
#
# It looks like you may be committing a merge.
# If this is not correct, please remove the file
#	.git/MERGE_HEAD
# and try again.


# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# All conflicts fixed but you are still merging.
#
# Changes to be committed:
#	modified:   index.html
#
```
