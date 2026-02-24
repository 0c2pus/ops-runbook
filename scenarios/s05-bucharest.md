# Scenario: Bucharest - Troubleshooting PostgreSQL Connectivity

## 🚩 Issue
A web application is unable to connect to the PostgreSQL 13 database (`app1`). 
Initial connection test fails with a `FATAL: pg_hba.conf rejects connection` error.

## 🔍 Investigation

### Step 1: Verification of the Failure
Executed the connection command provided in the requirements:
```bash
PGPASSWORD=app1user psql -h 127.0.0.1 -d app1 -U app1user -c '\q'
```
**Result:** `psql: error: FATAL: pg_hba.conf rejects connection for host "127.0.0.1", user "app1user", database "app1", SSL off`

### Step 2: Service and Port Status
Verified if the PostgreSQL service is active and listening on the default port (5432):
```bash
sudo netstat -plunt | grep postgres
```
**Result:** The service is listening on `127.0.0.1:5432`. The issue is not the service state, but the access policy.

### Step 3: Analyzing Authentication Configuration
Inspected the Host-Based Authentication (HBA) configuration file:
```bash
sudo cat /etc/postgresql/13/main/pg_hba.conf
```
**Observation:** At the beginning of the file, the following restrictive rules were found:
```bash
host    all             all             all                     reject
host    all             all             all                     reject
```
Since PostgreSQL processes this file from **top to bottom**, these `reject` rules were blocking all connections before reaching the allowed local host configurations.

## ✅ Root Cause
The `pg_hba.conf` file contained global `reject` rules at the top of the configuration, preventing any successful TCP/IP connections even for authorized users on localhost.

## 🛠 Resolution
### Step 1: Modify HBA Configuration
Edited the file to allow the specific user/database connection before the reject rules.
```bash
sudo nano /etc/postgresql/13/main/pg_hba.conf
```
**Action**: Commented out the `reject` lines or added a specific permit rule at the top:
```bash
# Allow app1user to app1 database from localhost
host    app1    app1user    127.0.0.1/32    md5
```

### Step 2: Restart Service
Applied changes by restarting the PostgreSQL service:
```bash
sudo systemctl restart postgresql
```

### Step 3: Final Verification
Re-ran the test command:
```bash
PGPASSWORD=app1user psql -h 127.0.0.1 -d app1 -U app1user -c '\q'
```
**Result:** Success (no error returned).

## 💡 Lessons Learned
* **Rule Precedence:** In `pg_hba.conf`, the first matching rule wins. Always place specific allow rules above global restrict/reject rules.
* **CIDR Notation:** Using /32` is essential for specifying a single host in PostgreSQL network configurations.
* **Authentication Methods:** Understanding the difference between `peer` (local socket) and `md5/scram-sha-256` (network) is key for DB troubleshooting.