# Basic Linux Processes

## What Is a Process?

A **process** is any program that is currently running on your system. This includes:
- Applications (web browsers, text editors, terminals)
- System services (network managers, print services)
- Background tasks

A **daemon** is a special type of process that runs in the background to provide services (like a web server or SSH service). Daemon names often end with "d" (like `sshd`, `httpd`, `systemd`).

### Process Basics
- Every process has a unique **PID** (Process ID)
- Processes can have **parent** and **child** processes
- Process ID 1 is always the **init system** (usually `systemd`)
- All other processes are descendants of PID 1

## Viewing Processes

### ps - List Processes

```bash
# Show your current processes
ps

# Show all processes with detailed info
ps aux

# Show processes in a tree format (see parent-child relationships)
ps auxf

# Show specific process by PID
ps -p 1

# Show all processes for a specific user
ps -u username

# Find processes by name
ps aux | grep firefox
```

**Understanding ps aux output:**
```
USER       PID  %CPU %MEM    VSZ   RSS TTY   STAT START TIME COMMAND
root         1   0.0  0.2 169564 11680 ?     Ss   10:30 0:01 /sbin/init
```
- **USER**: Who owns the process
- **PID**: Process ID
- **%CPU**: CPU usage percentage
- **%MEM**: Memory usage percentage
- **STAT**: Process state (S=sleeping, R=running, Z=zombie)
- **COMMAND**: The command that started the process

### Finding Process with PID 1
```bash
# View the init process (PID 1)
ps -p 1

# Get detailed info about PID 1
ps -fp 1

# See what PID 1 is running
ls -l /proc/1/exe
```

### Finding Child Processes
```bash
# Show processes in tree format to see parent-child relationships
ps auxf

# Show only children of PID 1
ps --ppid 1

# Alternative: use pstree to see process tree
pstree -p

# Show tree starting from PID 1
pstree -p 1
```

### top - Real-Time Process Monitor

```bash
# Launch top (press 'q' to quit)
top

# Sort by CPU usage (press 'P' in top)
# Sort by memory usage (press 'M' in top)
# Kill a process (press 'k' then enter PID)
```

**Top commands while running:**
- `P` - sort by CPU usage
- `M` - sort by memory usage
- `k` - kill a process (enter PID)
- `q` - quit top
- `h` - help

### htop - Better Process Monitor

```bash
# Launch htop (more user-friendly than top)
htop

# May need to install first
sudo apt install htop
```

**htop is easier to use:**
- Color-coded display
- Mouse support
- Easy to navigate with arrow keys
- F9 to kill processes
- F10 to quit

## Managing Processes

### kill - Terminate Processes

```bash
# Gracefully stop a process (SIGTERM)
kill PID

# Force kill a process (SIGKILL)
kill -9 PID

# Kill by process name
killall firefox

# Kill by pattern
pkill firefox
```

**Common kill signals:**
- `kill PID` or `kill -15 PID` - polite shutdown (SIGTERM)
- `kill -9 PID` - force kill (SIGKILL)
- `kill -1 PID` - restart/reload (SIGHUP)

### Example: Stopping a Process
```bash
# Find the process
ps aux | grep myapp

# Note the PID (e.g., 12345)
# Try graceful shutdown first
kill 12345

# Wait a few seconds, check if it's still running
ps aux | grep myapp

# If still running, force kill
kill -9 12345
```

### Background and Foreground

```bash
# Run a command in the background
command &

# Put current process in background
Ctrl+Z (to suspend)
bg (to continue in background)

# Bring background process to foreground
fg

# List background jobs
jobs
```

## Process Priority

### nice - Start Process with Priority

Priority ranges from -20 (highest) to 19 (lowest). Default is 0.

```bash
# Start with lower priority (nice value 10)
nice -n 10 my-command

# Start with higher priority (requires sudo)
sudo nice -n -10 my-command
```

### renice - Change Priority of Running Process

```bash
# Lower priority of existing process
renice -n 10 -p PID

# Increase priority (requires sudo)
sudo renice -n -5 -p PID

# Change priority for all processes of a user
sudo renice -n 5 -u username
```

## Quick Reference Commands

```bash
# View all processes
ps aux

# View process tree
ps auxf

# Find PID 1 (init process)
ps -p 1

# Find children of PID 1
ps --ppid 1

# Real-time monitoring
top

# Better real-time monitoring
htop

# Find process by name
ps aux | grep process-name

# Kill process gracefully
kill PID

# Force kill process
kill -9 PID
```

## Common Tasks for Beginners

### Task 1: View Process with PID 1
```bash
# Show PID 1 (the init system)
ps -p 1

# Get more details
ps -fp 1

# Expected output will show systemd or init
```

### Task 2: Find Child Processes of PID 1
```bash
# Method 1: Show direct children
ps --ppid 1

# Method 2: See full process tree
pstree -p 1

# Method 3: Use ps with formatting
ps -ef | grep " 1 " | grep -v "grep"
```

### Task 3: Monitor System Resources
```bash
# Check CPU and memory usage
top

# Or use htop if available
htop

# View processes using most CPU
ps aux --sort=-%cpu | head -10

# View processes using most memory
ps aux --sort=-%mem | head -10
```

### Task 4: Kill a Misbehaving Process
```bash
# Step 1: Find the process
ps aux | grep process-name

# Step 2: Note the PID (second column)
# Step 3: Kill it gracefully
kill PID

# Step 4: If it doesn't stop, force kill
kill -9 PID
```

## Process States

| State | Symbol | Meaning |
|-------|--------|---------|
| Running | R | Currently executing |
| Sleeping | S | Waiting for an event |
| Stopped | T | Stopped by signal |
| Zombie | Z | Finished but not cleaned up |
| Dead | X | Process is terminated |

## Tips for Beginners

1. **Always try `kill` before `kill -9`** - give processes a chance to clean up
2. **Use `htop` if available** - it's much easier to read than `top`
3. **PID 1 is special** - never kill it! It's the system's init process
4. **Use `ps auxf` to see relationships** - the tree view helps understand what spawned what
5. **Check `systemctl status`** - it shows if services are running properly
6. **Use `grep` to filter** - `ps aux | grep firefox` finds Firefox processes quickly

## Practice Examples

```bash
# 1. View all your running processes
ps aux | grep $USER

# 2. Find the init process (PID 1)
ps -p 1 -o pid,comm,cmd

# 3. See what services PID 1 manages
ps --ppid 1 -o pid,comm,cmd

# 4. Monitor system in real-time
top
# (press 'q' to quit)

# 5. Find and kill a test process
sleep 300 &  # Start a background process
ps aux | grep sleep  # Find its PID
kill PID  # Replace PID with the number you found

# 6. Check if SSH service is running
systemctl status ssh

# 7. List all running services
systemctl list-units --type=service --state=running
| List processes | `ps aux` |
| Process tree | `ps auxf` or `pstree` |
| Real-time monitor | `top` or `htop` |
| Find PID 1 | `ps -p 1` |
| Find children of PID 1 | `ps --ppid 1` |
| Kill process | `kill PID` |
| Force kill | `kill -9 PID` |
| Service status | `systemctl status service-name` |
| Start service | `sudo systemctl start service-name` |
| Stop service | `sudo systemctl stop service-name` |
