# SQL Reference for Support Engineers (MySQL/PostgreSQL)

Tools and queries for data extraction, performance checking, and basic DB maintenance.

## 1. Data Extraction (DQL)
* `SELECT * FROM table WHERE column = 'value' LIMIT 10;` - Basic data retrieval.
* `SELECT count(*), status FROM orders GROUP BY status;` - Aggregate data to find anomalies (e.g., too many 'Failed' orders).
* `SELECT * FROM logs ORDER BY created_at DESC LIMIT 50;` - Get the most recent entries.

## 2. Performance & Locks
* `SHOW PROCESSLIST;` - (MySQL) See all active queries and their duration.
* `SELECT * FROM pg_stat_activity;` - (PostgreSQL) Monitor active sessions and queries.
* `EXPLAIN ANALYZE <query>;` - Analyze the execution plan of a slow query to find missing indexes.

## 3. Database Schema & Health
* `DESCRIBE table_name;` - (MySQL) Check column types and constraints.
* `\d table_name` - (PostgreSQL) View table schema.
* `SHOW TABLE STATUS LIKE 'table_name';` - Check table size and index length.

## 4. Connectivity & CLI
* `mysql -u user -p -h host database` - Connect to a remote MySQL instance.
* `psql -U user -d database -h host` - Connect to a remote PostgreSQL instance.

## 5. MariaDB/MySQL Administration
* `mysql -u root` - Connect as root without password.
* `mysql -u root -p` - Connect as root with password prompt.
* `mysql -u root -p<password> <database>` - Connect to specific database with password (no space between -p and password).
* `mysql -u root <database> < backup.sql` - Restore database from SQL dump file.
* `mysql -u root -e "SHOW TABLES;"` - Execute SQL command from shell without entering interactive mode.

**Inside MariaDB:**
* `SHOW DATABASES;` - List all databases.
* `USE <database>;` - Switch to a specific database.
* `SHOW TABLES;` - List all tables in current database.
* `FLUSH PRIVILEGES;` - Reload grant tables - required after manual user table changes.
* `ALTER USER 'root'@'localhost' IDENTIFIED BY 'newpassword';` - Change user password.
* `ALTER USER 'root'@'localhost' IDENTIFIED BY '';` - Remove password (set empty).

**Reset root password (when password is unknown):**
1. `sudo systemctl stop mariadb`
2. `sudo mysqld_safe --skip-grant-tables --user=mysql &`
3. `mysql -u root` → `FLUSH PRIVILEGES;` → `ALTER USER ...;` → `exit`
4. `sudo kill $(pgrep -f mysqld_safe)` → `sudo systemctl start mariadb`