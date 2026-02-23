# Linux System Diagnostics & Performance Tuning

Comprehensive guide for identifying bottlenecks in CPU, Memory, I/O, and Processes.

## 1. System Load & CPU Analysis
* `uptime` — Check load average (1, 5, 15 min). Compare with `nproc` (number of cores).
* `htop` — Interactive monitor. Press `F6` to sort by CPU/Memory. Look for `wa` (I/O Wait).
* `vmstat 1 5` — Check `r` (runnable) vs `b` (blocked). If `b` is high, the system is waiting for I/O.
* `mpstat -P ALL 1` — Check if a single CPU core is bottlenecked (single-threaded process issues).

## 2. Advanced Process Troubleshooting
* `ps aux --sort=-%mem | head` — Identify top memory-consuming processes.
* `lsof -p <PID>` — List all files and network sockets opened by a specific process.
* `strace -p <PID> -e trace=open,connect` — Trace specific system calls (e.g., why a process can't open a file or connect to a DB).
* `pstack <PID>` — Show the current stack trace of a running process (for frozen/hung apps).

## 3. Storage & Disk I/O Integrity
* `df -hT` — Disk usage with filesystem type (XFS, Ext4). Check for Read-Only filesystems.
* `df -i` — **Critical:** Check Inode usage. If 100%, you cannot create files even with free GBs.
* `iostat -xz 1` — Monitor disk utilization (`%util`). Values > 80% indicate I/O saturation.
* `lsblk -f` — View block devices, UUIDs, and mount points.
* `smartctl -a /dev/sdX` — Check physical disk health (S.M.A.R.T. errors).

## 4. System Limits & Stability
* `ulimit -a` — Check system-imposed limits (Max open files, max processes).
* `dmesg -T | tail -n 50` — Check kernel ring buffer for OOM Killer (Out of Memory) or Hardware errors.
* `journalctl -p 3 -xb` — View only high-priority errors from the current boot.

## 5. Memory Deep Dive
* `free -mw` — Detailed memory stats in MB (Wide mode shows buffers/cache separately).
* `cat /proc/meminfo` — Low-level memory stats (check for HugePages or Slab leaks).
* `slabtop -o` — Identify kernel-level memory consumption (Slab cache).