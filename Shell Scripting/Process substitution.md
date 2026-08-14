Process substitution comes in two flavors: **Input `<(...)`** and **Output `>(...)`**.

### 1. Input Process Substitution: `<(command)`

_Treats the stdout of `command` as if it were a file on disk._

#### Basic: Comparing Command Outputs

Tools like `diff` require **file paths** as arguments. Without process substitution, you'd have to create temporary files on disk:

Bash

```
# Compare the contents of two directories directly without saving to disk
diff -u <(ls /usr/bin) <(ls /usr/local/bin)

# Compare a local file with a remote file over SSH
diff config.txt <(ssh server "cat /etc/config.txt")
```

#### Intermediate: Side-by-Side File Processing

Merge two live command streams side-by-side using `paste` or `comm`:

Bash

```
# Monitor two log streams side by side
paste <(tail -f /var/log/nginx/access.log) <(tail -f /var/log/nginx/error.log)
```

#### Advanced: Multi-Stream Data Pipeline

Pass multiple dynamic streams into a command like `ffmpeg` or `tar`:

Bash

```
# Create a tarball containing live system metrics without generating intermediate log files
tar -czf system_snapshot.tar.gz \
  <(hostname) \
  <(uptime) \
  <(df -h)
```

### 2. Output Process Substitution: `>(command)`

_Sends data written to a "file" into the stdin of `command`._

#### Basic: Global Script Logging

Redirect all stdout and stderr from your entire script through a logging utility:

Bash

```
#!/bin/bash
# Send all script output both to terminal AND syslog
exec > >(logger -t "my_script_tag") 2>&1

echo "This automatically goes to syslog!"
```

#### Intermediate: Splitting a Data Stream

Send a massive output stream to multiple background tasks simultaneously using `tee` **without saving intermediate files to disk**:

Bash

```
# Download a file once, but simultaneously compress it AND compute its SHA-256 hash
curl -s "https://example.com/large_file.iso" | tee >(sha256sum > iso.sha256) | gzip > large_file.iso.gz
```

#### Advanced: Multi-Branch Log Filtering

Filter standard error and standard output into completely different log processing tools in real-time:

Bash

```
# Send regular logs to stdout.log and errors to an alerting webhook
./my_app 1> >(tee -a stdout.log) 2> >(curl -X POST -d @- https://discord-webhook.url)
```