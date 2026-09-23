A branch is a pointer to a commit:

![Pasted image 20260913193550](../assets/Pasted%20image%2020260913193550.png)

### Creating a New Branch

```console
$ git branch testing
```

This creates a new pointer to the **same commit you’re currently on**.

![Pasted image 20260913193640](../assets/Pasted%20image%2020260913193640.png)

How does Git know what branch you’re currently on? It keeps a special pointer called `HEAD`

![Pasted image 20260913193721](../assets/Pasted%20image%2020260913193721.png)

### Switching Branches

To switch to an existing branch, you run the `git checkout` command.

```console
$ git checkout testing
```

What does this actually do? - moves `HEAD` to point to the `testing` branch.

![Pasted image 20260913193853](../assets/Pasted%20image%2020260913193853.png)

If you commit again:

![Pasted image 20260913193930](../assets/Pasted%20image%2020260913193930.png)

Switch again:

```console
$ git checkout master
```

![Pasted image 20260913194018](../assets/Pasted%20image%2020260913194018.png)

Commit again:

![Pasted image 20260913194042](../assets/Pasted%20image%2020260913194042.png)

Why are branches cheap? - branch in Git is actually a simple file that contains the 40 character SHA-1 checksum of the commit it points to, branches are cheap to create and destroy. Creating a new branch is as quick and simple as writing 41 bytes to a file (40 characters and a newline).

