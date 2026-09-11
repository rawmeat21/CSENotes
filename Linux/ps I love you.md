You can obtain a useful overview of all the processes running on the system with `ps aux`.

```
❯ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.1  20392 12664 ?        Ss   Sep10   0:01 /sbin/init
root           2  0.0  0.0      0     0 ?        S    Sep10   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        S    Sep10   0:00 [pool_workqueue_release]
root           4  0.0  0.0      0     0 ?        I<   Sep10   0:00 [kworker/R-rcu_gp]
root           5  0.0  0.0      0     0 ?        I<   Sep10   0:00 [kworker/R-sync_wq]
root           6  0.0  0.0      0     0 ?        I<   Sep10   0:00 [kworker/R-kvfree_rcu_reclaim]
root           7  0.0  0.0      0     0 ?        I<   Sep10   0:00 [kworker/R-slub_flushwq]
root           8  0.0  0.0      0     0 ?        I<   Sep10   0:00 [kworker/R-netns]
root          11  0.0  0.0      0     0 ?        I<   Sep10   0:00 [kworker/0:0H-kblockd]
root          14  0.0  0.0      0     0 ?        I<   Sep10   0:00 [kworker/R-mm_percpu_wq]
root          15  0.0  0.0      0     0 ?        S    Sep10   0:00 [ksoftirqd/0]
root          16  0.0  0.0      0     0 ?        I    Sep10   0:08 [rcu_preempt]
root          17  0.0  0.0      0     0 ?        S    Sep10   0:00 [rcub/0]
.
.
.
rawmeat     1123  0.0  0.0   8020  5760 tty1     S+   Sep10   0:00 /usr/sbin/sh /usr/lib/uwsm/signal-handler.sh wayland-session-envelope@hyprland.desktop.target
rawmeat     1145  0.0  0.0   5972  4004 ?        Ss   Sep10   0:00 waitpid -e 1123
rawmeat     1146  0.0  0.0  12640  7256 tty1     S+   Sep10   0:00 systemctl --user start --wait wayland-session-envelope@hyprland.desktop.target
rawmeat     1194  0.0  0.0  16876  3072 ?        Ssl  Sep10   0:00 /usr/bin/start-hyprland
rawmeat     1199  0.3  1.5 1621296 114984 ?      Sl   Sep10   4:56 Hyprland --watchdog-fd 4
rawmeat     1245  0.0  0.0 165876  6472 ?        Ssl  Sep10   0:00 /usr/lib/dconf-service
rawmeat     1252  0.0  0.0  85132  4692 ?        Sl   Sep10   0:03 hypridle
rawmeat     1254  0.0  0.6 1081404 47616 ?       Sl   Sep10   1:12 waybar
rawmeat     1257  0.0  0.3 1450452 24296 ?       Sl   Sep10   0:00 hyprpaper
rawmeat     1277  0.0  0.1 316108  9204 ?        Ssl  Sep10   0:00 /usr/lib/geoclue-2.0/demos/agent
rawmeat     1296  0.0  0.2 1246812 18388 ?       Sl   Sep10   0:00 Xwayland :0 -rootless -core -listenfd 65 -listenfd 66 -displayfd 121 -wm 118
polkitd     1297  0.0  0.1 383396 10536 ?        Ssl  Sep10   0:00 /usr/lib/polkit-1/polkitd --no-debug --log-level=notice
rawmeat     1298  0.0  0.0 380344  6692 ?        Ssl  Sep10   0:00 /usr/lib/at-spi-bus-launcher
rawmeat     1304  0.0  0.0   7584  3540 ?        S    Sep10   0:00 /usr/bin/dbus-broker-launch --config-file=/usr/share/defaults/at-spi2/accessibility.conf --scope user
rawmeat     1309  0.0  0.0   4844  2768 ?        S    Sep10   0:00 dbus-broker --log 10 --controller 9 --machine-id cf2896bf43d348d383b03cb88a1d1fb7 --max-bytes 100000000000000 --max-fds 6400
rawmeat     1316  0.0  0.1 331220 11032 ?        Ssl  Sep10   0:00 /usr/lib/gvfsd
rawmeat     1346  0.0  0.1 324972  7724 ?        Sl   Sep10   0:00 /usr/lib/gvfsd-fuse /run/user/1000/gvfs -f
```
You can also use `ps lax` (long output).

![[Pasted image 20260911231915.png]]

`ps lax` includes fields such as the parent process ID (PPID), niceness (NI), and the type of resource on which the process is waiting (WCHAN, short for “wait channel”).

![[Pasted image 20260911232043.png]]

Get PID of process:

```
$ ps aux | grep sshd
root 6811 0.0 0.0 78056 1340 ? Ss 16:04 0:00 /usr/sbin/sshd
bwhaley 13961 0.0 0.0 110408 868 pts/1 S+ 20:37 0:00 grep /usr/sbin/sshd
```

OR:

```
$ pidof sshd
6811
```


