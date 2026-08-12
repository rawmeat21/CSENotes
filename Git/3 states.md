Git has three main states that your files can reside in: _modified_, _staged_, and _committed_:

- Modified means that you have changed the file but have not committed it to your database yet.
    
- Staged means that you have marked a modified file in its current version to go into your next commit snapshot.
    
- Committed means that the data is safely stored in your local database.
    

This leads us to the three main sections of a Git project: the working tree, the staging area, and the Git directory.


![[Pasted image 20260812102434.png]]


The working tree is a single checkout of one version of the project. These files are pulled out of the compressed database in the Git directory and placed on disk for you to use or modify.

The staging area is a file, generally contained in your Git directory, that stores information about what will go into your next commit.

The Git directory is where Git stores the metadata and object database for your project. This is the most important part of Git, and it is what is copied when you _clone_ a repository from another computer.

The basic Git workflow goes something like this:

1. You modify files in your working tree.
    
2. You selectively stage just those changes you want to be part of your next commit, which adds _only_ those changes to the staging area.
    
3. You do a commit, which takes the files as they are in the staging area and stores that snapshot permanently to your Git directory.
    

If a particular version of a file is in the Git directory, it’s considered _committed_. If it has been modified and was added to the staging area, it is _staged_. And if it was changed since it was checked out but has not been staged, it is _modified_.