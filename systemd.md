# Understanding Systemd and Daemons

## What Is a Daemon?

A **daemon** is a background process that runs continuously to provide services to the system or users. Think of daemons as helpers that work behind the scenes without you seeing them.

**Common characteristics of daemons:**
- Run in the background (not interactive)
- Start automatically when the system boots
- Provide services (web servers, SSH access, printing, etc.)
- Usually have names ending in "d" (daemon)

**Examples of common daemons:**
- `sshd` - SSH server (lets you log in remotely)
- `httpd` - Apache web server
- `nginx` - Another web server
- `crond` - Task scheduler
- `systemd` - System and service manager

## What Is Systemd?

**Systemd** is the init system and service manager for modern Linux distributions. It:
- Is the first process that starts (PID 1)
- Manages all other processes and services
- Controls what starts at boot time
- Handles system logging
- Manages system resources

Think of systemd as the "manager" of all services on your Linux system.

### Why Systemd?
- Faster boot times (starts services in parallel)
- Easy service management with `systemctl`
- Better logging with `journalctl`
- Dependency management (starts services in the right order)

## Managing Services with systemctl

The `systemctl` command is your main tool for managing services (daemons) on a systemd-based system.

### Basic Service Commands

```bash
# Check if a service is running
systemctl status service-name

# Start a service
sudo systemctl start service-name

# Stop a service
sudo systemctl stop service-name

# Restart a service (stop then start)
sudo systemctl restart service-name

# Reload configuration without stopping
sudo systemctl reload service-name
```

### Service Status Examples

```bash
# Check SSH daemon status
systemctl status ssh

# Check Apache web server
systemctl status httpd

# Check if Nginx is running
systemctl status nginx

# Multiple services at once
systemctl status sshd httpd nginx
```

**Understanding status output:**
```
● ssh.service - OpenBSD Secure Shell server
   Loaded: loaded (/lib/systemd/system/ssh.service; enabled)
   Active: active (running) since Mon 2026-09-01 10:30:15
```
- **Loaded**: Service is recognized by systemd
- **enabled**: Will start automatically at boot
- **Active**: Currently running
- **inactive**: Not currently running

### Enable/Disable Services at Boot

```bash
# Enable service to start automatically at boot
sudo systemctl enable service-name

# Disable service from starting at boot
sudo systemctl disable service-name

# Check if service is enabled
systemctl is-enabled service-name

# Enable and start immediately
sudo systemctl enable --now service-name

# Disable and stop immediately
sudo systemctl disable --now service-name
```

**Example: SSH Service**
```bash
# Make sure SSH starts at boot
sudo systemctl enable sshd

# Check if it's enabled
systemctl is-enabled sshd
# Output: enabled

# Start it now
sudo systemctl start sshd

# Verify it's running
systemctl status sshd
```

## Viewing Services

### List Services

```bash
# List all running services
systemctl list-units --type=service --state=running

# List all services (running and stopped)
systemctl list-units --type=service

# List all enabled services
systemctl list-unit-files --type=service --state=enabled

# List all failed services
systemctl list-units --type=service --state=failed
```

### Simplified listing

```bash
# Just show running services (shorter output)
systemctl --type=service --state=running

# Show all services
systemctl --type=service
```

## Viewing Service Logs

Systemd captures all service logs with `journalctl`.

```bash
# View logs for a specific service
journalctl -u service-name

# Follow logs in real-time (like tail -f)
journalctl -u service-name -f

# Show only recent logs
journalctl -u service-name -n 50

# Show logs since last boot
journalctl -u service-name -b

# Show logs from today
journalctl -u service-name --since today

# Show logs from specific time
journalctl -u service-name --since "2026-09-01 10:00:00"
```

**Example: Checking SSH logs**
```bash
# See recent SSH activity
journalctl -u sshd -n 100

# Follow SSH logs live
journalctl -u sshd -f

# See SSH logs from today
journalctl -u sshd --since today
```

## Common Service Management Tasks

### Starting and Stopping Web Servers

```bash
# Apache (httpd)
sudo systemctl start httpd
sudo systemctl stop httpd
sudo systemctl restart httpd
systemctl status httpd

# Nginx
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
systemctl status nginx
```

### Managing SSH

```bash
# Check SSH status
systemctl status sshd

# Restart SSH (after config changes)
sudo systemctl restart sshd

# Stop SSH temporarily
sudo systemctl stop sshd

# Start SSH again
sudo systemctl start sshd

# Ensure SSH starts at boot
sudo systemctl enable sshd
```

### Database Services

```bash
# MariaDB (MySQL replacement on AWS Linux)
sudo systemctl start mariadb
sudo systemctl status mariadb
sudo systemctl enable mariadb

# PostgreSQL
sudo systemctl start postgresql
sudo systemctl status postgresql
```

## Quick Reference Commands

```bash
# Check service status
systemctl status service-name

# Start/Stop/Restart service
sudo systemctl start service-name
sudo systemctl stop service-name
sudo systemctl restart service-name

# Enable/Disable at boot
sudo systemctl enable service-name
sudo systemctl disable service-name

# List running services
systemctl list-units --type=service --state=running

# View service logs
journalctl -u service-name

# Follow logs in real-time
journalctl -u service-name -f

# Check if service is enabled
systemctl is-enabled service-name
```

## Understanding Service States

| State | Meaning |
|-------|---------|
| active (running) | Service is running |
| active (exited) | Successfully completed one-time task |
| inactive (dead) | Service is not running |
| failed | Service failed to start or crashed |
| activating | Currently starting up |
| deactivating | Currently stopping |

### Load States

| State | Meaning |
|-------|---------|
| loaded | Service file is loaded |
| not-found | Service file doesn't exist |
| error | Problem loading service file |
| masked | Service is completely disabled |

## Practical Examples for Beginners

### Example 1: Check What Services Are Running
```bash
# See all running services
systemctl --type=service --state=running

# Count how many services are running
systemctl --type=service --state=running | wc -l
```

### Example 2: Manage Apache Web Server
```bash
# Install Apache (if needed)
sudo dnf install httpd

# Check if it's running
systemctl status httpd

# Start it
sudo systemctl start httpd

# Make it start at boot
sudo systemctl enable httpd

# Test in browser: http://your-server-ip
```

### Example 3: Troubleshoot a Failed Service
```bash
# Find failed services
systemctl --failed

# Check specific service
systemctl status failed-service

# View detailed logs
journalctl -u failed-service -n 100

# Try to restart
sudo systemctl restart failed-service
```

### Example 4: Stop Unnecessary Services
```bash
# Find what's enabled at boot
systemctl list-unit-files --type=service --state=enabled

# Disable a service you don't need
sudo systemctl disable service-name

# Stop it immediately too
sudo systemctl stop service-name
```

## Common Daemons You'll Encounter

| Daemon | Purpose | Check Status |
|--------|---------|--------------|
| sshd | Remote login | `systemctl status sshd` |
| httpd | Apache web server | `systemctl status httpd` |
| nginx | Web server | `systemctl status nginx` |
| mariadb | Database | `systemctl status mariadb` |
| postgresql | Database | `systemctl status postgresql` |
| crond | Task scheduler | `systemctl status crond` |
| NetworkManager | Network connections | `systemctl status NetworkManager` |

## Tips for Beginners

1. **Always check status first** - Use `systemctl status` before making changes
2. **Use tab completion** - Type `systemctl status a` then press TAB to see available services
3. **Read the logs** - When something fails, `journalctl -u service-name` tells you why
4. **Restart vs Reload** - `reload` is gentler and doesn't drop connections
5. **Be careful with enable/disable** - This affects boot time behavior
6. **Don't need sudo for status** - Only reading requires no privileges
7. **Use `-f` to watch logs** - Great for debugging: `journalctl -u service-name -f`

## Common Tasks Summary

| Task | Command |
|------|---------|
| Check status | `systemctl status service-name` |
| Start service | `sudo systemctl start service-name` |
| Stop service | `sudo systemctl stop service-name` |
| Restart service | `sudo systemctl restart service-name` |
| Enable at boot | `sudo systemctl enable service-name` |
| Disable at boot | `sudo systemctl disable service-name` |
| List running | `systemctl --type=service --state=running` |
| View logs | `journalctl -u service-name` |
| Follow logs | `journalctl -u service-name -f` |
| Check if enabled | `systemctl is-enabled service-name` |

## Practice Exercises

```bash
# 1. Check if SSH is running and enabled
systemctl status sshd
systemctl is-enabled sshd

# 2. List all running services
systemctl --type=service --state=running

# 3. View SSH logs from today
journalctl -u sshd --since today

# 4. Check for any failed services
systemctl --failed

# 5. See what services are enabled at boot
systemctl list-unit-files --type=service --state=enabled | less
```
