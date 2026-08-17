### Basic syntax


```bash
date [OPTIONS] [+FORMAT]
```

With no arguments, it prints the current date/time in the system's default format:

bash

```bash
date
```

```
Sun Aug 17 10:42:03 IST 2026
```
format: day month date time standard year
### Key options

| Option                | Purpose                                                                    |
| --------------------- | -------------------------------------------------------------------------- |
| `+FORMAT`             | Custom output format using `%` sequences                                   |
| `-d STRING`           | Display a date described by `STRING`, not "now" (doesn't change the clock) |
| `-s STRING`           | **Set** the system date/time to `STRING` (needs root)                      |
| `-u`                  | Use UTC instead of local timezone                                          |
| `-r FILE`             | Show the last modification time of `FILE` (like `stat -c %y`)              |
| `-I[TIMESPEC]`        | ISO 8601 format; `TIMESPEC` = `date`, `hours`, `minutes`, `seconds`        |
| `-R`                  | RFC 5322 format (used in email headers)                                    |
| `--rfc-3339=TIMESPEC` | RFC 3339 format (`date`, `seconds`, `ns`)                                  |

### Format sequences (for `+FORMAT`)

dmY- date month year
HMS- hour minute seconds
AB- day-of-week-name month-of-year
u- day-of-week-number
TF- time format, date format

| Sequence | Meaning                    | Example    |
| -------- | -------------------------- | ---------- |
| `%Y`     | 4-digit year               | 2026       |
| `%y`     | 2-digit year               | 26         |
| `%m`     | month (01-12)              | 08         |
| `%d`     | day of month               | 17         |
| `%H`     | hour (24h)                 | 10         |
| `%I`     | hour (12h)                 | 10         |
| `%M`     | minute                     | 42         |
| `%S`     | second                     | 03         |
| `%N`     | nanoseconds                | 123456789  |
| `%s`     | Unix epoch seconds         | 1755407523 |
| `%A`     | full weekday name          | Monday     |
| `%a`     | abbreviated weekday        | Mon        |
| `%B`     | full month name            | August     |
| `%b`     | abbreviated month          | Aug        |
| `%j`     | day of year (001-366)      | 229        |
| `%Z`     | timezone name              | IST        |
| `%z`     | timezone offset            | +0530      |
| `%p`     | AM/PM                      | AM         |
| `%u`     | day of week (1=Mon..7=Sun) | 1          |
| `%T`     | equivalent to `%H:%M:%S`   | 10:42:03   |
| `%F`     | equivalent to `%Y-%m-%d`   | 2026-08-17 |

### Use cases

#### 1. Get a timestamped filename (extremely common pattern)

bash

```bash
touch "backup_$(date +%Y%m%d_%H%M%S).tar.gz"
```

```
backup_20260817_104203.tar.gz
```

Sortable, no spaces, safe for filenames — the standard pattern for log rotation or backup scripts.

#### 2. Get Unix epoch (for comparisons/scripting math)

bash

```bash
date +%s
```

Useful for measuring script runtime:

bash

```bash
start=$(date +%s)
# ... do work ...
end=$(date +%s)
echo "Elapsed: $((end - start)) seconds"
```

#### 3. Convert an epoch timestamp back to human-readable

bash

```bash
date -d @1755407523
```

```
Mon Aug 17 10:42:03 IST 2026
```

Handy when you pull a Unix timestamp out of a JSON API response (e.g. Codeforces contest API) and want to eyeball it.

#### 4. Compute a date in the past/future (relative dates)

bash

```bash
date -d "7 days ago"
date -d "next monday"
date -d "2 hours"
date -d "+1 month"
date -d "yesterday"
```

Great for log cleanup scripts:

bash

```bash
find . -type f -mtime +$(date -d "30 days ago" +%j) -delete   # not quite right, see note below
```

More correctly, just use `find`'s own relative flags for file age, but `date -d` is perfect for generating cutoff labels:

bash

```bash
cutoff=$(date -d "7 days ago" +%Y%m%d)
echo "Deleting logs older than $cutoff"
```

#### 5. Convert between timezones

bash

```bash
date -u                              # current UTC time
TZ="America/New_York" date           # current time in NYC
date -d "10:00 IST" -u                # convert IST time to UTC
```

Useful when scheduling something against a Codeforces/AtCoder contest start time listed in a different timezone.

#### 6. ISO 8601 output (for logs, APIs, configs)

bash

```bash
date -Iseconds
```

```
2026-08-17T10:42:03+05:30
```

This is the standard machine-readable format most APIs and structured logs expect — much better than parsing the default locale-dependent `date` output.

#### 7. Get file modification time without `stat`

bash

```bash
date -r file.txt
date -r file.txt +%s      # as epoch, directly comparable
```

Simpler than `stat -c %y` when you only need the mtime and want to reformat it inline.

#### 8. Set the system clock manually (needs root)

bash

```bash
sudo date -s "2026-08-17 10:42:00"
```

Useful after a fresh Arch install/chroot where the hardware clock hasn't synced yet, before NTP kicks in.

#### 9. Generate a sequence of dates (combine with a loop)

bash

```bash
for i in $(seq 0 6); do
  date -d "+$i days" +%Y-%m-%d
done
```

Good for generating a week's worth of labels for a script that processes daily logs, or building a contest-tracking calendar.

#### 10. RFC formats for emails/HTTP headers

bash

```bash
date -R                                    # RFC 5322 (email Date: header)
date --rfc-3339=seconds                    # RFC 3339
```

```
Mon, 17 Aug 2026 10:42:03 +0530
2026-08-17 10:42:03+05:30
```

Useful if you're scripting something that crafts raw email headers or HTTP `Date`/`If-Modified-Since` headers manually.

#### 11. Day-of-week / day-of-year logic in scripts

bash

```bash
if [ "$(date +%u)" -eq 6 ] || [ "$(date +%u)" -eq 7 ]; then
  echo "It's the weekend"
fi
```

Common in cron-adjacent scripts that should behave differently on weekends (e.g. skip a build, send a different status message).

#### 12. Measure elapsed time with nanosecond precision

bash

```bash
start=$(date +%s.%N)
sleep 0.3
end=$(date +%s.%N)
echo "Elapsed: $(echo "$end - $start" | bc) sec"
```

Useful for benchmarking scripts where `%s` alone (whole seconds) isn't precise enough — e.g. timing a competitive programming solution's wall-clock run.

#### 13. Parse a specific custom date string

bash

```bash
date -d "17/08/2026" +%A     # may fail depending on locale/ambiguity
date -d "2026-08-17" +%A     # safest: use ISO format as input
```

```
Monday
```

Note: `date -d` is best fed ISO 8601 (`YYYY-MM-DD`) input — ambiguous formats like `DD/MM/YYYY` vs `MM/DD/YYYY` depend on locale and can silently misparse.

### `date` vs `stat`, quick distinction

- `date` deals with **the current time, arbitrary date arithmetic, and formatting** — it doesn't know anything about files unless you use `-r`.
- `stat` deals with **a specific file's metadata** — timestamps are just one part of a bigger picture (permissions, inode, etc.).
- For "how old is this file" logic, `stat -c %Y file` + `date +%s` combined is the standard pattern:

bash

```bash
age=$(( $(date +%s) - $(stat -c %Y file.txt) ))
echo "File is $age seconds old"
```