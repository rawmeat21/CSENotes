Git maintains a history of which commits were made when. That's why most commits have ancestor commits above them -- we designate this with arrows in our visualization.


![[Pasted image 20260813115125.png]]


Branches in Git are incredibly lightweight, just like commits. **They are simply pointers to a specific commit**.

Because there is no storage / memory overhead with making many branches, it's easier to logically divide up your work than have big beefy branches.

**There is not much overhead in creating branches**

![[Pasted image 20260813115320.png]]

The branch `newImage` now refers to commit `C1`.

![[Pasted image 20260813115404.png]]

![[Pasted image 20260813115439.png]]

![[Pasted image 20260813115557.png]]

(Create new branch and switch to it.)

## Git merging

Now we need to learn some kind of way of combining the work from two different branches together. This will allow us to branch off, develop a new feature, and then combine it back in.

The first method to combine work that we will examine is `git merge`. Merging in Git creates a special commit that has two unique parents. A commit with two parents essentially means "I want to include all the work from this parent over here and this one over here, _and_ the set of all their parents."

![[Pasted image 20260813115940.png]]

Here we have two branches; each has one commit that's unique. This means that neither branch includes the entire set of "work" in the repository that we have done.

We will `merge` the branch `bugFix` into `main`.

```
git merge bugFix
```

![[Pasted image 20260813120017.png]]

First of all, `main` now points to a commit that has two parents. If you follow the arrows up the commit tree from `main`, you will hit every commit along the way to the root. This means that `main` contains all the work in the repository now.

Let's merge `main` into `bugFix`

```
git checkout bugFix; git merge main
```

![[Pasted image 20260813120450.png]]

Since `bugFix` was an ancestor of `main`, git didn't have to do any work; it simply just moved `bugFix` to the same commit `main` was attached to.


Example:

![[Pasted image 20260813120549.png]]

```
git checkout -b bugFix
```

![[Pasted image 20260813120840.png]]

```
git commit
```

![[Pasted image 20260813120901.png]]

```
git checkout main; git commit
```

![[Pasted image 20260813120940.png]]

```
git merge bugFix
```

![[Pasted image 20260813121029.png]]


## Git Rebase

The second way of combining work between branches is _rebasing._ Rebasing essentially takes a set of commits, "copies" them, and plops them down somewhere else.

While this sounds confusing, the advantage of rebasing is that it can be used to make a nice linear sequence of commits. The commit log / history of the repository will be a lot cleaner if only rebasing is allowed.


Here we have two branches yet again; note that the bugFix branch is currently selected (note the asterisk)

![[Pasted image 20260813121506.png]]

We would like to move our work from bugFix directly onto the work from main. **That way it would look like these two features were developed sequentially, when in reality they were developed in parallel.**

```
git rebase main
```

![[Pasted image 20260813121524.png]]


Now we are checked out on the `main` branch. Let's go ahead and rebase onto `bugFix`...

![[Pasted image 20260813121712.png]]

```
git rebase bugFix
```

Since `main` was an ancestor of `bugFix`, git simply moved the `main` branch reference forward in history.


## Moving around in Git

Before we get to some of the more advanced features of Git, it's important to understand different ways to move through the commit tree that represents your project.

### HEAD

`HEAD is the symbolic name for the currently checked out commit -- it's essentially what commit you're working on top of.

HEAD always points to the most recent commit which is reflected in the working tree. Most git commands which make changes to the working tree will start by changing HEAD.

Normally HEAD points to a branch name (like bugFix). When you commit, the status of bugFix is altered and this change is visible through HEAD.

```
$ git checkout C1
$ git checkout main
$ git commit 
$ git checkout C2
```
![[Pasted image 20260813122634.png]]

Right now, HEAD points to main branch

![[Pasted image 20260813122739.png]]

### Detaching HEAD

Detaching HEAD just means attaching it to a commit instead of a branch. This is what it looks like beforehand:

HEAD -> main -> C1


![[Pasted image 20260813122819.png]]
```
git checkout C1
```
![[Pasted image 20260813122842.png]]

And now it's

HEAD -> C1

## Relative Refs

Moving around in Git by specifying commit hashes can get a bit tedious. In the real world you won't have a nice commit tree visualization next to your terminal, so you'll have to use `git log` to see hashes.

Furthermore, hashes are usually a lot longer in the real Git world as well. For instance, the hash of the commit that introduced the previous level is `fed2da64c0efc5293610bdd892f82a58e8cbc5d8`.

The upside is that Git is smart about hashes. It only requires you to specify enough characters of the hash until it uniquely identifies the commit. So I can type `fed2` instead of the long string above.

With relative refs, you can start somewhere memorable (like the branch `bugFix` or `HEAD`) and work from there.

Relative commits are powerful, but we will introduce two simple ones here:

- Moving upwards one commit at a time with `^`
- Moving upwards a number of times with `~<num>`


Let's look at the Caret (^) operator first. Each time you append that to a ref name, you are telling Git to find the parent of the specified commit.

So saying `main^` is equivalent to "the first parent of `main`".

`main^^` is the grandparent (second-generation ancestor) of `main`

![[Pasted image 20260813123200.png]]

```
git checkout main^
```

![[Pasted image 20260813123222.png]]


You can also reference `HEAD` as a relative ref.

![[Pasted image 20260813123502.png]]

```
git checkout C3; <--- HEAD now points at C3
git checkout HEAD^; <--- Then C2
git checkout HEAD^; <--- now C1
git checkout HEAD^; <--- Finally C0
```
![[Pasted image 20260813123520.png]]


### The "~" operator

Say you want to move a lot of levels up in the commit tree. It might be tedious to type `^` several times, so Git also has the tilde (~) operator.

The tilde operator (optionally) takes in a trailing number that specifies the number of parents you would like to ascend.

![[Pasted image 20260813123728.png]]

```
git checkout HEAD~4
```

![[Pasted image 20260813123752.png]]


### Branch forcing

Relative refs can be used to move branches around. You can directly reassign a branch to a commit with the `-f` option. So something like:

`git branch -f main HEAD~3`

moves (by force) the main branch to three parents behind HEAD.

_Note: In a real git environment `git branch -f` command is not allowed for your current branch._

![[Pasted image 20260813123924.png]]

```
git branch -f main HEAD~3
```

![[Pasted image 20260813123954.png]]


