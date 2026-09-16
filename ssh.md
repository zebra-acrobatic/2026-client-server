# Beginner's Guide to SSH

## What Is SSH?

**SSH** (Secure Shell) is a protocol that lets you securely connect to and control another computer over a network. Everything you type and everything the server sends back is encrypted, so nobody snooping on the network can read it.

You'll use SSH to:
- Log in to a remote server's command line
- Copy files securely (via SFTP or SCP)
- Run commands on another machine without being physically at it

**Basic connection:**
```bash
ssh username@server-address
```

**Example:**
```bash
ssh alex@192.168.1.50
```

The first time you connect to a new server, you'll see something like this:

```
The authenticity of host '192.168.1.50 (192.168.1.50)' can't be established.
ED25519 key fingerprint is SHA256:abc123...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

This is normal - it's SSH warning you that it has never seen this server before.

## SSH Keys and Fingerprints

### Host Keys and Fingerprints

Every SSH server has its own **host key**, a unique cryptographic identity. The **fingerprint** is a short hash of that key, used so a human can quickly verify it instead of comparing the entire key.

When you accept a fingerprint the first time, SSH saves it in `~/.ssh/known_hosts`. Every time you reconnect, SSH checks the server's key against what's saved. If the key ever changes unexpectedly, you'll see a big warning - this usually means the server was reinstalled, but it can also mean someone is impersonating the server.

You can check a server's fingerprint directly on the server itself:

```bash
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

### User Keys (Key-Based Login)

Instead of typing a password every time, you can use a **key pair**:
- A **private key** (kept secret, stays on your computer, e.g. `~/.ssh/id_ed25519`)
- A **public key** (safe to share, e.g. `~/.ssh/id_ed25519.pub`)

You give the public key to servers you want to log in to. The server can then verify you own the matching private key without the private key ever leaving your machine.

**Generate a key pair:**
```bash
ssh-keygen -t ed25519 -C "alex@example.com"
```

This creates:
- `~/.ssh/id_ed25519` - your private key (never share this)
- `~/.ssh/id_ed25519.pub` - your public key (safe to share)

**Copy your public key to a server so you can log in without a password:**
```bash
ssh-copy-id alex@192.168.1.50
```

This appends your public key to `~/.ssh/authorized_keys` on the remote server.

**View your key's fingerprint:**
```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
```

## Where SSH Is Configured

On Fedora, SSH has two sides, each with its own config file:

| Side | Config file | Purpose |
|------|-------------|---------|
| Server (sshd) | `/etc/ssh/sshd_config` | Controls how the SSH server behaves |
| Client (ssh) | `/etc/ssh/ssh_config` (system-wide) or `~/.ssh/config` (per-user) | Controls how your `ssh` command connects |

Changes to `/etc/ssh/sshd_config` require restarting the `sshd` service to take effect (see the systemd section below).

### Sample Server Options (`/etc/ssh/sshd_config`)

```
# Change the default port (optional, adds a small amount of obscurity)
Port 2222

# Show a message before login
Banner /etc/issue.net

# Disable direct root login (log in as a normal user, then sudo)
PermitRootLogin no

# Disable password logins, require keys only
PasswordAuthentication no

# Limit which users can SSH in
AllowUsers alex alice
```

**Setting up a login banner:**

Create a text file with your message:
```bash
echo "Authorized access only. All activity is logged." > /etc/issue.net
```

Point `sshd_config` at it:
```
Banner /etc/issue.net
```

Then restart `sshd` for the change to apply. Anyone connecting will see the banner before they log in.

### Sample Client Options (`~/.ssh/config`)

Instead of typing long commands, you can define shortcuts:

```
Host myserver
    HostName 192.168.1.50
    User alex
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

Now you can just run:
```bash
ssh myserver
```

## SFTP: Secure File Transfer

**SFTP** (SSH File Transfer Protocol) lets you copy files to and from a remote machine using the same encrypted connection as SSH - no separate server or port needed, it just rides on top of `sshd`.

**Start an interactive SFTP session:**
```bash
sftp alex@192.168.1.50
```

Once connected, you get an FTP-like prompt:

```bash
sftp> ls              # list files on the remote server
sftp> lls              # list files on your local machine
sftp> cd Documents     # change remote directory
sftp> lcd Documents    # change local directory
sftp> get notes.txt    # download a file
sftp> put report.pdf   # upload a file
sftp> exit             # close the session
```

**Copy a single file without an interactive session (`scp`, a simpler alternative):**
```bash
# Copy a local file to the server
scp report.pdf alex@192.168.1.50:/home/alex/

# Copy a file from the server to your machine
scp alex@192.168.1.50:/home/alex/notes.txt .
```

### Restricting Users to SFTP Only

Sometimes you want a user to transfer files but never get a shell. In `/etc/ssh/sshd_config`:

```
Match User uploader
    ForceCommand internal-sftp
    ChrootDirectory /srv/uploads
    AllowTcpForwarding no
    X11Forwarding no
```

This forces that user into an SFTP-only session, locked to `/srv/uploads`.

## Managing SSH with systemd

On Fedora, the SSH server runs as the `sshd` service, managed with `systemctl` just like any other daemon.

**Check if it's running:**
```bash
systemctl status sshd
```

**Start / stop / restart:**
```bash
sudo systemctl start sshd
sudo systemctl stop sshd
sudo systemctl restart sshd
```

**Reload config without dropping existing connections:**
```bash
sudo systemctl reload sshd
```

**Make sure it starts automatically at boot:**
```bash
sudo systemctl enable --now sshd
```

**Check the logs when something goes wrong:**
```bash
journalctl -u sshd -f
```

> After editing `/etc/ssh/sshd_config`, always run `sudo sshd -t` to check for syntax errors before restarting the service - a broken config can lock you out.

```bash
sudo sshd -t
sudo systemctl restart sshd
```

## Quick Reference

```bash
ssh user@host                          # connect to a remote server
ssh-keygen -t ed25519                  # generate a new key pair
ssh-copy-id user@host                  # install your public key on a server
ssh-keygen -lf ~/.ssh/id_ed25519.pub   # show a key's fingerprint
sftp user@host                         # start an interactive file transfer session
scp file user@host:/path/              # copy a single file over SSH
sudo systemctl status sshd             # check the SSH service
sudo systemctl restart sshd            # restart the SSH service
journalctl -u sshd -f                  # watch SSH logs live
```
