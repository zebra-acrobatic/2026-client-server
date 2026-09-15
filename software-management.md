# Beginner's Guide to Software Management in Fedora

## Three Ways to Get Software

On Fedora there are three common ways to install software, from lowest-level to most convenient:

1. **Compile from source** - build the program yourself from its source code
2. **RPM** - install a pre-built package file directly
3. **DNF** - a package manager that finds, downloads, and installs RPM packages (plus their dependencies) for you

Most of the time you'll use DNF, but it helps to understand what's happening underneath.

## Compiling Software from Source

Sometimes software isn't packaged for Fedora yet, or you want the very latest version. In that case you compile it yourself from source code.

### A Simple Example

Most C programs follow a similar pattern: configure, compile, install.

```bash
# Install the basic tools needed to compile software
sudo dnf install gcc make

# Download and extract the source code
tar -xzf example-1.0.tar.gz
cd example-1.0

# Check the system and prepare the build
./configure

# Compile the source code into a program
make

# Install the compiled program onto the system
sudo make install
```

**What each step does:**
- `./configure` - checks your system has what's needed and generates a `Makefile`
- `make` - reads the `Makefile` and compiles the source code into a binary
- `sudo make install` - copies the finished binary (and any support files) into system directories like `/usr/local/bin`

**Downsides of compiling yourself:**
- No automatic updates
- No automatic dependency handling - if a library is missing, `configure` or `make` just fails and you have to install it yourself
- Nothing to track what got installed or how to cleanly remove it

This is exactly the problem package managers solve.

## RPM: The Package Format

**RPM** (Red Hat Package Manager) is both a file format (`.rpm`) and a low-level tool for installing, querying, and removing software packages. An RPM file bundles:
- The pre-compiled program (so you don't need to build it yourself)
- Metadata - name, version, description
- A list of dependencies it needs to run
- Instructions for where files go on the system

### Installing an RPM Directly

```bash
sudo rpm -i example-1.0-1.fc42.x86_64.rpm
```

If a required dependency is missing, `rpm` will refuse to install and just list what's missing - it does not download anything for you.

### Querying Installed Packages

```bash
# Check if a package is installed, and see its version
rpm -q httpd

# List every file a package installed
rpm -ql httpd

# Show detailed information about a package
rpm -qi httpd

# Find which package owns a file on disk
rpm -qf /etc/ssh/sshd_config
```

### Removing a Package

```bash
sudo rpm -e example
```

### Why RPM Alone Isn't Enough

RPM knows a package's dependencies but won't fetch them. If `example-1.0-1.fc42.x86_64.rpm` needs `libfoo`, and `libfoo` isn't installed, `rpm -i` just fails with a "Failed dependencies" error. You'd have to track down and install `libfoo` yourself first. That's the gap DNF fills.

## DNF: The Package Manager

**DNF** (Dandified YUM) is Fedora's package manager. It sits on top of RPM and adds:
- **Repositories** - online collections of packages it can search and download from
- **Automatic dependency resolution** - installs anything a package needs
- **Updates** - upgrade every installed package with one command
- **Clean removal** - tracks what was installed so it can be removed safely

### Searching and Installing

```bash
# Search for a package by name
dnf search web server

# Install a package (downloads it and any dependencies)
sudo dnf install httpd

# Install a specific version
sudo dnf install httpd-2.4.57
```

### Checking What's Installed

```bash
# Check if a package is installed
dnf list installed httpd

# Show detailed info about a package
dnf info httpd
```

### Updating Software

```bash
# See what updates are available
dnf check-update

# Update everything on the system
sudo dnf upgrade

# Update just one package
sudo dnf upgrade httpd
```

### Removing Software

```bash
# Remove a package
sudo dnf remove httpd

# Remove a package and any dependencies no longer needed
sudo dnf autoremove
```

### Repositories

DNF gets packages from **repositories** (repos) - servers listing available packages and their locations. Repo configuration lives in `/etc/yum.repos.d/`.

```bash
# List enabled repositories
dnf repolist

# List all repositories, including disabled ones
dnf repolist --all
```

## How It All Fits Together

```
Source code  --compile-->  Program (no tracking, no dependency handling)
RPM file     --rpm -i-->   Installed package (tracked, but you resolve dependencies yourself)
DNF          --dnf install--> Finds the RPM in a repo, downloads it, resolves dependencies, installs it
```

DNF is really just a smart wrapper around RPM: when you run `dnf install httpd`, DNF searches its repositories, works out every RPM package needed, downloads them all, and hands them to RPM to actually install.

## Quick Reference

```bash
./configure && make && sudo make install   # compile and install from source

rpm -q package-name                        # check if a package is installed
rpm -ql package-name                       # list files owned by a package
rpm -qf /path/to/file                      # find which package owns a file
sudo rpm -i package.rpm                    # install an RPM file directly
sudo rpm -e package-name                   # remove a package with rpm

dnf search keyword                         # search for a package
sudo dnf install package-name              # install a package with dependencies
dnf list installed package-name            # check if a package is installed
sudo dnf upgrade                           # update all installed packages
sudo dnf remove package-name                # remove a package
sudo dnf autoremove                        # remove unused dependencies
dnf repolist                               # list enabled repositories
```
