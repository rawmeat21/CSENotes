

1. See the blob hashes of each file being tracked:

```
git ls-files --stage
```

Remember: index isn’t “the list of changes to commit”. It’s a complete snapshot of what the _next commit will look like_. So, you will see all files that will go in the next commit when you run this.

A blob is like a representation of a file.

2. View contents of a blob hash:

```
git cat-file -p <blob-hash>
```

3. See how the blob file being tracked is different from the working tree version:

```
git diff <blob-hash> -- path/to/file
```

4. **Compare against a file in another commit/branch:** If you know the file path in a specific commit or branch, you don't need the exact Blob hash. You can use the `<ref>:<path>` syntax:

```
git diff main:src/app.js -- src/app.js
```

5. **Compare against a file staged in the Index:** To diff the staged version of a file (the index Blob) against the working directory version:

```
git diff -- src/app.js
```

6. Important table:

| Comparison Type                           | Command                                             | What it Shows                                                                                  | Typical Use Case                                                 |
| ----------------------------------------- | --------------------------------------------------- | ---------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| **Working Tree vs. Index (Staging area)** | `git diff`                                          | All modifications in your local files that **have not been staged** (`git add`) yet.           | "What did I just type that isn't ready to be committed?"         |
| **Index (Staging area) vs. Last Commit**  | `git diff --staged`  <br>_(or `git diff --cached`)_ | All modifications that **have been staged** and are locked in for the next commit.             | "What exactly am I about to commit when I run `git commit`?"     |
| **Working Tree vs. Last Commit**          | `git diff HEAD`                                     | **All changes** made since the last commit, regardless of whether they are staged or unstaged. | "What has changed overall in my workspace since my last commit?" |

7. How to know what **commit** HEAD points to?

```
❯ git rev-parse HEAD           
9fc33757b0c0fefa6afdb8fe6bf4e565f9ff4e06
```
This works for both attached and detached HEAD.

8. How to know what HEAD points to?

```
cat .git/HEAD
```

9. You want to move branch `my-work` to point to another commit. What to do?

```
git switch my-work (or git checkout my-work)
git reset <commit-hash> (use --hard option if you had uncommitted changes on that commit)
```
10. You want to compare the working directory with the branch you're trying to switch to. What do you do?

```
git diff <branch-name>
```

If it shows no differences, that means the working directory is same as the branch.

If it does show differences, that means the contents of working directory are different. Remember to commit uncommitted changes in the working directory if this is the case.