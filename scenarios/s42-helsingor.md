# Scenario: Helsingør - The First Walls of Postgres Physical Replication

## 🚩 Issue
Replica container `postgres-db-replica` constantly restarting with exit code 1. Master container healthy and running on port 5432.

## 🔍 Investigation

### Step 1: Check container status
```bash
docker ps
docker inspect postgres-db-replica
```

### Step 2: Check replica logs
```bash
docker logs postgres-db-replica
```
_Note: Logs revealed a series of FATAL errors - replica config values lower than master._

### Step 3: Compare config files
```bash
diff postgres/replica/postgres.conf postgres/master/postgres.conf
```
_Note: Found all mismatches at once instead of fixing errors one by one._

## ❌ What Didn't Work
Fixing config errors one by one after each restart - each cycle revealed only the next mismatch.

## ✅ Root Cause
Replica `postgres.conf` had multiple parameters set lower than master. PostgreSQL physical replication requires replica params to be >= master params and enforces this on startup. Additionally `primary_slot_name = 'replicator'` had wrong slot name - must match `-S replicator_slot` flag in `pg_basebackup`, or be removed since `-R` flag handles connection settings automatically.

## 🛠 Resolution
In `postgres/replica/postgres.conf` set:
```bash
max_connections = 100
max_worker_processes = 8
max_wal_senders = 10
#max_locks_per_transaction = 64
#primary_slot_name = 'replicator'
#hot_standby = on
```
Restart containers:
```bash
docker compose up -d --force-recreate
```

## 💡 Lessons Learned
- In physical replication replica params must be >= master params.
- Use `diff` to compare configs instead of chasing errors one by one from logs.
- `pg_basebackup -R` auto-writes connection settings - manual `primary_slot_name` is redundant.
- Slot name in config must exactly match `-S` flag in `pg_basebackup` if set manually.