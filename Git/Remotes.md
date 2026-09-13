## Basics

### Adding Remote Repositories

```bash
$ git remote add pb https://github.com/paulboone/ticgit
```
`pb` is shortname for server (like you have `origin`)

```console
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
pb	https://github.com/paulboone/ticgit (fetch)
pb	https://github.com/paulboone/ticgit (push)
```

## Fetching from remote

To fetch from a remote:

```console
$ git fetch <remote>
```

The command goes out to that remote project and pulls down all the data from that remote project that you don’t have yet. After you do this, you should have references to all the branches from that remote, which you can merge in or inspect at any time.

If you clone a repository, the command automatically adds that remote repository under the name “origin”. So, `git fetch origin` fetches any new work that has been pushed to that server since you cloned (or last fetched from) it. 

It’s important to note that the `git fetch` command only downloads the data to your local repository, it doesn’t automatically merge it with any of your work or modify what you’re currently working on. You have to merge it manually into your work when you’re ready.


Example: suppose we did: 

```
$ git fetch pb
```


```
                  (C4) <--- (C5)  <-- pb/master (Paul's branch pointer)
                 /
(C1) <--- (C2) <--- (C3)  <-- main (Your local branch pointer)
            ^
            |
    (Common Ancestor)

```

Both `main` and `pb/master` exist in the **exact same graph**:

- **`C1`, `C2`, `C3`**: Commits you have locally.
    
- **`C4`, `C5`**: New commits downloaded from Paul (`pb`).
    
- **`pb/master`**: A pointer sitting on commit `C5`.
    
- **`main`**: A pointer sitting on commit `C3`.


If your current branch is set up to track a remote branch, you can use the `git pull` command to automatically fetch and then merge that remote branch into your current branch.

By default, the `git clone` command automatically sets up your local `master` branch to track the remote `master` branch (or whatever the default branch is called) on the server you cloned from. Running `git pull` generally fetches data from the server you originally cloned from and automatically tries to merge it into the code you’re currently working on.


### Pushing to remote

```
git push <remote> <branch>
```
`git push <remote> <branch>` transfers **missing commit objects** to the remote server's object database and advances the remote branch reference pointer from commit C to your local commit C′. 

Basically, a branch points to a commit so it updates the server's commit version C with your changed local version C'.

**Does `HEAD` Position Matter?**

No. When you explicitly pass the `<branch>` argument (e.g., `git push origin dev`), Git inspects the reference file `.git/refs/heads/dev` directly.

- `HEAD` being attached to `main` is completely ignored during this command.
    
- You can push any local branch to a remote target without checking it out first.
    
- `HEAD` is only used if you execute a bare `git push` with no arguments, in which case Git defaults to pushing the current branch `HEAD` is attached to (based on your `push.default` configuration).


-> This command works only if you cloned from a server to which you have write access and if nobody has pushed in the meantime. 

-> If you and someone else clone at the same time and they push upstream and then you push upstream, your push will rightly be rejected. You’ll have to fetch their work first and incorporate it into yours before you’ll be allowed to push.

### Inspecting a Remote

If you want to see more information about a particular remote, you can use the `git remote show <remote>` command.

```console
$ git remote show origin
* remote origin
  Fetch URL: https://github.com/schacon/ticgit
  Push  URL: https://github.com/schacon/ticgit
  HEAD branch: master
  Remote branches:
    master                               tracked
    dev-branch                           tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```


### Renaming and Removing Remotes

Rename:
```console
$ git remote rename pb paul
$ git remote
origin
paul
```

Remove:
```console
$ git remote remove paul
$ git remote
origin
```

### Remote example because I work alone

**The Scenario: Open-Source Forking Workflow**

- **Alice**: Maintainer of the central, canonical repository (`[https://github.com/acme/app.git](https://github.com/acme/app.git)`).
    
- **Bob**: Contributor working on a bug fix.
    
- **Charlie**: Contributor working on a major feature.
    

Because Bob and Charlie do not have write access to Alice's main repository, they each fork `acme/app.git` into their own GitHub accounts (`bob/app.git` and `charlie/app.git`).

Here is how **Bob** manages multiple remotes from his terminal to collaborate with Alice and Charlie using every command from Git SCM Chapter 2.5.

1

Checking Existing Remotes

git remote & git remote -v

Bob clones his personal fork onto his machine. By default, Git names his fork `origin`.

Bash

```
git clone https://github.com/bob/app.git
cd app

# List remote shortnames
git remote
# Output: origin

# List remote shortnames alongside their fetch/push URLs
git remote -v
# Output:
# origin  https://github.com/bob/app.git (fetch)
# origin  https://github.com/bob/app.git (push)
```

2

Adding Central & Teammate Remotes

git remote add

Bob needs to pull updates from Alice's canonical repo. He adds it as a remote named `upstream`.

Bash

```
git remote add upstream https://github.com/acme/app.git
```

Charlie asks Bob to test a feature branch sitting in Charlie's personal fork before opening a Pull Request. Bob adds Charlie's fork directly as a third remote named `charlie`:

Bash

```
git remote add charlie https://github.com/charlie/app.git
```

Now Bob's local repository tracks **three distinct remote repositories**:

Bash

```
git remote -v
# Output:
# origin    https://github.com/bob/app.git (fetch/push)      <- Bob's personal fork
# upstream  https://github.com/acme/app.git (fetch/push)     <- Alice's main project
# charlie   https://github.com/charlie/app.git (fetch/push)  <- Charlie's fork
```

3

Inspecting Remote Details

git remote show

Before fetching from Charlie, Bob wants to see what branches exist on Charlie's remote and how his local configuration tracks them:

Bash

```
git remote show charlie
```

_Output:_

Plaintext

```
* remote charlie
  Fetch URL: https://github.com/charlie/app.git
  Push  URL: https://github.com/charlie/app.git
  HEAD branch: main
  Remote branches:
    main             new (has not been fetched)
    speed-experiment new (has not been fetched)
```

4

Fetching Changes from Remotes

git fetch

Bob downloads Alice's latest updates and Charlie's experiment into his local `.git/objects` database without modifying his working disk:

Bash

```
# Fetch latest commits from Alice's repo
git fetch upstream

# Fetch Charlie's branches into remote-tracking references (charlie/speed-experiment)
git fetch charlie
```

Bob can now inspect Charlie's experimental code locally:

Bash

```
git checkout charlie/speed-experiment
```

_(Note: This puts Bob in a Detached `HEAD` state pointing to Charlie's commit node inside Bob's unified commit graph.)_

5

Renaming a Remote

git remote rename

Bob decides that `charlie` is too vague and wants to rename the remote shortname to `charlie-dev`:

Bash

```
git remote rename charlie charlie-dev
```

Git renames all corresponding remote-tracking branches automatically. `charlie/speed-experiment` becomes `charlie-dev/speed-experiment`.

6

Pushing Work to a Remote

git push

Bob switches back to his local branch `fix-bug` and completes his work. He pushes his local branch up to his own fork (`origin`):

Bash

```
git checkout fix-bug
git push origin fix-bug
```

This creates `refs/heads/fix-bug` on `bob/app.git`. Bob then opens a Pull Request on GitHub asking Alice to pull `bob/app.git:fix-bug` into `acme/app.git:main`.

7

Removing an Unneeded Remote

git remote remove

Alice merges Charlie's code into `upstream/main`. Bob no longer needs a direct reference to Charlie's personal repository, so he deletes the remote reference:

Bash

```
git remote remove charlie-dev
```

This deletes the remote shortname and cleans up all `refs/remotes/charlie-dev/*` tracking pointers from Bob's repository.