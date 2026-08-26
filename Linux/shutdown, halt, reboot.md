The `halt` command performs the essential duties required for shutting down the system. halt logs the shutdown, kills nonessential processes, flushes cached filesystem
blocks to disk, and then halts the kernel. On most systems, `halt -p` powers down
the system as a final flourish.


`reboot` is essentially identical to `halt`, but it causes the machine to reboot instead
of halting.

The `shutdown` command is a layer over `halt` and `reboot` that provides for scheduled shutdowns and warnings to logged in users. 

It dates back to the days of time-sharing systems and is now largely obsolete. shutdown does nothing of technical value beyond halt or reboot, so feel free to ignore it if you don’t have multiuser systems.