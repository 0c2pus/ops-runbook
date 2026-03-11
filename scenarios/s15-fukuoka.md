# Scenario: Fukuoka - Forbidden Association

## 🚩 Issue

`curl localhost` returns 404 Not Found. Nginx is running but not serving the correct content.

## 🔍 Investigation

### Step 1: Check nginx service status
```bash
systemctl status nginx.service
```

_Observation: Service is `active (running)`._

### Step 2: Check nginx error logs
```bash
sudo journalctl -u nginx
```

_Observation: No critical errors — service started successfully._

### Step 3: Inspect the web root directory
```bash
sudo ls -la /var/www/html/
```

_Observation: `index.html` is a symlink pointing to `/opt/site-content/real_index.html`._

### Step 4: Check directory permissions
```bash
ls -la /var/www/
```

_Observation: `drwxr-xr-- root root` — others have no execute permission. Nginx runs as `www-data` which is not in the `root` group, so it cannot enter the directory at all._

### Step 5: Fix directory access and retest
```bash
sudo chmod o+x /var/www/
curl localhost
```

_Observation: Now returns `403 Forbidden` - progress. Nginx can enter the directory but cannot read the file._

### Step 6: Check symlink target permissions
```bash
ls -la /opt/site-content/
```

_Observation: `real_index.html` has permissions `-rw-r-----` - others have no read access. `www-data` cannot read the file._

### Step 7: Check the directory itself
```bash
ls -la /opt/
```

_Observation: `/opt/site-content/` directory permissions are correct - no changes needed there._

## ❌ What Didn't Work

- `chmod -R 666 /var/www` - avoided because recursive permission changes on web directories are dangerous and grant write access to everyone.

## ✅ Root Cause

Two separate permission issues blocked nginx (`www-data`) from serving the file:

1. `/var/www/` was missing execute permission for others - nginx could not enter the directory.
2. `/opt/site-content/real_index.html` was missing read permission for others — nginx could not read the symlink target.

## 🛠 Resolution

**Step 1:** Add execute permission for others on the web root:
```bash
sudo chmod o+x /var/www/
```

**Step 2:** Add read permission for others on the target file:
```bash
sudo chmod o+r /opt/site-content/real_index.html
```

**Step 3:** Verify:
```bash
curl localhost
```

_Returns `Welcome to the Real Site!`._

## 💡 Lessons Learned

- Always apply minimal permissions — `o+x` for directories to allow traversal, `o+r` for files to allow reading. Never use `chmod -R 666` on web directories.
- Symlinks introduce an additional permission chain - nginx must have access to every directory and file in the chain, not just the symlink itself.
- Each service runs as its own system user (`www-data`, `postgres`, etc.) for security isolation. This is the principle of least privilege.
- When debugging permission errors: 403 means the server is reachable but access is denied — trace the full path from web root to actual file.