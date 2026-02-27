# Linux Server Review & Exploration Guide

A practical guide for identifying the purpose, resource utilization, and application architecture of an unknown server.

## 1. Finding the Server Purpose (What is it doing?)
To understand why a server exists, you must identify which services are "exposed" (listening on ports) and what processes are serving them.
* `ss -tlpn` — View open ports and the specific processes serving them. Identify if services are bound to `0.0.0.0` (publicly accessible) or `127.0.0.1` (locally restricted).
* `ps auxf` — Inspect the full process tree to see how applications were started and identify parent/child relationships between services.
* `systemctl list-units --type=service --state=running` — List all active background services managed by the operating system.
* `docker ps -a` — Check if the server's primary applications are running inside isolated containers.



## 2. Checking Hardware Utilization (Is it struggling?)
To determine if the server is saturated or failing, check the three main pillars of hardware: CPU, RAM, and Disk.
* **CPU & RAM Saturation:** `top` or `htop` — Monitor which processes consume the most resources. Compare the "Load Average" against the total number of CPU cores.
* **Memory Errors (OOM):** `dmesg | grep -i "out of memory"` or `grep -i "oom" /var/log/syslog` — Verify if the Linux kernel has recently killed processes due to lack of available RAM.
* **Storage Capacity:** `df -h` — Check if any mounted partitions are at 100% capacity, which prevents services from writing logs or data.
* **Disk I/O Bottlenecks:** `iotop` or `iostat -xz 1` — Identify if a specific application (like a database) is overwhelming the disk read/write throughput.
* **Network Health:** `ip -s link` — Look for "errors" or "dropped" packets on active network interfaces that indicate physical or configuration issues.



## 3. Tracing Application Logic (How is it connected?)
Once you find the open ports, trace how data flows between the discovered services to map the architecture.
* **Web Entry Points:** `curl -I http://localhost:<port>` — Inspect HTTP response headers to identify the type of web server (e.g., Nginx vs. HAProxy).
* **Configuration Mapping:** `cat /etc/haproxy/haproxy.cfg` or `cat /etc/nginx/nginx.conf` — Review routing rules to see which "frontend" port directs traffic to which "backend" service.
* **Environment & Logic:** `cat .env` or `cat config.py` — Search the application directories for database connection strings, credentials, and dependency paths.
* **Database Discovery:** `psql -l` or `mysql -e "show databases;"` — List existing databases to understand the data storage backend used by the applications.

## 4. Troubleshooting via Logs (Why are there errors?)
Search for specific error signatures to diagnose why a service might be failing or performing poorly.
* `journalctl -p err` — Filter the system journal to show only high-priority error messages.
* `journalctl -u <service_name>` — View historical and real-time logs for a specific application unit.
* `tail -f /var/log/syslog` — Watch general system events in real-time to catch spontaneous failures.
* `docker logs <container_name>` — Retrieve logs from containerized applications that do not write to the standard system journal.

## 5. Summary of Architecture
After investigating, you should be able to visualize the full request path:
> **User Request** → `Port 8000 (Load Balancer)` → `Port 9000 (App Server)` → `Port 5432 (Database)`.