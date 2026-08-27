### systemd logging

systemd has a universal logging framework that includes all kernel and service messages from early boot to final shutdown. This facility, called the **journal**, is managed by the journald daemon.

-> System messages captured by journald are stored in the `/run` directory. `rsyslog` can process these messages and store them in traditional log files or forward them to a remote syslog server. You can also access the logs directly with the **journalctl** command.


Without arguments, `journalctl` displays all log entries (oldest first).

#### Configure `journald` to retain messages from prior boots

To do this, edit `/etc/systemd/journald.conf` and configure the Storage attribute: 

```
[Journal]
Storage=persistent
```

To get list of prior boots:

```bash
$ journalctl --list-boots
```

Output:

```
-1 a73415fade0e4e7f4bea60913883d180dc Fri 2016-02-26 15:01:25 UTC
Fri 2016-02-26 15:05:16 UTC
0 0c563fa3830047ecaa2d2b053d4e661d Fri 2016-02-26 15:11:03 UTC Fri
2016-02-26 15:12:28 UTC
```

To access msgs from a prior boot, refer by it's ID:

```bash
$ journalctl -b -1
$ journalctl -b a73415fade0e4e7f4bea60913883d180dc
```


**To restrict the logs to those associated with a specific unit, use the -u flag:**

```bash
$ journalctl -u ntp
```

```
-- Logs begin at Fri 2016-02-26 15:11:03 UTC, end at Fri 2016-02-26
15:26:07 UTC. --
Feb 26 15:11:07 ub-test-1 systemd[1]: Stopped LSB: Start NTP daemon.
Feb 26 15:11:08 ub-test-1 systemd[1]: Starting LSB: Start NTP daemon...
Feb 26 15:11:08 ub-test-1 ntp[761]: * Starting NTP server ntpd
...
```

### Single user mode

Think of your computer's boot process as going through stages, from "just the bare minimum" up to "fully running with everything on." Single-user mode is one of the earliest, most stripped-down stages.

### What it actually is

Normally, when your Linux/Unix machine boots, it starts a huge number of things: network services, login screens, background daemons, cron jobs, etc. **Single-user mode skips almost all of that.** You get:

- The root filesystem mounted (and usually `/usr` too)
- A root shell — you're logged in as root, no password prompt for regular use
- **No networking** — the network stack isn't even brought up
- No other users can log in — hence "single-user"
- Most services and daemons never start


On systemd-based systems (most modern distros — Ubuntu, Fedora, etc.), this same concept is called `rescue.target`.

### Why would you ever want this?

It's a rescue/repair mode. You use it when something is broken enough that a normal boot won't work cleanly, or when you need to make risky changes without other processes interfering. Common cases:

- Fixing a corrupted config file that's breaking normal boot
- Repairing a filesystem
- Undoing a bad update or kernel change
- Editing something in `/etc` that requires no other process to be touching it

### How you get into it

**At boot time:** you interrupt the normal boot and pass an extra option to the kernel — usually `single` or `-s`. You'd do this from your bootloader's menu (like GRUB) by editing the boot line temporarily.

**From a running system:** if the machine is already up, you can drop down _into_ single-user mode with a command:

- `systemctl rescue` on systemd systems
- `telinit 1` on older/traditional init systems

### An important catch: the password prompt

Well-configured ("sane") systems will actually ask for the **root password** before giving you the single-user shell. This is a security measure — otherwise anyone with physical access could reboot your machine and get instant root access.

The catch: this makes single-user mode useless for password recovery in the normal case, since you need the root password to get in. If you've genuinely forgotten root's password, you can't use single-user mode to reset it — you'd need to boot from separate media (a USB installer/live disk) instead.

### What you can and can't do once you're in

You get a shell and can run most commands like normal. But there are two catches:

**1. Limited filesystems are mounted.** Usually only the root partition is mounted automatically. If a program you need lives outside `/bin`, `/sbin`, or `/etc` (e.g., on a separate `/home` or `/var` partition), you'll need to mount that filesystem manually first. You can check `/etc/fstab` to see what filesystems exist and where they normally mount. On Linux, `fdisk -l` (list) shows you the disk partitions on the system.

**2. Root filesystem is often mounted read-only.** This is the big one for beginners. If you try to edit a file (like a config in `/etc`) and get a permission error even as root, it's because the filesystem itself is mounted read-only, not writable. You need to explicitly remount it as read/write:

bash

```bash
mount -o rw,remount /
```

That command tells the system: "take the filesystem that's already mounted at `/`, and remount it, but this time with read/write permissions instead of read-only."


**The `fsck` command is run during a normal boot to check and repair filesystems.
Depending on what filesystem you’re using for the root, you may need to run `fsck`
manually when you bring the system up in single-user or emergency mode.

