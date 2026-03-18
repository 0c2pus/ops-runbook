# Scenario: Nuuk - More SSH Troubles

## 🚩 Issue

`ssh admin@127.0.0.1` fails with `Permission denied (publickey)` despite the SSH key pair existing in `~/.ssh/`.

## 🔍 Investigation

### Step 1: Attempt SSH connection and read the error
```bash
ssh admin@127.0.0.1
```

_Observation: Two errors appear:_
```bash
hostkeys_foreach failed for /home/admin/.ssh/known_hosts: Permission denied
Permission denied (publickey)
```

_Both errors point to permission problems with the `.ssh` directory._

### Step 2: Check directory permissions
```bash
ls -la ~
```

_Observation:_
```bash
d--------- 2 admin admin 4096 .ssh
```

_The `.ssh` directory has no permissions for anyone - not even the owner. SSH client cannot read the private key or `authorized_keys`._

## ✅ Root Cause

The `.ssh` directory had permissions `000` - completely locked. SSH requires strict ownership and permission rules to function. The SSH client refused to use the keys because it could not read the directory contents.

## 🛠 Resolution

Restore correct permissions on the `.ssh` directory:
```bash
sudo chmod 700 ~/.ssh/
```

Verify:
```bash
ls -la ~
# Should show: drwx------ 2 admin admin .ssh
```

Test SSH connection:
```bash
ssh admin@127.0.0.1
```

## 💡 Lessons Learned

- SSH enforces strict permission requirements as a security feature. If permissions are too open or too closed, SSH refuses to use the keys.
- Required permissions for SSH to work:
  - `~/.ssh/` directory: `700` (rwx------)
  - `~/.ssh/authorized_keys`: `600` (rw-------)
  - `~/.ssh/id_*` private keys: `600` (rw-------)
  - `~/.ssh/id_*.pub` public keys: `644` (rw-r--r--)
- `Permission denied (publickey)` does not always mean the key is wrong - it can mean SSH cannot read the key due to permission issues.