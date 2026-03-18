# Scenario: Yokohama - Linux Users Working Together

## 🚩 Issue

Four users (`abe`, `betty`, `carlos`, `debora`) share a project directory `/home/admin/shared`. Two requirements must be met:

1. Each user can read all project files but cannot modify another user's file.
2. All users can append content to `shared/ALL` but cannot alter existing content.

## 🔍 Investigation

### Step 1: Inspect current file permissions
```bash
ls -la /home/admin/shared/
```

_Observation: Each project file is owned by its respective user with group set to the same username (`abe`, `betty`, etc.) and permissions `-rw-r-----`. No shared group exists yet._

## ✅ Root Cause

No shared group existed to grant cross-user read access. The `ALL` file required append-only protection which standard chmod cannot provide - a filesystem-level attribute is needed.

## 🛠 Resolution

**Step 1:** Create a shared group:
```bash
sudo groupadd all
```

**Step 2:** Add all four users to the group:
```bash
sudo usermod -aG all abe
sudo usermod -aG all betty
sudo usermod -aG all carlos
sudo usermod -aG all debora
```

Verify:
```bash
cat /etc/group | grep all
```

**Step 3:** Change the group owner of all project files and `ALL`:
```bash
sudo chgrp all /home/admin/shared/project_*
sudo chgrp all /home/admin/shared/ALL
```

**Step 4:** Grant group write access to `ALL`:
```bash
sudo chmod g+w /home/admin/shared/ALL
```

**Step 5:** Set append-only attribute on `ALL` to prevent modification of existing content:
```bash
sudo chattr +a /home/admin/shared/ALL
```

Verify attributes:
```bash
lsattr /home/admin/shared/ALL
```

**Step 6:** Verify final permissions:
```bash
ls -la /home/admin/shared/
```

## ❌ What Didn't Work

- Applying `chattr +a` before setting the correct group and chmod - once the append-only attribute is set, even root cannot change permissions with `chmod` until the attribute is removed with `chattr -a`.
- **Critical order:** Always set ownership and permissions first, then apply `chattr +a` last.

## 💡 Lessons Learned

- `chattr +a` sets append-only at the filesystem level - even root cannot overwrite existing content. Use `lsattr` to inspect these attributes.
- Order matters: set group ownership and chmod before applying `chattr +a`. Once set, the file is locked and permissions cannot be changed without first removing the attribute.
- `usermod -aG <group> <user>` adds a user to a group without removing them from existing groups. Without `-a` flag, all existing groups are replaced.
- Standard file permissions (`rwx`) control read/write/execute. `chattr` attributes work at a lower filesystem level and override permissions.