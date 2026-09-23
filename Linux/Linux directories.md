![Pasted image 20260624233123](../assets/Pasted%20image%2020260624233123.png)
![Pasted image 20260624233141](../assets/Pasted%20image%2020260624233141.png)
![Pasted image 20260624233203](../assets/Pasted%20image%2020260624233203.png)
![Pasted image 20260624233214](../assets/Pasted%20image%2020260624233214.png)
![Pasted image 20260624233223](../assets/Pasted%20image%2020260624233223.png)


Let's start. In Linux, everything is a file. Like:

- Processes
- Audio devices
- Kernel data structures and tuning parameters
- Interprocess communication channels

Yep.

What is a filesystem?

The filesystem can be thought of as comprising four main components:

• A namespace – a way to name things and organize them in a hierarchy
• An API – a set of system calls for navigating and manipulating objects
• Security models – schemes for protecting, hiding, and sharing things
• An implementation – software to tie the logical model to the hardware


Some filesystems live on disk partitions or on logical volumes backed by physical disks, but filesystems can be anything that obeys the proper API: a network file server, a kernel component, a memory-based disk emulator, etc.

## `mount`

**Filesystems are attached to the tree with the `mount` command.**

`mount` maps a directory within the existing file tree, **called the mount point**, to the
root of the newly attached filesystem. 

The previous contents of the mount point become temporarily inaccessible as long as another filesystem is mounted there.

Mount points are usually empty directories, however.


```
sudo mount /dev/sda4 /users
```

installs the filesystem stored on the disk partition represented by `/dev/sda4` under
the path `/users`. You could then use `ls /users` to see that filesystem’s contents.

You can run the mount command without any arguments to see all the filesystems
that are currently mounted.

The `/etc/fstab` file contains information about the filesystems mounted on the system. This is a very useful file!

The information in this file allows filesystems to be automatically checked (with `fsck`) and mounted (with `mount`) at boot time, with options you specify.


To detach filesystems, use `umount`.

-> `umount` complains if you try to unmount a filesystem that’s in use. The filesystem to be detached must not have open files or processes whose current directories are located there, and if the filesystem contains executable programs, none of them can be running.

-> There is `umount -f` (force unmount). However, it’s almost always a bad idea to use it on non-NFS mounts, and it may not work on certain types of filesystems (e.g., those that keep journals, such as XFS or ext4). 

Instead of using this, use `fuser`.

`fuser -c mountpoint` prints the PID of every process that’s using a file or directory on that filesystem, plus a series of letter codes that show the nature of the activity. 

For example,

```
freebsd$ fuser -c /usr/home
/usr/home: 15897c 87787c 67124x 11201x 11199x 11198x 972x
```

In this example from a FreeBSD system, 

-> c indicates that a process has its current working directory on the filesystem.

-> x indicates a program being executed. However, the details are usually unimportant, the PIDs are what you want.

A more elaborate alternative to `fuser` is the `lsof` utility.


Under Linux, scripts in search of specific information about processes’ use of filesystems can also read the files in `/proc` directly. However, `lsof -F`, which formats lsof’s output for easy parsing, is an easier and more portable solution. Use additional command-line flags to request just the information you need.

