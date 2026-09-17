![[Pasted image 20260916113121.png]]
![[Pasted image 20260916113131.png]]
![[Pasted image 20260916113211.png]]

![[Pasted image 20260916113308.png]]

![[Pasted image 20260916113457.png]]

![[Pasted image 20260916113522.png]]

Execute a task at 2:30pm **everyday**:

```
30 14 * * * /tmp/basic.sh
```
Execute a task at 2:30pm each month, at date 15:

```
30 14 15 * *
```
Execute a task at 2:30pm on February 15

```
30 14 15 2 *
```
Execute a task at 2:30pm every sunday
```
30 14 * * 0
```

Example script:

```bash
/usr/bin/touch $HOME/myfile.txt

echo "Cronning" >> $HOME/myfile.txt
```

![[Pasted image 20260916114537.png]]

![[Pasted image 20260916114609.png]]

![[Pasted image 20260916114722.png]]

![[Pasted image 20260916114754.png]]

![[Pasted image 20260916114828.png]]


### Anacron - What is this?

![[Pasted image 20260916115042.png]]

Suppose you have a backup script that runs every week and takes a backup (wow).

![[Pasted image 20260916115110.png]]

What if server is down? - Then you miss the schedule, no backup taken.

![[Pasted image 20260916115200.png]]

So how does it work? Suppose the script runs at day 1, then its scheduled to run on day 8, but server is down, so the task doesn't run. Say the server becomes actie again at day 9. Then, anacron would see that a task was scheduled but not completed, so it would run this task then. 

![[Pasted image 20260916115411.png]]


This is how the file should look like without edits:

```
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# the maximal random delay added to the base delay of the jobs
RANDOM_DELAY=45
# the jobs will be started during the following hours only
START_HOURS_RANGE=3-22

#period in days   delay in minutes   job-identifier   command
1       5       cron.daily              nice run-parts /etc/cron.daily
7       25      cron.weekly             nice run-parts /etc/cron.weekly
@monthly 45     cron.monthly            nice run-parts /etc/cron.monthly
```

Look at the 1st row:

`period in days = 1`: Means interval time
`delay in minutes = 5`: After the system is running, after how much time should the tasks that were supposed to be run get executed?
`job-identifier = cron.daily`: Name of the job
`command = nice run-parts /etc/cron.daily`: The command

```
# the jobs will be started during the following hours only
START_HOURS_RANGE=3-22
```
 This means the anacron jobs can get executed anytime between 3am - 10pm.

```
# the maximal random delay added to the base delay of the jobs
RANDOM_DELAY=45
```
This means max time upto which a job can get delayed.

We can write our own:

```
15 5 backup_job ./tmp/backup.sh
```


Trigger crontabs:

```
$ sudo anacron -f -d
```

To trigger now:

```
$ sudo anacron -fdn
```

![[Pasted image 20260916120758.png]]

How to see logs?

```
$ sudo less /var/log/cron
```


![[Pasted image 20260916120929.png]]


