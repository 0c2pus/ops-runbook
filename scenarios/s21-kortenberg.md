# Scenario: Kortenberg - Can't Touch This!

## 🚩 Issue

The `admin` user cannot list new directories or write into new files. Every newly created directory gets permissions `000`.

## 🔍 Investigation

### Step 1: Reproduce the problem
```bash
mkdir test_dir
ls -la
```

_Observation: New directory has permissions `d---------` - no access for anyone._

### Step 2: Check current umask value
```bash
umask
```

_Observation: `0777` - maximum possible mask. Linux subtracts umask from default permissions (`777` for dirs, `666` for files), resulting in `000` for everything._

### Step 3: Find where umask is set permanently
```bash
cat ~/.bashrc | grep umask
cat ~/.profile | grep umask
cat /etc/profile | grep umask
```

_Observation: `/etc/profile` contains `umask 777` at the end of the file - this is a system-wide setting that applies to all login shells._

## ✅ Root Cause

`umask 777` was added to `/etc/profile`, causing all new files and directories to be created with permissions `000`. Since `/etc/profile` is loaded on every login, the problem persisted across sessions.

## 🛠 Resolution

**Step 1:** Fix the current session immediately:
```bash
umask 022
```

**Step 2:** Fix permanently by editing `/etc/profile`:
```bash
sudo vim /etc/profile
# Change: umask 777
# To:     umask 022
```

**Step 3:** Verify in current session:
```bash
mkdir test_fix
ls -la
# Should show: drwxr-xr-x
```

## 💡 Lessons Learned

- `umask` defines the permissions that are *removed* from newly created files and directories. `umask 022` → dirs get `755`, files get `644`.
- `umask 777` removes all permissions - every new file and directory gets `000`. This is a common sabotage pattern in CTF and interview scenarios.
- Changes to `umask` in the current shell are temporary. For permanent effect, set it in `/etc/profile` (system-wide) or `~/.profile` (user-specific).
- `/etc/profile` is loaded for every login shell - changes there affect all users on the system.