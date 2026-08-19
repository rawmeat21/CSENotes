init has multiple functions, but its main goal is to make sure the system runs the right complement of services and daemons at any given time.

Some functions: 

• Setting the name of the computer
• Setting the time zone
• Checking disks with fsck
• Mounting filesystems
• Removing old files from the /tmp directory
• Configuring network interfaces
• Configuring packet filters
• Starting up other daemons and network services


### Implementations of init

Today, three very different flavors of system management processes are in wide-
spread use:

• An init styled after the init from AT&T’s System V UNIX, which we refer to as “traditional init.” This was the predominant init used on Linux systems until the debut of systemd.

• An init variant that derives from BSD UNIX and is used on most BSD based systems, including FreeBSD, OpenBSD, and NetBSD. This one is just as tried-and-true as the SysV init and has just as much claim to being called “traditional,” but for clarity we refer to it as “BSD init.” This variant is quite simple in comparison with SysV-style init. 

• A more recent contender called **systemd** which aims to be one-stop-shopping for all daemon- and state-related issues. As a consequence, systemd carves out a significantly larger territory than any historical version of init.


