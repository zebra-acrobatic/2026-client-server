# Beginner's Guide to Installing Linux (Fedora / RHEL Family)

## Two Ways to Get a Linux System

There are two very different situations where you "install" Linux:

1. **Local/manual installation** - booting an installer (like Fedora's Anaconda) on real or virtual hardware and choosing every option yourself
2. **Cloud deployment** - launching a pre-built image (like Amazon Linux) in a cloud provider such as AWS EC2, where most decisions are made for you or replaced by cloud-specific settings

This guide explains the important options you'll encounter during a manual Fedora/RHEL install, then compares that to deploying Amazon Linux in EC2 - not the exact steps to click through, but what each option means and why it matters.

## Key Options During a Manual Install

Fedora and RHEL both use the **Anaconda** installer, so the same concepts apply to either.

### Language and Locale

Sets the language for the installer and installed system, along with regional formatting like date, time, and keyboard layout. This matters because it affects every user's default experience on the system, not just yours during setup.

### Time Zone

Determines how the system displays and logs timestamps. This is especially important on servers, since log files (checked with `journalctl`, covered in the systemd guide) are much easier to correlate with real events if the time zone is correct.

### Installation Destination (Disk Selection and Partitioning)

Chooses which disk(s) the system installs to, and how that disk is divided into partitions (e.g. `/`, `/home`, `/boot`, swap). Options here include:
- **Automatic partitioning** - the installer picks sensible defaults
- **Custom/manual partitioning** - you define partitions and sizes yourself, useful when you want to separate `/home` from `/` (so a reinstall doesn't wipe user data) or size things for a specific workload

### Disk Encryption

Encrypts the partitions (usually using **LUKS**) so the data can't be read without a passphrase, even if the physical disk is removed and accessed elsewhere. This matters for laptops or any machine that could be physically stolen. The tradeoff is that the system can't boot unattended without the passphrase being entered (or a key stored separately) since the disk can't be read until it's unlocked.

### Software Selection

Chooses the base environment - a minimal server install, a full desktop (like GNOME), or something in between - along with optional add-on package groups. Picking a minimal install for a server reduces attack surface and resource use, since you're not running services you don't need.

### Network and Hostname

Sets the system's hostname and configures network interfaces (often just using DHCP by default). This matters because the hostname identifies the machine on the network and in logs, and network settings determine whether the system can reach the internet or other machines once installed.

### Root Password and User Creation

Sets the root (administrator) password and creates a regular user account. RHEL-family installers generally push you toward creating a normal user and using `sudo`, rather than logging in directly as root all the time, so day-to-day mistakes don't happen with full administrative power.

### Kdump

Configures a crash-dump mechanism that saves the state of memory if the kernel crashes, for later debugging. Useful on production servers where diagnosing a rare crash matters more than the extra disk space it reserves.

## Deploying Amazon Linux in AWS EC2

**EC2** (Elastic Compute Cloud) lets you launch virtual machines ("instances") from pre-built **images** (AMIs - Amazon Machine Images), rather than running an interactive installer. Amazon Linux is AWS's own RHEL-family distribution, built to work smoothly on EC2.

Because you're launching a ready-made image instead of installing from scratch, some of the options above disappear, some are replaced by cloud-native equivalents, and some new options appear that don't exist in a local install.

## Comparing the Two Approaches

| Option (local install) | Local Fedora/RHEL install | Amazon Linux on EC2 |
|---|---|---|
| Language/locale | Chosen during install | Set after boot (defaults to English; changeable via `localectl`) |
| Time zone | Chosen during install | Defaults to UTC; changed after boot or via instance metadata |
| Disk partitioning | Manual or automatic, full control | Mostly fixed by the AMI; you choose EBS volume size/type, not partition layout, at launch |
| Disk encryption | LUKS passphrase, entered at boot | EBS volume encryption, handled transparently by AWS - no passphrase prompt, keys managed via AWS KMS |
| Software selection | Choose minimal/server/desktop and package groups | Fixed by the AMI (Amazon Linux is minimal/server-only by design); you add packages afterward with `dnf` |
| Network configuration | Configured interactively, often DHCP | Defined by a **VPC**, **subnet**, and **security group** chosen at launch - the instance itself just gets DHCP from AWS |
| Root password / user creation | Set during install | No root password by default - you log in as `ec2-user` using an **SSH key pair** chosen at launch, then `sudo` |
| Hostname | Set during install | Auto-assigned by AWS (based on private IP), changeable later |
| Kdump | Configurable during install | Not typically configured by default; can be set up manually after boot |
| Physical/hardware access | You control real or virtual hardware directly | No concept of "hardware" - AWS manages the underlying host; you only control the instance |

### Options That Only Exist on EC2

- **Instance type** - the amount of virtual CPU, memory, and network performance (e.g. `t3.micro`, `m5.large`), chosen at launch instead of being limited by physical hardware you own
- **Security groups** - a firewall-like set of rules controlling what network traffic can reach the instance, configured outside the OS entirely
- **Key pairs** - SSH keys (see the SSH guide) selected at launch time, automatically installed into `~/.ssh/authorized_keys` for the default user - there's no password-based login to set up
- **User data** - a script you can supply at launch time that runs automatically on first boot, used to automate configuration that a local install would normally do interactively

### Why the Difference Matters

A local install assumes you're setting up one specific physical or virtual machine by hand, so it asks you everything up front. A cloud deployment assumes you'll launch many identical, disposable instances quickly, so most "installer" decisions are baked into the AMI ahead of time, and the remaining choices (instance size, network, security group, SSH key) are about how the instance fits into the cloud environment rather than how the OS itself is set up.

## Quick Reference

```
Local install option        -> Cloud (EC2) equivalent
-----------------------------------------------------
Disk partitioning            -> EBS volume size/type at launch
Disk encryption (LUKS)       -> EBS encryption (via AWS KMS)
Software selection           -> Fixed AMI + dnf install after boot
Root password/user setup     -> SSH key pair + ec2-user + sudo
Network configuration        -> VPC / subnet / security group
Hostname                     -> Auto-assigned by AWS
Kdump                        -> Manual setup after boot, if needed
(none - no such concept)     -> Instance type (vCPU/RAM sizing)
(none - no such concept)     -> Security groups (firewall rules)
(none - no such concept)     -> User data (first-boot automation script)
```
