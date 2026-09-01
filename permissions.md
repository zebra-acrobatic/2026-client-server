# Basic Linux Permissions

## What Are Linux Permissions?

Every file and directory in Linux has three types of permissions for three types of users:

**Permission Types:**
- **r** (read) - view file contents or list directory contents
- **w** (write) - modify file or create/delete files in directory
- **x** (execute) - run file as a program or enter directory

**User Types:**
- **Owner** (u) - the user who owns the file
- **Group** (g) - users in the file's group
- **Others** (o) - everyone else

## Reading Permissions

When you run `ls -l`, you'll see something like:
```
-rw-r--r-- 1 user group 1024 Jan 15 10:30 myfile.txt
drwxr-xr-x 2 user group 4096 Jan 15 10:30 mydir
```

Breaking down `-rw-r--r--`:
- First character: `-` = file, `d` = directory
- Next 3: `rw-` = owner can read and write
- Next 3: `r--` = group can only read
- Last 3: `r--` = others can only read

## chmod - Change Permissions

### Using Numbers (Numeric Mode)
Each permission has a number:
- **r** = 4
- **w** = 2
- **x** = 1

Add them up for each user type:

| Number | Permission | Meaning |
|--------|------------|---------|
| 7 | rwx | read + write + execute (4+2+1) |
| 6 | rw- | read + write (4+2) |
| 5 | r-x | read + execute (4+1) |
| 4 | r-- | read only |
| 0 | --- | no permissions |

**Examples:**
```bash
# Make a script executable for everyone
chmod 755 script.sh
# Owner: rwx (7), Group: r-x (5), Others: r-x (5)

# Make a file readable and writable only by owner
chmod 600 private.txt
# Owner: rw- (6), Group: --- (0), Others: --- (0)

# Make a file readable by everyone, writable by owner
chmod 644 document.txt
# Owner: rw- (6), Group: r-- (4), Others: r-- (4)
```

### Using Letters (Symbolic Mode)
```bash
# Add execute permission for owner
chmod u+x script.sh

# Remove write permission for group and others
chmod go-w file.txt

# Add read permission for everyone
chmod a+r document.txt

# Set exact permissions: owner can read/write, others can read
chmod u=rw,go=r file.txt
```

**Letter shortcuts:**
- `u` = owner (user)
- `g` = group
- `o` = others
- `a` = all (everyone)
- `+` = add permission
- `-` = remove permission
- `=` = set exact permission

## chown - Change Ownership

### Basic Usage
```bash
# Change owner of a file
chown username file.txt

# Change owner and group
chown username:groupname file.txt

# Change only the group
chown :groupname file.txt
# or use chgrp
chgrp groupname file.txt
```

### Common Examples
```bash
# Change owner of a directory and all its contents
chown -R username directory/

# Change owner and group of a web directory
chown -R www-data:www-data /var/www/html/

# Give ownership of a file to another user
chown john report.txt
```

## Quick Reference Commands

```bash
# View permissions
ls -l filename

# Make a script executable
chmod +x script.sh

# Make a file private (owner only)
chmod 600 secretfile.txt

# Standard permissions for web files
chmod 644 index.html

# Standard permissions for directories
chmod 755 mydir/

# Change owner of a file
chown username file.txt

# Change owner recursively in a directory
chown -R username:groupname directory/
```

## Common Permission Patterns

| Use Case | Command | Permissions |
|----------|---------|-------------|
| Executable script | `chmod 755` | rwxr-xr-x |
| Private file | `chmod 600` | rw------- |
| Shared file | `chmod 644` | rw-r--r-- |
| Public directory | `chmod 755` | rwxr-xr-x |
| Private directory | `chmod 700` | rwx------ |

## Tips for Beginners

1. **Use `ls -l`** to check permissions before and after changes
2. **Be careful with chmod 777** - this gives everyone full access (usually a security risk!)
3. **Use `-R` flag** with chmod/chown to apply changes to all files in a directory
4. **You need sudo** for changing ownership or modifying files you don't own

## Practice Examples

```bash
# Create a test file
touch testfile.txt

# Check its permissions
ls -l testfile.txt

# Make it readable/writable by owner only
chmod 600 testfile.txt

# Check permissions again
ls -l testfile.txt

# Add execute permission for owner
chmod u+x testfile.txt

# Check final permissions
ls -l testfile.txt
```
