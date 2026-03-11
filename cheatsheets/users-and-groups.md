# Linux Users & Groups Management

Essential commands for managing users, groups, and file ownership on Linux systems.

## 1. Inspecting Users & Groups
* `cat /etc/passwd` - List all users with UID, home directory, and shell.
* `cat /etc/group` - List all groups with GID and member list.
* `id <username>` - Show UID, GID, and all groups a user belongs to.
* `groups <username>` - List all groups a specific user belongs to.
* `whoami` - Show the current logged-in user.

## 2. Managing Users
* `sudo useradd <username>` - Create a new user.
* `sudo useradd -m <username>` - Create a new user with a home directory.
* `sudo userdel <username>` - Delete a user.
* `sudo userdel -r <username>` - Delete a user and their home directory.
* `sudo usermod -aG <group> <user>` - Add a user to a group without removing existing group memberships. **Warning:** Without `-a`, all existing groups are replaced.
* `sudo passwd <username>` - Set or change a user's password.

## 3. Managing Groups
* `sudo groupadd <group>` - Create a new group.
* `sudo groupdel <group>` - Delete a group.
* `sudo chgrp <group> <file>` - Change the group owner of a file or directory.
* `sudo chgrp -R <group> <directory>` - Recursively change group owner of a directory.

## 4. File Permissions (chmod)
* `ls -la` - List files with permissions, owner, and group.
* `chmod <permissions> <file>` - Change file permissions.

Permission targets: `u` (user/owner), `g` (group), `o` (others), `a` (all).
```bash
# Symbolic mode
chmod u+x file       # Add execute for owner
chmod g+w file       # Add write for group
chmod o-r file       # Remove read for others
chmod 644 file       # Numeric: rw-r--r--
chmod 755 file       # Numeric: rwxr-xr-x
```

Numeric values: `r=4`, `w=2`, `x=1`. Sum them per target (owner/group/others).

## 5. Filesystem Attributes (chattr / lsattr)
Filesystem-level attributes that work below standard permissions - even root cannot bypass them without removing the attribute first.

* `lsattr <file>` - Show filesystem attributes of a file.
* `sudo chattr +a <file>` - Set append-only: existing content cannot be modified, only new content can be added.
* `sudo chattr -a <file>` - Remove append-only attribute.
* `sudo chattr +i <file>` - Set immutable: file cannot be modified, deleted, or renamed by anyone including root.
* `sudo chattr -i <file>` - Remove immutable attribute.

**Critical:** Always set ownership and chmod before applying `chattr +a` or `+i` — once set, permissions cannot be changed until the attribute is removed.

## 6. Default Permission Mask (umask)
* `umask` - Check the current permission mask for the active session.
* `umask 022` - Standard mask: new directories get `755`, new files get `644`.
* `umask 077` - Strict mask: new files and directories are private to owner only (`700`/`600`).

**How umask works:** Linux starts with maximum permissions (`777` for dirs, `666` for files) and subtracts the umask value. `umask 022` → `777-022=755` for dirs, `666-022=644` for files.

**Where umask is configured:**
* `~/.bashrc`, `~/.profile`, `~/.bash_profile` - Per-user settings, loaded on login.
* `/etc/profile` - System-wide setting, applies to all users on every login shell.

**Note:** Changing `umask` in the current shell is temporary. To make it permanent, edit the appropriate profile file.