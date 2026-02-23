# SQL Reference for Support Engineers (MySQL/PostgreSQL)

Tools and queries for data extraction, performance checking, and basic DB maintenance.

## 1. Data Extraction (DQL)
* `SELECT * FROM table WHERE column = 'value' LIMIT 10;` — Basic data retrieval.
* `SELECT count(*), status FROM orders GROUP BY status;` — Aggregate data to find anomalies (e.g., too many 'Failed' orders).
* `SELECT * FROM logs ORDER BY created_at DESC LIMIT 50;` — Get the most recent entries.

## 2. Performance & Locks
* `SHOW PROCESSLIST;` — (MySQL) See all active queries and their duration.
* `SELECT * FROM pg_stat_activity;` — (PostgreSQL) Monitor active sessions and queries.
* `EXPLAIN ANALYZE <query>;` — Analyze the execution plan of a slow query to find missing indexes.

## 3. Database Schema & Health
* `DESCRIBE table_name;` — (MySQL) Check column types and constraints.
* `\d table_name` — (PostgreSQL) View table schema.
* `SHOW TABLE STATUS LIKE 'table_name';` — Check table size and index length.

## 4. Connectivity & CLI
* `mysql -u user -p -h host database` — Connect to a remote MySQL instance.
* `psql -U user -d database -h host` — Connect to a remote PostgreSQL instance.