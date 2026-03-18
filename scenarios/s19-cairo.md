# Scenario: Cairo - Fix a Broken systemd Timer

## 🚩 Issue

A health check script at `/opt/scripts/health.sh` is supposed to run every 10 seconds via a systemd timer and write `STATUS: OK` to `/var/log/health.log`.
The log file is not being updated.

## 🔍 Investigation

### Step 1: Check active timers
```bash
sudo systemctl list-timers
```

_Observation: No `health` timer in the list. The timer exists but is not active._

### Step 2: Check the health units status
```bash
sudo systemctl status health.timer
sudo systemctl status health.service
```

_Observation: Both units are `inactive (dead)` and `disabled`. The timer
triggers the service - starting only the service would run it once without
repeat._

### Step 3: Enable and start the timer
```bash
sudo systemctl enable health.timer
sudo systemctl start health.timer
```

_Observation: Timer is now active. But logs still show `STATUS: FAILED`._

### Step 4: Investigate why the script fails

The script does `curl http://localhost` and checks for "Welcome to nginx".
Check if Nginx is running:
```bash
sudo systemctl status nginx
```

_Observation: Nginx is `active (running)`._

### Step 5: Test curl manually
```bash
curl --max-time 3 -v http://localhost
```

_Observation: Connection timeout. Nginx is running and listening on port 80 but not responding to local requests._

### Step 6: Check iptables filter rules
```bash
sudo iptables -L -n -v
```

_Observation: Found a `DROP` rule in the `OUTPUT` chain blocking all TCP traffic to `127.0.0.1:80`:_
```bash
DROP  tcp  --  *  *  0.0.0.0/0  127.0.0.1  tcp dpt:80 /* The hidden problem (IPv4) */
```

## ❌ What Didn't Work

- Checked nat table first (`sudo iptables -t nat -L -n -v`) - no blocking rules there. The problem was in the default filter table, visible without `-t nat` flag.
- Tried deleting from nat table (`sudo iptables -t nat -D OUTPUT 1`) - had no effect because the blocking rule was in the filter table.

## ✅ Root Cause

Two separate problems:

1. `health.timer` and `health.service` were disabled and not running.
2. An iptables rule in the filter `OUTPUT` chain was dropping all local TCP traffic to port 80, causing `curl http://localhost` to time out and the script to always return `STATUS: FAILED`.

## 🛠 Resolution

**Step 1:** Enable and start the timer:
```bash
sudo systemctl enable health.timer
sudo systemctl start health.timer
```

**Step 2:** Find the line number of the blocking rule:
```bash
sudo iptables -L OUTPUT -n -v --line-numbers
```

**Step 3:** Delete the blocking rule from the filter OUTPUT chain:
```bash
sudo iptables -D OUTPUT <line_number>
```

**Step 4:** Verify the fix:
```bash
tail -f /var/log/health.log
```

_Log now shows `STATUS: OK` every 10 seconds._

## 💡 Lessons Learned

- `systemctl list-timers` shows only active timers. Use `systemctl status <unit>` to find disabled units that exist but don't run.
- Always `enable` a timer, not just `start` - without `enable` the timer won't survive a reboot.
- `iptables -L` without `-t` shows the default filter table. `-t nat` shows the NAT table. They are separate and serve different purposes.
- A service can be `active (running)` and still not respond to requests if firewall rules block traffic at the kernel level.