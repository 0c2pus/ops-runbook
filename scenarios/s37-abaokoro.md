# Scenario: Abaokoro - Restore MySQL Databases Spooked by a Ghost

## 🚩 Issue

Three databases `first`, `second`, `third` need to be created and restored from `/home/admin/dbs_to_restore.zip`. Root password is unknown and disk keeps filling up.

## 🔍 Investigation

### Step 1: Check disk space and MariaDB access
```bash
df -h
mysql -u root -e "SHOW DATABASES;"
```

_Observation: Disk is 98% full. Root access denied - password unknown._

### Step 2: Find what fills the disk
```bash
du -h /var/log 2>/dev/null | sort -rh | head -10
```

_Observation: `/var/log/custom` contains multi-GB log files belonging to `admin`._

### Step 3: Find the source of disk filling
```bash
systemctl status generate-files.timer
```

_Observation: A hidden timer `generate-files.timer` runs a script every second that writes large files to `/var/log/custom/` until quota is exceeded._

### Step 4: Check user quota
```bash
quota -s
```

_Observation: User `admin` has 5120M quota and it is already at the limit._

## ✅ Root Cause

Three separate issues:

1. A malicious `generate-files.timer` continuously fills `/var/log/custom/` making it impossible to extract files.
2. Root MariaDB password is unknown.
3. Backup is a `.gz and .tar` archive that needs extraction before import.

## 🛠 Resolution

**Step 1:** Stop and disable the rogue timer:
```bash
sudo systemctl stop generate-files.timer
sudo systemctl disable generate-files.timer
```

**Step 2:** Clear log files to free quota:
```bash
rm -rf /var/log/custom/*
```

**Step 3:** Reset MariaDB root password:
```bash
sudo systemctl stop mariadb.service
sudo mysqld_safe --skip-grant-tables --user=mysql &
# Wait 3-5 seconds
mysql -u root
```

Inside MariaDB:
```sql
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY '';
exit
```

Kill safe mode and restart:
```bash
sudo kill $(pgrep -f mysqld_safe)
sudo kill $(pgrep mariadbd)
sudo systemctl start mariadb
```

**Step 4:** Extract backup archive:
```bash
gunzip dbs_to_restore.tar.gz
tar -xvf dbs_to_restore.tar
```

**Step 5:** Create databases and restore:
```bash
mysql -u root -e "CREATE DATABASE first; CREATE DATABASE second; CREATE DATABASE third;"
mysql -u root first < first.sql
mysql -u root second < second.sql
mysql -u root third < third.sql
```

**Step 6:** Verify:
```bash
mysql -u root -e "SHOW DATABASES;"
mysql -u root first -e "SHOW TABLES;"
```

## 💡 Lessons Learned

- Disk quota and disk space are different - even with free disk space, a user can be blocked from writing if their quota is exhausted. Use `quota -s` to check user quota.
- Hidden timers or cron jobs can continuously fill disk space. Always check `systemctl list-timers` and running processes when disk fills unexpectedly.
- `.tar.gz` requires two steps: `gunzip` first, then `tar -xvf`. Or use `tar -xzvf` to do both in one command.