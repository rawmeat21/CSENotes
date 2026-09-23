The cron daemon is the traditional tool for running commands on a predetermined schedule.
It starts when the system boots and runs as long as the system is up.


-> cron reads configuration files containing lists of command lines and times at which
they are to be invoked. 

-> The command lines are executed by `sh`, so almost anything you can do by hand from the shell can also be done with cron. If you prefer, you can even configure cron to use a different shell.

-> A cron configuration file is called a “crontab,” short for “cron table.” Crontabs
for individual users are stored under /var/spool/cron (Linux) or /var/cron/tabs (FreeBSD). **There is at most 1 crontab file per user.** Crontab files are plain text files named with the login names of the users to whom they belong. `cron` uses these filenames (and the file ownership) to figure out which UID to use when running the commands contained in each file.

-> The `crontab` command transfers crontab files to and from this directory.

-> You shouldn’t edit crontab files directly, because this approach might result in `cron` not noticing your changes. Probably, you should use `crontab`.

-> cron normally does its work silently, but most versions can keep a log file (usually `/var/log/cron`) that lists the commands that were executed and the times at which they ran.

**Format of a crontab file:**

```
minute hour dom month weekday command <- entry for a cron job 'command'
```

![Pasted image 20260911235941](../assets/Pasted%20image%2020260911235941.png)


The `command` is the `sh` command line to be executed. It can be **any valid shell command**and should not be quoted.

Percent signs (%) indicate newlines within the command field. Only the text up to
the first percent sign is included in the actual command. The remaining lines are
given to the command as standard input. Use a backslash (\\) as an escape character
in commands that have a meaningful percent sign, for example, `date +\%s`.

Although `sh` is involved in executing the command, **the shell does not act as a login shell** and does not read the contents of `~/.profile` or `~/.bash_profile`. As a result,
the command’s environment variables might be set up somewhat differently from
what you expect.

Use the fully qualified name, then it will work even if `PATH` is not set.

```
* * * * * echo $(/bin/date) - $(/usr/bin/uptime) >> ~/uptime.log
```
Alternatively, you can set environment variables explicitly at the top of the crontab:

```
PATH=/bin:/usr/bin
* * * * * echo $(date) - $(uptime) >> ~/uptime.log
```

Note!

cron jobs run via `sh`, with none of your interactive shell's environment, no `~/.bashrc`, no `PATH` additions from your dotfiles, none of your aliases or functions. This is _the_ number one cause of "it works when I run it manually but silently fails in cron." Always use full paths or set `PATH` explicitly.


### systemd timers

A timer is two files working together.

**The service:**

`/etc/systemd/system/home-backup.service`:

```ini
[Unit]
Description=Nightly home directory backup

[Service]
Type=oneshot
ExecStart=/usr/bin/rsync -a --delete /home/rawmeat/ /mnt/backup/home/
```

**The timer:**

`/etc/systemd/system/home-backup.timer`:

```ini
[Unit]
Description=Run home-backup nightly

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true
RandomizedDelaySec=300

[Install]
WantedBy=timers.target
```

The `*-*-*` before the time is year-month-day, all wildcarded; so this fires at 2:00:00 AM every single day.

**`Persistent=true`**: this is the systemd feature that has **no cron equivalent at all**, and it's genuinely important: if your laptop is asleep/off at 2:00 AM (very plausible for a personal Arch laptop, unlike an always-on server), the timer would normally just... not fire, and you'd silently skip a day's backup. `Persistent=true` tells systemd "remember the last time this fired (it stores this in `/var/lib/systemd/timers/`), and if we missed a scheduled run because the system was off, run it once as soon as we boot back up." 

**`RandomizedDelaySec=300`** — adds a random delay up to 5 minutes before actually firing, to avoid thundering-herd effects if this same unit ran on many machines simultaneously

**`[Install] WantedBy=timers.target`** — parallel to `multi-user.target` for regular services, but the specific target that "enabling a timer" hooks into.


```bash
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now home-backup.timer
$ systemctl list-timers                    # see it, and when it'll next fire
$ systemctl status home-backup.timer
```


You can also trigger the service directly:

```bash
$ sudo systemctl start home-backup.service   # runs it RIGHT NOW, timer untouched
$ journalctl -u home-backup.service -n 50    # see exactly what happened
```

This cannot really be done with  cron I think, you'd have to wait or set `* * * * *`.







