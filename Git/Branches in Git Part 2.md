## Reversing Changes in Git

There are two primary ways to undo changes in Git -- one is using `git reset` and the other is using `git revert`.

## Git Reset

`git reset` reverses changes by moving a branch reference backwards in time to an older commit. In this sense you can think of it as "rewriting history;" `git reset` will move a branch backwards as if the commit had never been made in the first place.

![[Pasted image 20260813201147.png]]

```
git reset HEAD~1
```

![[Pasted image 20260813201217.png]]


## Git Revert

While resetting works great for local branches on your own machine, its method of "rewriting history" doesn't work for remote branches that others are using.

In order to reverse changes and _share_ those reversed changes with others, we need to use `git revert`.

![[Pasted image 20260813201436.png]]

```
git revert HEAD^
```

![[Pasted image 20260813201510.png]]

Example usage:

![[Pasted image 20260813201622.png]]

pushed - remote branch
local - local branch

![[Pasted image 20260813202054.png]]

How to achieve this?

```
$ git reset HEAD^
$ git checkout pushed
$ git revert HEAD
```

## Git Cherry-pick

`git cherry-pick <Commit1> <Commit2> <...>`


It's a very straightforward way of saying that you would like to copy a series of commits below your current location (`HEAD`).


Here's a repository where we have some work in branch `side` that we want to copy to `main`. This could be accomplished through a rebase (which we have already learned), but let's see how cherry-pick performs.

![[Pasted image 20260813202819.png]]

```
git cherry-pick C2 C4
```

![[Pasted image 20260813202854.png]]


Example: 

![[Pasted image 20260813202937.png]]


```
git cherry-pick C3 C4 C7
```


![[Pasted image 20260813203056.png]]


## Git Interactive Rebase

Git cherry-pick is great when you know which commits you want (_and_ you know their corresponding hashes).

But what about the situation where you don't know what commits you want?

We can use interactive rebasing for this -- it's the best way to review a series of commits you're about to rebase.

Interactive rebase means is that Git is using the `rebase` command with the `-i` option.

![[Pasted image 20260813203757.png]]

```
git rebase HEAD~4
```

![[Pasted image 20260813203814.png]]

(new order)

![[Pasted image 20260813203844.png]]


## Undoing with `git restore`

`git restore` is the modern, purpose-built undo button for your working directory and staging area.

It comes in two flavors:

- `git restore --staged <file>`: **unstage** a file (move it back out of the staging area, keeping your edits)
- `git restore <file>`: **discard** your edits to a file entirely (careful, this throws the changes away!)

_(These replace the older `git reset HEAD <file>` and `git checkout -- <file>` tricks.)_


Example:

```
Changes to be committed:
  modified:   app.js
  modified:   secret.env
Changes not staged for commit:
  modified:   experiment.js
```

You want to commit `app.js`, but `secret.env` got staged early by accident (it should be a commit on top), so lets save that for later. Also the `experiment.js` changes did not work so lets throw that out entirely.

```
$ git restore --staged secret.env
$ git restore experiment.js
$ git commit
```


## Locally stacked commits

Here's a development situation that often happens: I'm trying to track down a bug but it is quite elusive. In order to aid in my detective work, I put in a few debug commands and a few print statements.

All of these debugging / print statements are in their own commits. Finally I track down the bug, fix it, and rejoice!

Only problem is that I now need to get my `bugFix` back into the `main` branch. If I simply fast-forwarded `main`, then `main` would get all my debug statements which is undesirable. There has to be another way...


We need to tell git to copy only one of the commits over. This is just like the levels earlier on moving work around -- we can use the same commands:

- `git rebase -i`
- `git cherry-pick`

![[Pasted image 20260813205521.png]]

what to do?

```
$ git checkout main (switch to main first)
$ git cherry-pick C4
```

![[Pasted image 20260813205836.png]]


## Juggling Commits

Here's another situation that happens quite commonly. You have some changes (`newImage`) and another set of changes (`caption`) that are related, so they are stacked on top of each other in your repository (aka one after another).

The tricky thing is that sometimes you need to make a small modification to an earlier commit. In this case, design wants us to change the dimensions of `newImage` slightly, even though that commit is way back in our history!!


