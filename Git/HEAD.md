HEAD is a pointer to a branch or a commit. YES it can point to both.

### HEAD pointing at a branch vs at a commit

Consider 2 cases, HEAD points to a commit C2 vs HEAD points to a branch "my-branch" which points to  C2. What is the difference in what I would see my working directory to be?

First: There is **zero difference in your Working Directory**. In both cases, Git resolves the tree object associated with commit `C2`, updates `.git/index` with `C2`'s file manifest, and writes those exact files to disk.

The differences exist entirely in how Git handles references and future mutations:

**1. Internal `.git/HEAD` File Content**

- **Detached `HEAD` (`HEAD -> C2`)**: `.git/HEAD` contains the literal 40-character hash of `C2` (e.g., `6ffb4f2304905174a197900540bcce1...`).
    
- **Attached `HEAD` (`HEAD -> my-branch -> C2`)**: `.git/HEAD` contains a path string: `ref: refs/heads/my-branch`.
    

**2. State Mutation on `git commit`**

- **Detached `HEAD`**: When you run `git commit`, Git constructs new commit object `C3` (with parent `C2`) and overwrites `.git/HEAD` directly with `C3`'s hash. The branch file `.git/refs/heads/my-branch` is not dereferenced and remains stuck pointing to `C2`. Here, **`my-branch` still points at `C2`**.

- **Attached `HEAD`**: When you run `git commit`, Git creates `C3`, reads the `ref:` inside `.git/HEAD`, and updates `.git/refs/heads/my-branch` to hold `C3`'s hash. `HEAD` remains pointed to `my-branch`. So, here, **`my-branch` changes to point at `C3`**.
    

**3. Garbage Collection & Reachability (Dangling Commits)**

- **Detached `HEAD`**: If you create commits `C3` and `C4` while detached and then run `git checkout main`, no named reference in `.git/refs/` points to `C4`. `C3` and `C4` become **unreachable (dangling) objects**. Unless you explicitly create a branch pointing to them, Git's garbage collection (`git gc`) will permanently prune them after `gc.reflogExpireUnreachable` expires (default 30 days).

```
[main] ─────────────────────► C1 ◄─── C2 ◄─── C3 ◄─── C4
                                        ▲ 
 [my-branch] ───────────────────────────┘
```
In Git’s DAG structure, **directed edges point backwards**, from child commit to parent commit (C4→C3→C2→C1). Parent commits contain zero reference pointers to their children.

So, once you checkout, you lose the reference to `C3` and `C4`.

- **Attached `HEAD`**: Every commit created is immediately linked to `refs/heads/my-branch`, keeping the commit graph reachability chain intact indefinitely.
    

**4. Tooling and Status Output**

- Running `git status` when detached reports: `HEAD detached at <hash>`.
    
- Running `git status` when attached reports: `On branch my-branch`.

**Walkthrough: Empty Repo to Attached vs. Detached HEAD**


1. Initialize an empty repository:


```
mkdir git_demo && cd git_demo
git init
```

- **`.git/HEAD` state**: Stores `ref: refs/heads/main` (or `master`). The branch pointer `main` does not physically exist yet because no commits have been written.


2. Create and commit C1 (Attached State)

```
echo "version 1" > file.txt
git add file.txt
git commit -m "C1"
```

- **`.git/HEAD` state**: Still stores `ref: refs/heads/main`.
    
- **`.git/refs/heads/main` state**: Created automatically, stores hash `a1b2c3d` (Commit C1).
    
- **`git status` output**:

```
On branch main
nothing to commit, working tree clean
```

3. Create C2 (Attached State)

Bash

```
echo "version 2" >> file.txt
git commit -am "C2"
```

- **`.git/refs/heads/main` state**: Updates from `a1b2c3d` to `e5f6g7h` (Commit C2). So, main moved!
    
- **`.git/HEAD` state**: Unchanged (`ref: refs/heads/main`).


4. Switch to Detached HEAD State

Checkout Commit C1 directly using its hash:

```
git checkout a1b2c3d
```

- **`.git/HEAD` state**: Git overwrites `.git/HEAD` to directly contain the raw hash `a1b2c3d` instead of `ref: refs/heads/main`. Now, `HEAD` points to `C1`, which is a commit.
    
- **Working Directory**: `file.txt` is updated on disk to contain only `"version 1"`.
    
- **`git status` output**:

```
HEAD detached at a1b2c3d (<--- see the difference?)
nothing to commit, working tree clean
```

5. Create C3 while Detached

```
echo "version 3 experimental" >> file.txt
git commit -am "C3"
```

- **`.git/HEAD` state**: Overwritten with new hash `9i8u7y6` (Commit C3).
    
- **`.git/refs/heads/main` state**: Remains unchanged at `e5f6g7h` (Commit C2).
    
- **`git status` output**:
```
HEAD detached from a1b2c3d
nothing to commit, working tree clean
```

6. Return to attached main branch

```
git checkout main
```

- **`.git/HEAD` state**: Replaced back to `ref: refs/heads/main`.
    
- **Result**: `HEAD` points to `main` (C2). Commit C3 (`9i8u7y6`) now has no branch pointing to it. It is detached and unreachable unless you inspect `git reflog` to recover its hash.