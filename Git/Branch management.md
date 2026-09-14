https://git-scm.com/book/en/v2/Git-Branching-Branch-Management

1. To see all branches:

```console
$ git branch
  iss53
* master
  testing
```

`*` means HEAD points to that branch.

2. See last commit on each branch:

```console
$ git branch -v
  iss53   93b412c Fix javascript issue
* master  7a98805 Merge branch 'iss53'
  testing 782fd34 Add scott to the author list in the readme
```


3. See which branches are already merged into the branch you’re on:

```console
$ git branch --merged
  iss53
* master
```

Branches on this list without the `*` in front of them are generally fine to delete with `git branch -d`; you’ve already incorporated their work into another branch, so you’re not going to lose anything.


4. To see all the branches that contain work you haven’t yet merged in:

```console
$ git branch --no-merged
  testing
```

You cannot delete these branches:

```console
$ git branch -d testing
error: The branch 'testing' is not fully merged.
If you are sure you want to delete it, run 'git branch -D testing'.
```


5. Changing branch name:

```console
$ git branch --move bad-branch-name corrected-branch-name
```

To have it show up on the remote:

```console
$ git push --set-upstream origin corrected-branch-name
```

But, the `bad-branch-name` isn't gone:

```console
$ git branch --all
* corrected-branch-name
  main
  remotes/origin/bad-branch-name
  remotes/origin/corrected-branch-name
  remotes/origin/main
```


To get rid of it:

```console
$ git push origin --delete bad-branch-name
```


6. Renaming `master` branch:

```console
$ git branch --move master main
```

```console
$ git push --set-upstream origin main
```

Now: 

```console
$ git branch --all
* main
  remotes/origin/HEAD -> origin/master
  remotes/origin/main
  remotes/origin/master
```

Your local `master` branch is gone, as it’s replaced with the `main` branch. The `main` branch is present on the remote. However, the old `master` branch is still present on the remote. Other collaborators will continue to use the `master` branch as the base of their work, until you make some further changes.

Now you have a few more tasks in front of you to complete the transition:

- Any projects that depend on this one will need to update their code and/or configuration.
    
- Update any test-runner configuration files.
    
- Adjust build and release scripts.
    
- Redirect settings on your repo host for things like the repo’s default branch, merge rules, and other things that match branch names.
    
- Update references to the old branch in documentation.
    
- Close or merge any pull requests that target the old branch.
    

After you’ve done all these tasks, and are certain the `main` branch performs just as the `master` branch, you can delete the `master` branch:

```console
$ git push origin --delete master
```

