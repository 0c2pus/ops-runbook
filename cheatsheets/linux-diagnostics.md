# Linux System Diagnostics Checklist

This guide provides a structured approach to identifying system bottlenecks and service failures.

## 1. System Overview & Load
Check general system health and CPU/Load Average.
* `uptime` — Quick view of load average and boot time.
* `top` (or `htop`) — Real-time process monitoring and CPU/RAM usage.
* `vmstat 1` — Virtual memory statistics and CPU activity over time.

## 2. Memory Analysis
Identify RAM exhaustion or OOM (Out of Memory) issues.
* `free -h` — Display amount of free and used system memory in human-readable format.
* `cat /proc/meminfo` — Detailed memory statistics.
* `ps aux --sort=-%mem | head -n 10` — List top 10 memory-consuming processes.

## 3. Storage & Disk I/O
Check for full partitions or disk performance bottlenecks.
* `df -h` — Check disk space usage per partition.
* `du -sh /* 2>/dev/null` — Identify the largest directories in the root filesystem.
* `iostat -xz 1` — Detailed disk I/O statistics (check for high %util).
* `lsof +L1` — Find deleted files that are still consuming space (held by processes).

## 4. Networking & Ports
Identify connectivity issues and port bindings.
* `ss -tulpn` — List all listening ports and the associated processes.
* `ip addr` — Check interface configurations and IP addresses.
* `ping -c 4 8.8.8.8` — Verify basic external connectivity.
* `nc -zv <host> <port>` — Check if a specific port is open on a remote host.

## 5. Services & Logs
Investigate service failures and system errors.
* `systemctl list-units --failed` — List all services that failed to start.
* `systemctl status <service_name>` — Check the current state of a specific service.
* `journalctl -p 3 -xb` — Show high-priority errors from the current boot.
* `tail -f /var/log/syslog` (or `/var/log/messages`) — Monitor general system logs in real-time.