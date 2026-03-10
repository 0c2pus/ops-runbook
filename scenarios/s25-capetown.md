# Scenario: Cape Town - Borked Nginx

## 🚩 Issue

Nginx is installed and managed by systemd but `curl -I 127.0.0.1:80` returns `Connection refused`. The service fails to start.

## 🔍 Investigation

### Step 1: Check service status
```bash
systemctl status nginx.service
```

_Observation: `failed (Result: exit-code)`. Log shows:_
```bash
nginx: [emerg] unexpected ";" in /etc/nginx/sites-enabled/default:1
```

### Step 2: Validate nginx configuration
```bash
sudo nginx -t
```

_Observation: Confirms syntax error at line 1 of `/etc/nginx/sites-enabled/default` - a stray `;` at the very beginning of the file._

### Step 3: Fix the syntax error
```bash
sudo vim /etc/nginx/sites-enabled/default
```

_Removed the stray `;` from line 1._

### Step 4: Start nginx and test
```bash
sudo systemctl start nginx.service
curl -Is 127.0.0.1:80 | head -1
```

_Observation: Service starts but returns `HTTP/1.1 500 Internal Server Error`._

### Step 5: Check error logs
```bash
sudo tail /var/log/nginx/error.log
```

_Observation: New error appears:_
```bash
[alert] nginx: socketpair() failed (24: Too many open files)
[crit] open() failed (24: Too many open files)
```

### Step 6: Check system limits
```bash
ulimit -a
```

_Observation: `open files (-n) 1024` - very low for a web server._

### Step 7: Check nginx service file for file limit setting
```bash
cat /etc/systemd/system/nginx.service
```

_Observation: `LimitNOFILE=10` - deliberately set to an extremely low value, overriding the system default._

## ❌ What Didn't Work

- Restarting nginx without fixing the config - service failed immediately due to syntax error.
- Fixing only the syntax error - revealed a second hidden problem with open file limits.

## ✅ Root Cause

Two separate issues:

1. A stray `;` at line 1 of `/etc/nginx/sites-enabled/default` caused a syntax error that prevented nginx from starting at all.
2. `LimitNOFILE=10` in the nginx systemd unit file set an extremely low open file limit, preventing nginx worker processes from opening any files after startup.

## 🛠 Resolution

**Step 1:** Remove the stray `;` from `/etc/nginx/sites-enabled/default`:
```bash
sudo vim /etc/nginx/sites-enabled/default
```

**Step 2:** Increase the open file limit in the nginx service file:
```bash
sudo vim /etc/systemd/system/nginx.service
# Change: LimitNOFILE=10
# To:     LimitNOFILE=65536
```

**Note:** In production, prefer editing
`/etc/systemd/system/nginx.service.d/override.conf` instead of modifying the service file directly - system package updates can overwrite it.

**Step 3:** Reload systemd and restart nginx:
```bash
sudo systemctl daemon-reload
sudo systemctl restart nginx.service
```

**Step 4:** Verify the fix:
```bash
curl -Is 127.0.0.1:80 | head -1
```

_Returns `HTTP/1.1 200 OK`._

## 💡 Lessons Learned

- Always use `nginx -t` to validate config before restarting - it pinpoints the exact file and line number of syntax errors.
- `Too many open files` in a web server context usually means a `LimitNOFILE` value that is too low in the systemd unit file, not a system-wide problem.
- Always run `systemctl daemon-reload` after modifying any systemd unit file - without it, systemd uses the old cached version.
- Prefer `/etc/systemd/system/<name>.service.d/override.conf` over editing the unit file directly - package updates will overwrite direct edits.