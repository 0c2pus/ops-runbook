# Cron - Task Scheduling

Cron is a time-based job scheduler. It runs commands automatically at
specified intervals.

## 1. Managing Crontabs
* `crontab -l` - List current user's cron jobs.
* `crontab -e` - Edit current user's cron jobs.
* `sudo crontab -l -u <user>` - List cron jobs for a specific user.
* `cat /etc/crontab` - View system-wide cron jobs.
* `ls /etc/cron.d/` - List application-specific cron job files.

## 2. Cron Schedule Syntax
```bash
* * * * * /path/to/command
│ │ │ │ │
│ │ │ │ └── Day of week (0-7, Sunday = 0 or 7)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```

Common examples:
* `* * * * *` - Every minute.
* `*/5 * * * *` - Every 5 minutes.
* `0 * * * *` - Every hour at minute 0.
* `0 2 * * *` - Every day at 02:00.
* `0 2 * * 1` - Every Monday at 02:00.

## 3. Output & Logging
* By default cron emails output to the local user. To suppress: `* * * * * /script.sh > /dev/null 2>&1`
* **Caution:** This silences all errors. Prefer logging to a file: `* * * * * /script.sh >> /var/log/myjob.log 2>&1`

## 4. Debugging Silent Failures
```bash
# Check cron service is running
sudo systemctl status cron

# Check system logs for cron execution
grep CRON /var/log/syslog | tail -20

# Test script manually as the cron user
sudo -u <user> /path/to/script.sh
```

## 5. Lock Files & flock
Manual lock files can cause silent failures if a script crashes without cleaning up. Use `flock` for safer locking:
```bash
# Run script with automatic lock - released even if script crashes
* * * * * flock -n /tmp/myjob.lock /path/to/script.sh
```