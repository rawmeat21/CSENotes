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




