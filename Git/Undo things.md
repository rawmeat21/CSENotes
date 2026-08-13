https://git-scm.com/book/en/v2/Git-Basics-Undoing-Things

```console
$ git commit --amend
```

what is does: it replaces the last commit with the commit from current staging area

example: If you commit and then realize you forgot to stage the changes in a file you wanted to add to this commit, you can do something like this:

```console
$ git commit -m 'Initial commit'
$ git add forgotten_file
$ git commit --amend
```

You end up with a single commit — the second commit replaces the results of the first.

The obvious value to amending commits is to make minor improvements to your last commit, without cluttering your repository history with commit messages of the form, “Oops, forgot to add a file” or “Darn, fixing a typo in last commit”.


### Unstaging a Staged File

```console
$ git reset HEAD <filename>
```

### Unmodifying a Modified File

What if you realize that you don’t want to keep your changes to the `CONTRIBUTING.md` file? How can you easily unmodify it — revert it back to what it looked like when you last committed (or initially cloned, or however you got it into your working directory)?

```console
$ git checkout -- CONTRIBUTING.md
```

**It’s important to understand that `git checkout -- <file>` is a dangerous command. Any local changes you made to that file are gone — Git just replaced that file with the last staged or committed version. Don’t ever use this command unless you absolutely know that you don’t want those unsaved local changes.**


### `git restore` (new alt for git reset)


#### Unstaging a Staged File with git restore

```console
$ git restore --staged CONTRIBUTING.md
```
#### Unmodifying a Modified File with git restore


```console
$ git restore CONTRIBUTING.md
```








