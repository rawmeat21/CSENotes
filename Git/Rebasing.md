https://git-scm.com/book/en/v2/Git-Branching-Rebasing

![Pasted image 20260914223757](../assets/Pasted%20image%2020260914223757.png)

After a merge:

![Pasted image 20260914223809](../assets/Pasted%20image%2020260914223809.png)

However, there is another way: you can take the patch of the change that was introduced in `C4` and reapply it on top of `C3`. In Git, this is called _rebasing_. 

With the `rebase` command, you can take all the changes that were committed on one branch and replay them on a different branch.

For this example, you would check out the `experiment` branch, and then **rebase it onto** the `master` branch as follows:

```console
$ git checkout experiment
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: added staged command
```

![Pasted image 20260914223922](../assets/Pasted%20image%2020260914223922.png)

If you do:

```console
$ git checkout master
$ git merge experiment
```

Result:

![Pasted image 20260914223955](../assets/Pasted%20image%2020260914223955.png)

Now, the snapshot pointed to by `C4'` is **exactly the same** as the one that was pointed to by `C5`.There is no difference in the end product of the integration, but rebasing makes for a cleaner history. 

If you examine the log of a rebased branch, it looks like a linear history: it appears that all the work happened in series, even when it originally happened in parallel.



### Rebasing example

Say you have this:

![Pasted image 20260914224413](../assets/Pasted%20image%2020260914224413.png)


Suppose you decide that you want to merge your client-side changes into your mainline for a release, but you want to hold off on the server-side changes until it’s tested further. 

You can take the changes on `client` that aren’t on `server` (`C8` and `C9`) and replay them on your `master` branch by using the `--onto` option of `git rebase`:

```console
$ git rebase --onto master server client
```

This basically says, “Take the `client` branch, figure out the patches since it diverged from the `server` branch, and replay these patches (`C8` and `C9`) in the `client` branch as if it was based directly off the `master` branch instead.”

![Pasted image 20260914231045](../assets/Pasted%20image%2020260914231045.png)

Fast forward `master`:

```console
$ git checkout master
$ git merge client
```

![Pasted image 20260914231114](../assets/Pasted%20image%2020260914231114.png)

You can rebase the `server` branch onto the `master` branch without having to check it out first by running `git rebase <basebranch> <topicbranch>`, which checks out the topic branch (in this case, `server`) for you and replays it onto the base branch (`master`):

![Pasted image 20260914231209](../assets/Pasted%20image%2020260914231209.png)

Now:

```console
$ git checkout master
$ git merge server
```

Remove the `server` and `client` branches:

```console
$ git branch -d client
$ git branch -d server
```

![Pasted image 20260914231301](../assets/Pasted%20image%2020260914231301.png)



### Where to not use rebase?

**Do not rebase commits that exist outside your repository and that people may have based work on.**

#### An example

https://git-scm.com/book/en/v2/Git-Branching-Rebasing