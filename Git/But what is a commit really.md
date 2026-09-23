https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell

Let’s assume that you have a directory containing three files, and you stage them all and commit. Staging the files computes a checksum for each one (the SHA-1 hash), stores that version of the file in the Git repository (_blobs_), and adds that checksum to the staging area:

```console
$ git add README test.rb LICENSE
$ git commit -m 'Initial commit'
```

When you create the commit by running `git commit`, Git checksums each subdirectory (in this case, just the root project directory) and stores them as a tree object in the Git repository. Git then creates a commit object that has the metadata and a pointer to the root project tree so it can re-create that snapshot when needed.

Your Git repository now contains 5 objects: three _blobs_ (each representing the contents of one of the three files), one _tree_ that lists the contents of the directory and specifies which file names are stored as which blobs, and one _commit_ with the pointer to that root tree and all the commit metadata.

![Pasted image 20260913193438](../assets/Pasted%20image%2020260913193438.png)

If you make some changes and commit again, the next commit stores a pointer to the commit that came immediately before it.

![Pasted image 20260913193455](../assets/Pasted%20image%2020260913193455.png)





