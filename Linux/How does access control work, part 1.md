Some rules:

-  Access control decisions depend on **which user is attempting to perform an operation**, or in some cases, on that user’s membership in a UNIX group.
-  Objects (e.g., files and processes) have owners. Owners have broad (not necessarily unrestricted) control over their objects.
-  You own the objects you create.
-  The special user account called “root” can act as the owner of any object.
-  Only root can perform certain sensitive administrative operations.

![[Pasted image 20260827140912.png]]

 -> Groups are traditionally defined in the `/etc/group` file, but these days group information is often stored in a network database system such as LDAP.

-> The owner of a file gets to specify what the group owners can do with it. This scheme allows files to be shared among members of the same project.

-> Both the kernel and the filesystem track owners and groups as numbers rather than as text names. In the most basic case, **user identification numbers (UIDs for short)** are mapped to usernames in the `/etc/passwd` file, and **group identification numbers (GIDs)** are mapped to group names in `/etc/group`.

-> The owner of a process can send the process signals and can also reduce (degrade) the process’s scheduling priority.

-> Processes have multiple identities associated with them: a real, effective, and saved UID; a real, effective, and saved GID; and under Linux, a “filesystem UID” that is used only to determine file access permissions.

-> The defining characteristic of the root account is its UID of 0. Traditional UNIX allows the superuser (that is, any process for which the effective UID is 0) to perform any valid operation on any file or process. Don't make a user have UID = 0!

-> A process that is owned by root can change its UID and GID. This is how the `login` program works btw.

### SetUID and SetGID

When the kernel runs an executable file that has its “setuid” or “setgid” permission bits set, it changes the effective UID or GID of the resulting process to the UID or GID of the file containing the program image rather than the UID and GID of the user that ran the command. 

**The user’s privileges are thus promoted for the execution of that specific command only.

For example, users must be able to change their passwords. But since passwords are (traditionally) stored in the protected `/etc/master.passwd` or `/etc/shadow` file, users need a setuid `passwd` command to mediate their access. 

The `passwd` command checks to see who’s running it and customizes its behavior accordingly: users can change only their own passwords, but root can change any password.

**Programs that run setuid, especially ones that run setuid to root, are prone to secu-
rity problems.

**You can disable setuid and setgid execution on individual filesystems by specifying
the `nosuid` option to `mount`.


### Don't do things as root

-  root logins leave no record of what operations were performed as root. That’s bad enough when you realize that you broke something last night at 3:00 a.m. and can’t remember what you changed.
- log-in-as-root scenario leaves no record of who was actually doing the work. If several people have access to the root account, you won’t be able to tell who used it and when.

### `su` (substitute user identity)

-  If invoked without arguments, su prompts for the root password and then starts up a root shell.
- Root privileges remain in effect until you terminate the shell by typing `<Control-D>` or the exit command.
-  su doesn’t record the commands executed as root, but it does create a log entry that states who became root and when.
- If you know someone’s password, you can access that person’s account directly by executing `su - username`. `-` makes su spawn the shell in login mode. The exact implications of login mode vary by shell, but login mode normally changes the number or identity of the files that the shell reads when it starts up. For example, bash reads `~/.bash_profile` in login mode and `~/.bashrc` in nonlogin mode. **When diagnosing other users’ problems, it helps to reproduce their login environments as closely as possible by running in login mode.**
- On most systems, you must be a member of the group “wheel” to use `su`.

### `sudo`

`sudo` takes as its argument a command line to be executed as root (or as another
restricted user). sudo consults the file` /etc/sudoers` (`/usr/local/etc/sudoers` on
FreeBSD), which lists the people who are authorized to use sudo and the commands
they are allowed to run on each host. 

If the proposed command is permitted, sudo prompts for the user’s own password and executes the command.


`sudo` keeps a log of the command lines that were executed, the hosts on which they were run, the people who ran them, the directories from which they were run, and the times at which they were invoked. This information can be logged by `syslog` or placed in the file of your choice.

![[Pasted image 20260827145902.png]]


**`sudoers` file:**

![[Pasted image 20260827145942.png]]

