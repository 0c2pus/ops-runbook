# Scenario: Alexandria - The Vanishing Backups

## 🚩 Issue

A daily backup cron job silently stopped working 3 days ago. The backup script `/opt/backup/backup.sh` is not creating new files in `/var/backups/daily/`. No error emails, no logs - the failure is completely silent.

## 🔍 Investigation

### Step 1: Read the backup script
```bash
cat /opt/backup/backup.sh
```

_Observation: The script uses a lock file mechanism at `/opt/backup/backup.lock`. If the lock file exists, the script exits immediately with an error._

### Step 2: Check if the lock file exists
```bash
ls -la /opt/backup/
```

_Observation: `backup.lock` exists with a timestamp from 3 days ago - exactly when backups stopped. A previous run crashed without cleaning up the lock file, blocking all subsequent runs._

### Step 3: Check the cron job
```bash
crontab -l
```

_Observation: Found the following entry:_

```bash
*/5 * * * * /opt/backup/old_backup.sh > /dev/null 2>&1
```

_Two problems: cron is calling `old_backup.sh` which does not exist, and `> /dev/null 2>&1` silences all output including errors - this is why there were no error logs or emails._

## ❌ What Didn't Work

- Simply deleting the lock file is not enough - cron was pointing to a non-existent script, so backups would still not run.

## ✅ Root Cause

Two separate issues caused the failure:

1. A stale lock file at `/opt/backup/backup.lock` left by a crashed backup run was blocking all subsequent executions.
2. The cron job was referencing a non-existent script `old_backup.sh` instead of the correct `backup.sh`, with all output redirected to `/dev/null` - making the failure completely silent.

## 🛠 Resolution

**Step 1:** Remove the stale lock file:
```bash
rm -f /opt/backup/backup.lock
```

**Step 2:** Fix the cron job:
```bash
crontab -e
```

Replace the broken entry with:
```bash
* * * * * /opt/backup/backup.sh
```

**Step 3:** Verify backups are being created:
```bash
ls -lt /var/backups/daily/
```

_New backup files appear every minute._

## 💡 Lessons Learned

- A lock file left by a crashed script silently blocks all future runs - always check for stale lock files when a scheduled job stops working.
- `> /dev/null 2>&1` suppresses all output including errors. Never use it on critical jobs without a proper logging alternative.
- `crontab -l` shows the current user's cron jobs - always check this early when diagnosing silent scheduled job failures.
- In production, use `flock` instead of manual lock files - it automatically releases the lock even if the script crashes.