# Scenario: Tokyo - Can't Serve Web File

## 🚩 Issue

Apache web server is running but `curl 127.0.0.1:80` returns no response. The file `/var/www/html/index.html` exists with content "hello sadserver".

## 🔍 Investigation

### Step 1: Check Apache service status
```bash
systemctl list-units | grep apache
systemctl status apache2.service
```

_Observation: Service is `active (running)`. Apache process exists and listens on port 80._

### Step 2: Verify port and process
```bash
ss -tulpn
ps aux | grep apache2
```

_Observation: Apache is listening on `*:80`. Process is running as expected._

### Step 3: Check iptables rules
```bash
sudo iptables -L -n -v
```

_Observation: Found a `DROP` rule in the `INPUT` chain blocking all TCP traffic on port 80:_
```bash
DROP  tcp  --  *  *  0.0.0.0/0  0.0.0.0/0  tcp dpt:80
```

### Step 4: Remove the blocking rule
```bash
sudo iptables -L -n --line-numbers
sudo iptables -D INPUT <line_number>
```

_Observation: curl now returns a response, but with `403 Forbidden`._

### Step 5: Investigate 403 Forbidden
403 means the server is reachable but access to the resource is denied - either file permissions or directory permissions are wrong.
```bash
ls -la /var/www/html/
ls -la /var/www/html/index.html
```

_Observation: `index.html` has permissions `600` - only owner (root) can read it. Apache runs as `www-data` and has no read access._
```bash
-rw------- 1 root root  index.html
```

## ❌ What Didn't Work

- Reloading Apache (`systemctl reload apache2.service`) - service config was fine, the problem was at the network and filesystem level.

## ✅ Root Cause

Two separate issues:

1. An iptables `DROP` rule in the `INPUT` chain was silently blocking all incoming TCP traffic on port 80.
2. `index.html` had permissions `600` - Apache's worker process (`www-data`) had no read access to the file, causing a `403 Forbidden` response.

## 🛠 Resolution

**Step 1:** Remove the iptables blocking rule:
```bash
sudo iptables -L -n --line-numbers
sudo iptables -D INPUT <line_number>
```

**Step 2:** Fix file permissions:
```bash
chmod 644 /var/www/html/index.html
```

**Step 3:** Verify the fix:
```bash
curl 127.0.0.1:80
```

_Returns `hello sadserver`._

## 💡 Lessons Learned

- Always check iptables early when a service is running but not responding - a DROP rule leaves no trace in service logs.
- 403 Forbidden means the server is reachable but access is denied at the filesystem level. Check file and directory permissions.
- Apache runs as `www-data` - files must be readable by this user. Standard permissions for web files: `644` (files), `755` (directories).
- Two separate problems can stack - fixing one reveals the next.