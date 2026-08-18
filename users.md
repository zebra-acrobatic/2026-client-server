# Linux Users and Access Control

## What are User Accounts?

**User accounts** are individual identities on a Linux system that determine what a person can access and what actions they can perform. Every person (or service) that uses a Linux system has their own account with specific permissions.

**Why user accounts matter:**
- **Security** - separate users can't access each other's files without permission
- **Accountability** - the system tracks who made changes
- **Organization** - each user has their own home directory and settings
- **Resource management** - system administrators can control what resources each user can access

**Key concept:** Unlike Windows where many users have administrator access, Linux encourages using regular user accounts for daily work and only elevating privileges when necessary.

## User Space vs. Root Access

Linux has two main levels of access:

### Regular Users (User Space)
- Have access to their own files and directories
- Can run most programs
- **Cannot** modify system files or install software system-wide
- **Cannot** access other users' private files
- Safer for everyday work

### Root (Superuser)
- Has **complete control** over the system
- Can modify any file, install/remove software, change system settings
- Can access any user's files
- Represented by UID 0
- Username is always `root`

**Important rule:** Never log in as root for routine work. Use your regular account and escalate privileges only when needed using `sudo`.

**Why this matters:** Running as root all the time means one mistake (like `rm -rf /` typo) could destroy your entire system. Regular users are protected from these catastrophic errors.

## Understanding User Information

Every user account has specific attributes:

| Attribute | Description | Example |
|-----------|-------------|---------|
| **Username** | The name you log in with | `john`, `ubuntu`, `admin` |
| **UID** (User ID) | Unique number identifying the user | `1000`, `1001` |
| **GID** (Group ID) | Primary group number | `1000` |
| **Real name** | Full name or description | `John Smith` |
| **Home directory** | User's personal directory | `/home/john` |
| **Shell** | Command interpreter the user gets | `/bin/bash` |

### The `id` Command

Check your current user information:
```bash
id
```

**Example output:**
```
uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu),27(sudo)
```

This shows:
- UID is 1000
- Username is `ubuntu`
- Primary group is 1000 (also called `ubuntu`)
- User is also a member of the `sudo` group (ID 27)

**Try it yourself:** Run `id` to see your current user details.

## The `/etc/passwd` File

This file contains a list of all user accounts on the system. It's readable by everyone but only writable by root.

**View all users:**
```bash
cat /etc/passwd
```

**Example line:**
```
ubuntu:x:1000:1000:Ubuntu User:/home/ubuntu:/bin/bash
```

**Breaking it down:**
```
username : password : UID : GID : real name : home directory : shell
```

- `ubuntu` - username
- `x` - password is stored elsewhere (in `/etc/shadow`, a protected file)
- `1000` - User ID
- `1000` - Primary Group ID
- `Ubuntu User` - real name/description
- `/home/ubuntu` - home directory location
- `/bin/bash` - default shell

**Note:** You'll see many system accounts (like `daemon`, `bin`, `sys`) that aren't for people to log into - they're used by system services.

**Search for a specific user:**
```bash
grep "ubuntu" /etc/passwd
```

## Groups in Linux

**Groups** are collections of users that share common permissions. Instead of giving permissions to individual users, you can give them to a group, and all members inherit those permissions.

### Why Groups?

Imagine 10 developers need access to a project directory. Instead of:
- Adding permissions for each person individually (10 times)
- Having to remember to update permissions when someone joins/leaves

You can:
- Create a `developers` group
- Give the group access to the directory
- Add users to the group

**Common groups:**
- `sudo` - can use the `sudo` command to get root privileges
- `wheel` - similar to sudo on some distributions
- `adm` - can read log files
- `docker` - can use Docker without sudo

### Primary vs. Secondary Groups

- **Primary group** - your main group, typically created when your account was created (same name as username)
- **Secondary groups** - additional groups you're a member of

**View your groups:**
```bash
groups
```

**View groups with IDs:**
```bash
id
```

### The `/etc/group` File

Lists all groups on the system:

```bash
cat /etc/group
```

**Example line:**
```
sudo:x:27:ubuntu,john
```

Breaking it down:
- `sudo` - group name
- `x` - password placeholder (rarely used)
- `27` - Group ID (GID)
- `ubuntu,john` - members of this group

**Find groups a specific user belongs to:**
```bash
groups username
```

## User Login and Authentication

Linux systems support two main ways users can log in and access the system:

### Local Login (Direct Access)

**Local login** means you're physically at the computer or using its console directly. This is what happens when:
- You boot up a Linux machine and see a login prompt
- You're at the keyboard and monitor of a Linux computer
- You're using a virtual machine console window
- You switch to a different terminal with `Ctrl+Alt+F1` through `F6`

**Local login process:**
1. System displays a login prompt: `login:`
2. You enter your username
3. System asks for password: `Password:`
4. You type your password (it won't show on screen)
5. If correct, you get a shell session

**Graphical login (GUI):**
- Modern Linux desktop systems show a graphical login screen
- You select your username and type your password in a window
- After login, you get a desktop environment
- You can open a terminal from within the GUI to get a command line

**Why local login matters:**
- Fastest way to access a physical Linux machine
- Useful for troubleshooting when network is down
- Required for initial setup and emergency recovery
- Always available even if SSH service isn't running

**Try it:** If you're using a Linux desktop, open a terminal with `Ctrl+Alt+T` or search for "Terminal" in your applications menu. You're already logged in locally!

### Remote Login (SSH)

**SSH** (Secure Shell) is how users can connect to Linux servers remotely over a network. It provides an encrypted connection so your password and commands can't be intercepted.

**When you use SSH:**
- Connecting to a cloud server (like AWS EC2)
- Managing a server in a data center from your desk
- Accessing a lab computer from home
- Working on a Linux server that has no monitor attached

**Basic SSH Connection:**

```bash
ssh username@hostname
```

**Examples:**
```bash
ssh ubuntu@192.168.1.100      # Connect to server by IP address
ssh user@server.example.com   # Connect to server by hostname
ssh -i mykey.pem ubuntu@10.0.0.5  # Connect using a specific SSH key
```

**What happens when you SSH:**
1. Your computer contacts the remote server
2. Server asks for authentication (password or SSH key)
3. You provide credentials
4. If successful, you get a shell session on the remote server
5. You can run commands as if you were sitting at that server
6. Type `exit` or press `Ctrl+D` to disconnect

**Key difference from local login:**
- Local: You're typing on the actual machine's keyboard
- SSH: Your keystrokes are encrypted and sent over the network to the remote machine

### Authentication Methods

Both local and remote logins use authentication to verify your identity:

#### Password Authentication

**How it works:**
- You type your password
- System checks it against stored encrypted password
- If match, you're logged in

**Used for:**
- Local login (typing password at console)
- SSH when no key is configured

**Pros:** Simple, no setup required  
**Cons:** Can be guessed, vulnerable to brute-force attacks over network

#### SSH Key Authentication (Remote Only)

**SSH keys** are a more secure alternative to passwords for remote access. They use cryptographic key pairs:

- **Private key** - stays on your computer, never shared (like a physical key you carry)
- **Public key** - placed on servers you want to access (like installing your lock on a door)

**Why SSH keys are better than passwords:**
- Can't be guessed or brute-forced like passwords
- Can't be stolen by keyloggers or network sniffing
- More convenient - no typing passwords repeatedly for each SSH connection
- Can be protected with a passphrase for extra security
- One key can authenticate to many servers

**How SSH Keys Work:**

```
Your Computer                   Remote Server
┌─────────────┐                ┌──────────────┐
│ Private key │  <──encrypted  │  Public key  │
│  (secret)   │    challenge──>│  (shared)    │
└─────────────┘                └──────────────┘
```

1. You initiate SSH connection
2. Server sends a challenge encrypted with your public key
3. Your private key decrypts it and responds
4. Server confirms you have the matching private key and grants access
5. No password needed!

**Key concepts:**
- Private key = your identity, keep it secret (like your house key)
- Public key = distributed to servers, safe to share (like giving a copy of your lock)
- One private key can unlock many servers (if your public key is on them)
- SSH keys only work for SSH - local logins still use passwords

**SSH Key Location:**

Keys are typically stored in your home directory:
```
~/.ssh/id_rsa       # Private key (RSA type)
~/.ssh/id_rsa.pub   # Public key (RSA type)
~/.ssh/id_ed25519   # Private key (ED25519 type, newer/better)
~/.ssh/id_ed25519.pub   # Public key (ED25519 type)
~/.ssh/authorized_keys  # Public keys allowed to log into THIS server
```

**Important:** The `.ssh` directory and private keys should have restricted permissions:
```bash
chmod 700 ~/.ssh          # Only you can read/write/access
chmod 600 ~/.ssh/id_rsa   # Only you can read/write private key
chmod 644 ~/.ssh/id_rsa.pub  # Public key can be readable by others
```

**View your public key:**
```bash
cat ~/.ssh/id_ed25519.pub
```

This is safe to view and share - it's public!

**Example public key looks like:**
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJl3dIeudNqd0DMRD6OIIeoTnbJYDk+OFNUZs9VjY+8v user@hostname
```

### Comparing Local vs Remote Login

| Aspect | Local Login | SSH (Remote) Login |
|--------|-------------|-------------------|
| **Location** | At the machine | From another computer |
| **Connection** | Direct keyboard/monitor | Over network |
| **Security** | Physical access required | Encrypted network connection |
| **Authentication** | Username + password | Username + password OR SSH key |
| **Speed** | Instant | Depends on network |
| **When used** | Initial setup, recovery, desktop use | Server management, cloud systems |
| **Availability** | Always works | Requires network and SSH service |

## Privilege Escalation with `sudo`

**`sudo`** means "Superuser DO" - it temporarily gives you root privileges to run a single command.

### Why `sudo` Exists

Instead of:
- Logging in as root (dangerous)
- Staying logged in as root (very dangerous)

You:
- Work as a regular user (safe)
- Use `sudo` only when needed (controlled)
- System logs exactly what you did (accountable)

### Basic `sudo` Usage

```bash
sudo command
```

**Examples:**

```bash
# Update system packages (requires root)
sudo apt update

# Edit a system file (requires root)
sudo vim /etc/hosts

# Read a protected log file
sudo cat /var/log/auth.log

# Install software (requires root)
sudo apt install nginx
```

**What happens:**
1. You type `sudo command`
2. System asks for **your password** (not root's password)
3. If you're in the `sudo` group, command runs as root
4. You have root privileges for that command only
5. System logs the action

**Sudo timeout:** After entering your password once, `sudo` remembers for about 15 minutes so you don't have to type it repeatedly.

### Common `sudo` Patterns

**Read a protected file:**
```bash
sudo cat /var/log/secure
```

**Edit a system configuration:**
```bash
sudo vim /etc/ssh/sshd_config
```

**Run multiple commands as root:**
```bash
sudo bash
```
This gives you a root shell. **Be careful!** Exit with `exit` when done.

**Run a command as a different user:**
```bash
sudo -u username command
```

### Who Can Use `sudo`?

Only users in the **`sudo`** group (or `wheel` on some systems) can use sudo.

**Check if you're in the sudo group:**
```bash
groups
```

Look for `sudo` in the output.

**Alternative check:**
```bash
id | grep sudo
```

### What if `sudo` Fails?

**Error: "user is not in the sudoers file"**
- You're not in the `sudo` group
- Ask your system administrator to add you
- On cloud systems, make sure you're using the correct default username

**Error: "sudo: command not found"**
- Rare, but sudo might not be installed
- Contact your administrator

## File Ownership and Permissions

Every file and directory is owned by a user and a group.

**Check ownership:**
```bash
ls -lah
```

**Example output:**
```
drwxr-xr-x  2 ubuntu ubuntu 4096 Jan 15 10:30 Documents
-rw-r--r--  1 ubuntu ubuntu  220 Jan 15 10:28 .bashrc
```

Breaking it down:
- `ubuntu ubuntu` - first is the owner user, second is the owner group
- You can only modify files you own (or have permission to access)
- Root can access everything

**Why this matters:** When working with files, you'll often need to check ownership to understand why you can or can't access something.

## Checking User Activity

### View Processes You Own

**See what processes are running under your account:**
```bash
ps -ef | grep $USER
```

Or simpler:
```bash
ps -u $USER
```

This shows:
- Programs you're currently running
- Background processes
- Shell sessions

**Why check this:** Helps you see what's consuming resources under your account or find processes you need to stop.

### Currently Logged-in Users

**See who's logged into the system:**
```bash
who
```

**More detailed view:**
```bash
w
```

This shows what each user is doing.

## Common User Commands Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `id` | Show user and group IDs | `id` |
| `whoami` | Show current username | `whoami` |
| `groups` | Show groups you belong to | `groups` |
| `sudo command` | Run command as root | `sudo apt update` |
| `su - user` | Switch to another user | `su - ubuntu` |
| `passwd` | Change your password | `passwd` |
| `who` | Show logged-in users | `who` |
| `ps -u user` | Show user's processes | `ps -u ubuntu` |

## Security Best Practices

1. **Use SSH keys** instead of passwords when possible
2. **Never share your private key** - it's like sharing your password
3. **Use strong passwords** - even with SSH keys as a backup
4. **Don't run as root** - use `sudo` for specific commands only
5. **Protect your SSH directory** - ensure `~/.ssh` has 700 permissions
6. **Log out when done** - especially on shared systems
7. **Check what groups you're in** - don't have more privileges than needed
8. **Use sudo carefully** - understand what a command does before running it with sudo

## Practice Exercise

Try these commands to understand your user environment:

```bash
# 1. Check your user information
id
whoami
groups

# 2. View your entry in /etc/passwd
grep "^$USER:" /etc/passwd

# 3. Check your group information
grep "^$USER:" /etc/group

# 4. See your home directory contents and ownership
ls -lah ~

# 5. View your running processes
ps -u $USER

# 6. Try reading a protected file (should fail)
cat /var/log/auth.log

# 7. Try with sudo (should work if you're in sudo group)
sudo cat /var/log/auth.log

# 8. Check if you have an SSH key
ls -la ~/.ssh/
```

## Troubleshooting Common Issues

**Can't access a file:** Check ownership with `ls -l filename`. You might need `sudo` or the file might belong to another user.

**`sudo` doesn't work:** Check if you're in the sudo group with `groups`. If not, you need an administrator to add you.

**SSH connection refused:** 
- Check if the server is running
- Verify you're using the correct username
- Ensure SSH service is running on the remote server

**Permission denied when running a command:**
- Check if you need `sudo`
- Verify file permissions with `ls -l`
- Make sure you're in the right directory

**Forgot which user you are:** Run `whoami` to check current user.

## Key Takeaways

1. **Every action on Linux is done by a user** - even system processes
2. **Regular users are limited** - this protects the system
3. **Root (UID 0) can do anything** - use sparingly
4. **Groups simplify permission management** - users inherit group permissions
5. **`sudo` is safer than root** - temporary privilege elevation with logging
6. **SSH keys are more secure** than passwords for remote access
7. **The sudo group is special** - membership grants privilege escalation ability

---