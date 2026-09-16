# Beginner's Guide to Viewing System Logs (Fedora and AWS Linux)

## Why Logs Matter

Logs are the record of what your system and its services have been doing - service start/stop events, errors, login attempts, kernel messages, and more. When something breaks, logs are the first place you look.

Linux has two main ways logs get stored and viewed:
1. **Syslog / `/var/log` files** - the traditional approach, plain text log files written by `rsyslog`
2. **systemd-journald** - the modern systemd-based logging service, viewed with `journalctl`

Both Fedora and Amazon Linux run on **systemd**, so both have `journald` running and both can view logs with `journalctl`. The difference you'll notice in practice is emphasis: Fedora desktop/server installs typically also have `rsyslog` set up to write familiar plain-text files under `/var/log`, while a minimal cloud image like Amazon Linux leans on `journald` alone unless you add `rsyslog` yourself.

## Method 1: Syslog and `/var/log`

**Syslog** is an old, well-established logging standard. A syslog daemon (`rsyslog` on Fedora) listens for log messages from the kernel and other programs, and writes them out to plain text files in `/var/log/`, sorted by category.

### Common Log Files

```bash
# List what's in /var/log
ls /var/log
```

| File | Contains |
|---|---|
| `/var/log/messages` | General system messages (the main log on RHEL-family systems) |
| `/var/log/secure` | Authentication and security events (logins, `sudo` use) |
| `/var/log/boot.log` | Messages from the boot process |
| `/var/log/cron` | Scheduled task (`cron`) activity |
| `/var/log/httpd/` | Web server logs (if `httpd` is installed) |

### Viewing Log Files

Since these are just text files, ordinary text tools work:

```bash
# View the whole file, oldest first
cat /var/log/messages

# View the last 20 lines
tail -n 20 /var/log/messages

# Follow the file live as new lines are appended
tail -f /var/log/messages

# Search for a keyword across all log lines
grep "sshd" /var/log/messages

# Look at authentication attempts
sudo tail -n 20 /var/log/secure
```

### Why This Method Exists

Plain text files are simple, portable, and easy to search with basic tools you already know (`cat`, `grep`, `tail`). The downside is there's no built-in structure - just lines of text - and log rotation (deleting/compressing old logs) has to be managed separately by a tool called `logrotate`.

## Method 2: systemd-journald

**`systemd-journald`** is the logging service built into systemd. Instead of plain text files, it stores log entries in a structured binary format, and you view them with the `journalctl` command. Since every systemd service is started and supervised by systemd, journald can automatically capture all of their output without any extra configuration.

### Viewing Journal Logs

```bash
# View all logs (oldest first), piped through a pager
journalctl

# View the most recent logs first
journalctl -r

# Follow logs live (like tail -f)
journalctl -f

# View logs for one specific service
journalctl -u sshd

# View logs since the last boot
journalctl -b

# View logs from today only
journalctl --since today

# View logs from a specific time range
journalctl --since "2026-09-01 08:00:00" --until "2026-09-01 09:00:00"

# Show only errors and more severe messages
journalctl -p err
```

### Why This Method Exists

Because every entry is structured (with fields like the service name, timestamp, and priority), `journalctl` can filter precisely - by service, by time, by severity - without you having to know which text file to grep. It also automatically manages storage size and rotation for you, and survives reboots by default when persistent storage is enabled (`/var/log/journal/`).

## Quick Comparison

| | Syslog (`rsyslog` + `/var/log`) | systemd-journald (`journalctl`) |
|---|---|---|
| Storage format | Plain text files | Structured binary format |
| Viewed with | `cat`, `tail`, `grep`, `less` | `journalctl` |
| Filtering | Manual, with `grep`/`awk` on text | Built-in filters (`-u`, `--since`, `-p`, etc.) |
| Captures service output automatically | Only if the service is configured to log via syslog | Yes, automatically for any systemd-managed service |
| Log rotation | Needs `logrotate` configured separately | Handled automatically by journald |
| Typical presence | Common on Fedora (via `rsyslog`) | Present everywhere systemd runs, including Amazon Linux |

## A Note on Fedora vs. Amazon Linux

Both distributions actually run `journald`, since both use systemd (see the systemd guide). The practical difference you'll run into:

- **Fedora** installs commonly include `rsyslog`, so you'll find familiar files like `/var/log/messages` and `/var/log/secure` in addition to the journal.
- **Amazon Linux** EC2 images are minimal by default and often don't have `rsyslog` installed, so `/var/log` will have far fewer general-purpose log files, and `journalctl` is the main way to see what systemd-managed services are doing. You can always `dnf install rsyslog` on Amazon Linux if you want traditional `/var/log` files too.

Either way, `journalctl` will work on both systems - it's the more universal starting point when you're not sure which log files exist.

## Quick Reference

```bash
# Syslog / /var/log
ls /var/log                        # list log files
tail -f /var/log/messages          # follow the main log live
sudo tail -f /var/log/secure       # follow authentication log live
grep "error" /var/log/messages     # search for a keyword

# systemd-journald
journalctl                         # view all logs
journalctl -r                      # newest logs first
journalctl -f                      # follow logs live
journalctl -u service-name         # logs for one service
journalctl -b                      # logs since last boot
journalctl --since today           # logs from today
journalctl -p err                  # only errors and worse
```
