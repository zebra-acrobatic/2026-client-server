# Beginner's Guide to NetworkManager

## What Is NetworkManager?

**NetworkManager** is the service that manages network connections on Fedora - Wi-Fi, Ethernet, VPNs, and more. Instead of manually editing network configuration files, you use NetworkManager's tools to create and switch between **connection profiles**.

NetworkManager runs as a background service (managed by systemd) and provides several ways to control it:
- `nmcli` - command-line tool, great for scripts and servers
- `nmtui` - simple text-based menu, great for beginners
- GNOME/KDE network settings - graphical, for desktop use

This guide focuses on `nmcli` and `nmtui`, which work the same whether you're on a desktop or a headless server over SSH.

## Connection Profiles

A **profile** is a saved set of settings for a network connection - things like the interface it applies to, whether it uses DHCP or a static IP, and Wi-Fi credentials. Profiles are stored under `/etc/NetworkManager/system-connections/`.

You can have multiple profiles for the same device (e.g. "home-wifi" and "office-wifi") and just switch between them instead of reconfiguring each time.

## Using nmtui (Text Menu)

`nmtui` gives you a simple full-screen menu, useful when you don't remember exact `nmcli` syntax.

```bash
sudo nmtui
```

From the menu you can:
- **Edit a connection** - create or modify a profile (IP address, DNS, Wi-Fi password, etc.)
- **Activate a connection** - connect or disconnect using a profile
- **Set system hostname**

Use the arrow keys and Enter/Tab to navigate, and Esc to go back.

## Using nmcli (Command Line)

### Checking Status

```bash
# Overall network status
nmcli general status

# List all devices (interfaces) and their state
nmcli device status

# List all saved connection profiles
nmcli connection show

# Show details of one profile
nmcli connection show "home-wifi"
```

**Example output:**
```
DEVICE  TYPE      STATE      CONNECTION
eth0    ethernet  connected  wired-office
wlan0   wifi      disconnected --
```

### Connecting to Wi-Fi

```bash
# List available Wi-Fi networks
nmcli device wifi list

# Connect to a network (creates a profile automatically)
sudo nmcli device wifi connect "MyWiFiName" password "MyPassword123"
```

### Creating a Wired Profile with a Static IP

```bash
sudo nmcli connection add \
  type ethernet \
  ifname eth0 \
  con-name "static-eth0" \
  ipv4.addresses 192.168.1.100/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns 8.8.8.8 \
  ipv4.method manual
```

### Creating a Profile That Uses DHCP

```bash
sudo nmcli connection add \
  type ethernet \
  ifname eth0 \
  con-name "dhcp-eth0" \
  ipv4.method auto
```

### Activating and Deactivating Profiles

```bash
# Bring a profile up
sudo nmcli connection up "static-eth0"

# Take a profile down
sudo nmcli connection down "static-eth0"
```

### Editing an Existing Profile

```bash
# Change a setting on an existing profile
sudo nmcli connection modify "static-eth0" ipv4.dns "1.1.1.1 8.8.8.8"

# Re-activate the profile so the change takes effect
sudo nmcli connection up "static-eth0"
```

### Deleting a Profile

```bash
sudo nmcli connection delete "static-eth0"
```

## Managing NetworkManager with systemd

NetworkManager runs as a systemd service called `NetworkManager`, managed like any other daemon.

**Check if it's running:**
```bash
systemctl status NetworkManager
```

**Start / stop / restart:**
```bash
sudo systemctl start NetworkManager
sudo systemctl stop NetworkManager
sudo systemctl restart NetworkManager
```

**Make sure it starts automatically at boot:**
```bash
sudo systemctl enable --now NetworkManager
```

**Check the logs when something goes wrong:**
```bash
journalctl -u NetworkManager -f
```

> Restarting `NetworkManager` will briefly drop and reconnect your network connections - if you're connected over SSH, prefer `nmcli connection up`/`down` on a specific profile instead of restarting the whole service.

## Quick Reference

```bash
nmtui                                   # simple text-menu interface
nmcli general status                    # overall network status
nmcli device status                     # list interfaces and their state
nmcli connection show                   # list saved profiles
nmcli device wifi list                  # list nearby Wi-Fi networks
nmcli device wifi connect "SSID" password "pass"   # connect to Wi-Fi
nmcli connection up "profile-name"      # activate a profile
nmcli connection down "profile-name"    # deactivate a profile
nmcli connection modify "profile-name" ipv4.dns "1.1.1.1"  # edit a profile
nmcli connection delete "profile-name"  # remove a profile
sudo systemctl status NetworkManager    # check the NetworkManager service
journalctl -u NetworkManager -f         # watch NetworkManager logs live
```
