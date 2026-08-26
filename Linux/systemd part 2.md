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



