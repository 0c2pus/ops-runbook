# Scenario: Rosario - Restore a MySQL Database

## 🚩 Issue

A MariaDB database `main` needs to be restored from `/home/admin/backup.sql` but the root password is unknown.

## 🔍 Investigation

### Step 1: Check MariaDB service and backup file
```bash
systemctl status mariadb
cat backup.sql
```

_Observation: Service is running but `Access denied for user 'root'`. The backup file uses `?` instead of `;` as SQL statement terminator - the file is corrupted._

### Step 2: Attempt direct restore
```bash
mysql -u root main < backup.sql
```

_Observation: `Access denied` - root password is unknown._

## ✅ Root Cause

Two separate issues:

1. Root password is unknown - need to reset it via `--skip-grant-tables`.
2. `backup.sql` uses `?` instead of `;` as statement terminator - must be fixed before importing.

## 🛠 Resolution

**Step 1:** Stop MariaDB service:
```bash
sudo systemctl stop mariadb
```

**Step 2:** Start in safe mode without authentication:
```bash
sudo mysqld_safe --skip-grant-tables --user=mysql &
```

Wait 3-5 seconds for socket to appear:
```bash
ls /run/mysqld/mysqld.sock
```

**Step 3:** Connect and reset root password:
```bash
mysql -u root
```

Inside MariaDB:
```sql
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY '';
exit
```

**Step 4:** Stop safe mode process and restart service:
```bash
sudo kill $(pgrep -f mysqld_safe)
sudo kill $(pgrep mariadbd)
sudo systemctl start mariadb
```

**Step 5:** Fix corrupted backup file:
```bash
sed 's/?/;/g' backup.sql > backup_fixed.sql
```

**Step 6:** Restore database:
```bash
mysql -u root main < backup_fixed.sql
```

**Step 7:** Verify:
```bash
mysql -u root main -e "SHOW TABLES;"
```

_Returns table `solution`._

## 💡 Lessons Learned

- `--skip-grant-tables` starts MariaDB without authentication - allows root access without password. Always stop the normal service first to avoid file conflicts between two processes.
- After `--skip-grant-tables`, run `FLUSH PRIVILEGES` first to activate the grant system, then `ALTER USER` to set the password.
- SQL dump files can be corrupted - always inspect with `cat` or `head` before importing. A `?` instead of `;` as terminator will cause `ERROR 1064` syntax errors.
- Check what credentials the test script uses before setting passwords - if the test expects passwordless root access, set an empty password.
- `mysql -e "SQL COMMAND"` executes SQL from the shell without entering interactive mode.