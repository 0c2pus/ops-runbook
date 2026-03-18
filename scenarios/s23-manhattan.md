# Scenario: Manhattan - Can't Write Data into Database

## 🚩 Issue

Inserting a row into a PostgreSQL database fails with a socket connection error. The `postgresql.service` appears active but no actual postgres process is running.

## 🔍 Investigation

### Step 1: Confirm the failure
```bash
sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt
```

_Observation: Connection fails - socket file not found. Postgres is not running._

### Step 2: Check the service status
```bash
systemctl status postgresql
```

_Observation: `active (exited)` - suspicious. `ExecStart=/bin/true` means this unit just returns success immediately without starting any real process._

### Step 3: Find the real postgres unit
```bash
systemctl list-units | grep postgres
```

_Observation: `postgresql@14-main.service` has status `failed` - this is the unit that actually manages the postgres process._

### Step 4: Check why the unit failed
```bash
journalctl -u postgresql@14-main.service
```

_Observation: `pg_ctl start -D /opt/pgdata/main` failed. Error mentions inability to open PID file - postgres cannot write to its data directory._

### Step 5: Check disk space
```bash
df -h
```

_Observation: `/dev/nvme0n1` mounted at `/opt/pgdata` is 100% full - only 28K remaining. Postgres cannot start because it cannot write anything to disk._

### Step 6: Inspect the data directory
```bash
ls -lah /opt/pgdata/
```

_Observation: Three large backup files (`file1.bk` 7.0G, `file2.bk` 923M, `file3.bk` 488K) and a file named `deleteme` are occupying the entire disk._

## ❌ What Didn't Work

- Checking `postgresql.service` status gave a false positive (`active`) - the real unit `postgresql@14-main.service` was hidden and failed.

## ✅ Root Cause

The disk volume mounted at `/opt/pgdata` was filled to 100% by leftover backup files. PostgreSQL cannot start when its data directory has no free space - it cannot write PID files, WAL logs, or any data.

## 🛠 Resolution

**Step 1:** Remove the files filling the disk:
```bash
sudo rm /opt/pgdata/file1.bk /opt/pgdata/file2.bk /opt/pgdata/file3.bk /opt/pgdata/deleteme
```

**Step 2:** Verify free space is restored:
```bash
df -h /opt/pgdata
```

**Step 3:** Restart the postgres unit:
```bash
sudo systemctl restart postgresql@14-main.service
```

**Step 4:** Verify the fix:
```bash
sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt
```

_Returns `INSERT 0 1` - success._

## 💡 Lessons Learned

- `active (exited)` in systemd does not mean the service is healthy - always check if the actual process exists with `ps aux | grep <name>`.
- A parent unit (e.g. `postgresql.service`) can mask a failed child unit (e.g. `postgresql@14-main.service`) - always check `list-units` for the full picture.
- A full disk is a silent killer - postgres fails to start with cryptic errors that don't obviously point to disk space. Always check `df -h` early.
- In production, never delete backup files without authorization. In this case the files were explicitly named `deleteme` and `.bk` as part of the exercise.