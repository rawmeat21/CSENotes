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

To manually modify the sudoers file, use the visudo command, which checks to be sure no one else is editing the file, invokes an editor on it (vi, or whichever editor you specify in your EDITOR environment variable), and then verifies the syntax of the edited file before installing it. **This last step is particularly important because an invalid sudoers file might prevent you from sudoing again to fix it.**

sudo has a couple of disadvantages:

- The worst of these is that any breach in the security of a sudoer’s personal account can be equivalent to breaching the root account itself.
- `sudo`’s command logging can easily be subverted by tricks such as shell escapes from within an allowed program, or by `sudo sh` and `sudo su`. (Such commands do show up in the logs, so you’ll at least know they’ve been run.)

**sudo runs on all UNIX and Linux systems. You do need not worry about managing different solutions on different platforms.**


#### Environment managment

Many commands consult the values of environment variables and modify their behavior depending on what they find. In the case of commands run as root, this mechanism can be both a useful convenience and a potential route of attack.


For example, several commands run the program specified in your `EDITOR` environment variable to spawn a text editor. If this variable points to a hacker’s malicious program instead of an editor, it’s likely that you’ll eventually end up running that program as root!


-> To minimize this risk, sudo’s default behavior is to pass only a minimal, sanitized environment to the commands that it runs. If your site needs additional environment variables to be passed, you can whitelist them by adding them to the sudoers file’s `env_keep` list.

example:
```
Defaults		env_keep += "SSH_AUTH_SOCK"
Defaults		env_keep += "DISPLAY XAUTHORIZATION XAUTHORITY"
```

If you need to preserve an environment variable that isn’t listed in the `sudoers` file,
you can set it explicitly on the sudo command line. For example, the command

```
$ sudo EDITOR=emacs vipw
```

edits the system password file with emacs. This feature has some potential restric-
tions, but they’re waived for users who can run ALL commands.

### `sudo` without passwords?

don't do it :)

For more, see the sudo section on the sysadmin book. (chapter 3)

### How to disable root account?

You can disable root logins entirely by setting root’s encrypted password to * or to some other fixed, arbitrary string. 

On Linux, `passwd -l` “locks” an account by prepending a ! to the encrypted password, with equivalent results.

The * and the ! are just conventions; no software checks for them explicitly. Their effect derives from their **not being valid password hashes.** As a result, attempts to verify root’s password simply fail.


### System accounts other than root

You can identify these accounts by their low UIDs, usually less than 100. 

Most often, UIDs under 10 are system accounts, and UIDs between 10 and 100 are pseudo-users associated with specific pieces of software.

**Better to replace the encrypted password field of these special users in the**
**`shadow` or `master.passwd` file with a * so that their accounts cannot be logged**
**in to. Their shells should be set to `/bin/false` or `/bin/nologin` as well, to protect**
**against remote login exploits that use password alternatives such as SSH key files.**

There are also system-related groups that have similarly low GIDs.

-> Files and processes that are part of the operating system but that need not be owned
by root are sometimes assigned to the users bin or daemon.

-> The main advantage of defining pseudo-accounts and pseudo-groups is that they can be used more safely than the root account to provide access to defined groups of resources. 

For example, databases often implement elaborate access control systems of their own. From the perspective of the kernel, they run as a pseudo-user such as “mysql” that owns all database-related resources.

-> The Network File System (NFS) uses an account called “nobody” to represent root users on other systems.


![[Pasted image 20260829002740.png]]

