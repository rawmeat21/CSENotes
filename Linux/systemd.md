systemd is not a single daemon but a collection of programs, daemons, libraries, technologies, and kernel components.

### Units and unit files

An entity that is managed by systemd is known generically as a unit.

A unit can be a service, a socket, a device, a mount point, an automount point, a swap file or partition, a startup target, a watched filesystem path, a timer controlled and supervised by systemd, a resource management slice, a group of externally created processes.

Within systemd, the behavior of each unit is defined and configured by a **unit file**. In the case of a service, for example, the unit file specifies the location of the executable file for the daemon, tells systemd how to start and stop the service, and identifies any other units that the service depends on.

#### Example: 

This unit file is rsync.service; it handles startup of the rsync daemon that mirrors filesystems.

```
[Unit]
Description=fast remote file copy program daemon
ConditionPathExists=/etc/rsyncd.conf
[Service]
ExecStart=/usr/bin/rsync --daemon --no-detach
[Install]
WantedBy=multi-user.target
```

### Where are my unit files?

`/usr/lib/systemd/system` is the main place where packages deposit their unit files during installation; on some systems, the path is `/lib/systemd/system` instead.

Your local unit files and customizations can go in `/etc/systemd/system`. There’s also a unit directory in `/run/systemd/system` that’s a scratch area for transient units.

**If there’s any conflict, the files in /etc have the highest priority.**



**By convention, unit files are named with a suffix that varies according to the type
of unit being configured. For example, service units have a .service suffix and tim-
ers use .timer. 

```
sshd.service     -> a service (a program to run)
home.mount       -> a mount point
tmp.timer        -> a scheduled/recurring job
sshd.socket      -> a network socket
multi-user.target -> a "target"
```

## systemctl

Used for managing systemd (checking status, modifying config).

```bash
$ systemctl
```

This invokes the default **list-units subcommand**, which shows all loaded and active services, sockets, targets, mounts, and devices. To show only loaded and active services, use the `--type=service` qualifier

```bash
$ systemctl list-units --type=service
```

To see all installed unit files:

```bash
$ systemctl list-unit-files --type=service
```

![[Pasted image 20260826150043.png]]

What do I mean by subcommand? Well these are things you add after `systemctl`, like `systemctl status` or `systemctl reboot`. 

Usually you shouldn't need the `.service` part when you mention _unit_, but you can add it to be safe.

#### More about `systemctl status`

![[Pasted image 20260826150613.png]]

The enabled and disabled states apply only to unit files that live in one of systemd’s system directories (that is, they are not linked in by a symbolic link) and that have an `[Install]` section in their unit files. 

**Enabled units: **

**Enabled** units should perhaps really be thought of as “installed,” meaning that the directives in the `[Install]` section have been executed and that the unit is wired up to its normal activation triggers. In most cases, this state causes the unit to be activated automatically once the system is bootstrapped.

**Disabled units:**

The **disabled** state is something of a misnomer because the only thing
that’s actually disabled is the **normal activation path**. You can manually activate a
unit that is disabled by running `systemctl start`.

**Static units:**

Many units have no installation procedure, so they can’t truly be said to be enabled or disabled; they’re just available. Such units’ status is listed as `static`. They only become active if activated by hand (`systemctl start`) or named as a dependency of other active units.

**Linked units:**

Unit files that are `linked` were created with `systemctl link`. 

This command creates a symbolic link from one of systemd’s system directories to a unit file that lives elsewhere in the filesystem. 

Such unit files can be addressed by commands or named as dependencies, but they are not full citizens of the ecosystem and have some no-table quirks. 

For example, running `systemctl disable` on a linked unit file deletes the link and all references to it.

**Masked units:**

The `masked` status means **administratively blocked**. `systemd` knows about the unit, but has been forbidden from activating it or acting on any of its configuration directives by `systemctl mask`. 

**As a rule of thumb, turn off units whose status is enabled or linked with `systemctl disable`and reserve `systemctl mask` for static units.


### Targets

A target is just a **name/label** that groups other units together.

Its entire purpose is to answer the question: _"When someone says 'bring the system to this named state,' which units need to be active?"_

**Why would you need that?**

Imagine booting your computer. There isn't one single thing that "boots the system" — booting means a whole pile of separate units all becoming active together:

- the network coming up
- SSH starting
- your filesystems getting mounted
- (if you have a desktop) the display manager showing a login screen

You need some way to say "all of this belongs together, this is what 'normal running system' means." That collective state is what a target represents.

**How does a unit "attach" itself to a target?**

Every unit file that wants to be started as part of a certain state includes a line like this in its `[Install]` section:

```ini
[Install]
WantedBy=multi-user.target
```

This means: _"If the system is trying to reach multi-user.target, please start me too."_

So `multi-user.target` doesn't contain a list of services inside its own file. Instead, other unit files point _at_ it, saying "I belong to you." systemd builds the picture by scanning all units for these `WantedBy=` lines.

Visually:

```
        sshd.service  ----WantedBy---->  multi-user.target
   NetworkManager.service ----WantedBy---->  multi-user.target
        cron.service  ----WantedBy---->  multi-user.target
```

Important targets:

- **multi-user.target** — a fully working system: networking, services, ability to log in — but text-based, no graphical desktop. This is basically what a server runs.
- **graphical.target** — everything in multi-user.target, _plus_ a graphical login/desktop on top. This is what a normal desktop Linux install boots into.


**Practical commands**

**Check what target you're currently in / booting into by default:**

bash

```bash
systemctl get-default
```

**See everything a target pulls in:**

bash

```bash
systemctl list-dependencies multi-user.target
```

**(Switch the system to a different target _right now_ (stops anything not needed by the new target):)**

bash

```bash
sudo systemctl isolate multi-user.target
```

You can do this for servers, which don't need GUI.


**Change what target boots by default:**

bash

```bash
sudo systemctl set-default multi-user.target
```

**To see all available targets:**

```bash
$ systemctl list-units --type=target
```



![[Pasted image 20260826153321.png]]


### Dependencies among units

Our system needs a lot of separate things running before it's "ready" — networking, filesystem mounts, a graphics stack, a database, whatever. But these things can't just start in random order or all at once. Some genuinely depend on others:

- SSH can't accept connections before the network is up.
- A web app can't work if its database hasn't started.
- Nothing should mount `/home` before the disk it lives on is available.


You can extend a unit’s Wants or Requires cohorts by creating a `<unit-file>.wants` or `<unit-file>.requires` directory in `/etc/systemd/system` and adding symlinks there to
other unit files. OR, use `systemctl`:

```bash
$ sudo systemctl add-wants multi-user.target my.local.service
```
This adds a dependency on `my.local.service` to the standard `multiuser target`, ensuring
that the service will be started whenever the system enters multiuser mode.

#### Example

Say you have `my-web-app.service`, and it genuinely can't function without `postgresql.service`:


```ini
[Unit]
Description=My Web App
Requires=postgresql.service
After=postgresql.service
```

Two things happening here:

- `Requires=postgresql.service` — "postgres must be active for me to run; if postgres fails, stop me too"
- `After=postgresql.service` — this is a _separate_ concept: pure ordering. `Requires` says "must be active," but doesn't say _in what order_ they start. `After` says "start me only once postgres has started."

You almost always pair `Requires=` with `After=`, because wanting something active isn't the same as controlling the sequence.


![[Pasted image 20260826154625.png]]


### Execution order

If unit A `Requires` unit B, then unit B will be started or configured before unit A ---> FALSE

**The order in which units are activated (or deactivated) is an entirely separate question from that of which units to activate.**

When the system transitions to a new state, 

systemd first traces the various sources of dependency information to identify the units that will be affected. It then uses `Before` and `After` clauses from the unit files to sort the work list appropriately.


### Example of a unit file

**nginx.service:**

```
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
KillMode=process
KillSignal=SIGQUIT
TimeoutStopSec=5
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

- This service is of type forking, which means that the startup command is expected to terminate even though the actual daemon continues running in the background.
- Since systemd won’t have directly started the daemon, the daemon records its PID (process ID) in the stated `PIDFile` so that systemd can determine which process is the daemon’s primary instance.
- The `Exec` lines are commands to be run in various circumstances. 

	 - `ExecStartPre` commands are run before the actual service is started.
	 - `ExecStart` is the command that actually starts the service.
	 - `ExecReload` tells systemd how to make the service reread its configuration file. (systemd automatically sets the environment variable MAINPID to the appropriate value.)
- Termination for this service is handled through `KillMode` and `KillSignal`, which tell systemd that the service daemon interprets a QUIT signal as an instruction to clean up and exit.
-  If the daemon doesn’t terminate within `TimeoutStopSec` seconds, systemd will force the issue by pelting it with a `TERM` signal and then an uncatchable `KILL` signal.
- The `PrivateTmp` setting is an attempt at increasing security. It puts the service’s `/tmp` directory somewhere other than the actual `/tmp`, which is shared by all the system’s processes and users.



As a general rule, you should never edit a unit file you didn’t write. 

Instead, create a configuration directory in `/etc/systemd/system/unit-file.d` and add one or more configuration files there called `xxx.conf.` The `xxx` part doesn’t matter; just make
sure the file has a `.conf` suffix and is in the right location. `override.conf` is the
standard name.

`.conf` files have the same format as unit files, and in fact systemd smooshes them
all together with the original unit file. However, **override files have priority over**
**the original unit file should both sources try to set the value of a particular option.**

![[Pasted image 20260826160711.png]]

