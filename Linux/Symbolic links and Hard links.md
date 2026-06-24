![[Pasted image 20260624233315.png]]

l -> means symbolic link (also called soft links or symlinks)

It is a reference to some other file.

Picture this scenario: A program requires the use of a shared resource of some kind con-
tained in a file named “foo,” but “foo” has frequent version changes. It would be good to
include the version number in the filename so the administrator or other interested party
could see what version of “foo” is installed. This presents a problem. If we change the
name of the shared resource, we have to track down every program that might use it and
change it to look for a new resource name every time a new version of the resource is in-
stalled. That doesn't sound like fun at all.

Here is where symbolic links save the day. Suppose we install version 2.6 of “foo,” which
has the filename “foo-2.6” and then create a symbolic link simply called “foo” that points
to “foo-2.6.” This means that when a program opens the file “foo”, it is actually opening
the file “foo-2.6”. Now everybody is happy. The programs that rely on “foo” can find it
and we can still see what actual version is installed. When it is time to upgrade to “foo-
2.7,” we just add the file to our system, delete the symbolic link “foo” and create a new
one that points to the new version. Not only does this solve the problem of the version
upgrade, but it also allows us to keep both versions on our machine. Imagine that “foo-
2.7” has a bug (damn those developers!) and we need to revert to the old version. Again,
we just delete the symbolic link pointing to the new version and create a new symbolic
link pointing to the old version.


#### ln – Create Links

##### Create a hard link

```shell
ln file.txt link.txt
```

##### Create a symlink

```shell
ln -s item link
```


#### What are inodes?

An inode is a data structure that keeps track of all the files and directories within a Linux or UNIX-based filesystem. 

So, every file and directory in a filesystem is allocated an inode, which is identified by an integer known as an “inode number”. These unique identifiers store metadata about each file and directory.

Points to note in inode :

- Every file in the system has an inode (Index Node)
- Contains all the file information except the file contents & name.
- Just like a personal ID or a passport (Without a name!)

Inodes store metadata such as the following

- Inode number
- File size
- Owner information
- Permissions
- File type
- The number of links etc.

Each inode is identified by an inode number. Therefore, when creating or copying a file, Linux assigns a different inode number to the new file. 

However, when moving a file, the inode number will only change if the file is moved to a different filesystem. This applies to directories as well.

![[Pasted image 20260624235654.png]]

The links means number of hard links. 

![[Pasted image 20260624235732.png]]

Hard links basically point to the inode for a file. When a hard link is deleted, the file will still be in memory if there are other hard links to it. So, deleting a hard link will not necessarily free up the space. 

- **Shared Identity:** Because the original file and the hard link point to the same inode, they share the exact same permissions, ownership, and file size. If you modify the content of a hard link, the original file changes too (because they share the same data blocks).
    
- **Independent Existence:** If you delete the original file, the hard link still works perfectly, and your data is safe. Linux keeps a "link count" in the inode. The actual data on the disk is only deleted when the link count drops to zero—meaning every single hard link to that file has been deleted.
    
- **Zero Extra Space:** Creating a hard link doesn't duplicate the actual file data, so it consumes virtually no additional disk space.
    
- **Limitations:** **Same Filesystem Only:** You cannot create a hard link between files on different partitions or drives because inode numbers are only unique within the same filesystem. Also, hard links are not allowed on directories.


View inode numbers:
```shell
$ ls -i original_file.txt hard_link.txt
1234567 original_file.txt
1234567 hard_link.txt
```

(They are the same)



#### Soft links

It is simply a reference to some path. When that path is gone, the soft link is a dangling reference.

Hard links point to an inode, but soft links point to the path of the file.






