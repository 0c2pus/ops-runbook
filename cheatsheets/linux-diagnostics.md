# Linux System Diagnostics & Performance Tuning

Comprehensive guide for identifying bottlenecks in CPU, Memory, I/O, and Processes.

## 1. System Load & CPU Analysis
* `uptime` - Check load average (1, 5, 15 min). Compare with `nproc` (number of cores).
* `htop` - Interactive monitor. Press `F6` to sort by CPU/Memory. Look for `wa` (I/O Wait).
* `vmstat 1 5` - Check `r` (runnable) vs `b` (blocked). If `b` is high, the system is waiting for I/O.
* `mpstat -P ALL 1` - Check if a single CPU core is bottlenecked (single-threaded process issues).

## 2. Advanced Process Troubleshooting
* `ps aux --sort=-%mem | head` - Identify top memory-consuming processes.
* `lsof -p <PID>` - List all files and network sockets opened by a specific process.
* `strace -p <PID> -e trace=open,connect` - Trace specific system calls (e.g., why a process can't open a file or connect to a DB).
* `pstack <PID>` - Show the current stack trace of a running process (for frozen/hung apps).

## 3. Storage & Disk I/O Integrity
* `df -hT` - Disk usage with filesystem type (XFS, Ext4). Check for Read-Only filesystems.
* `df -i` - **Critical:** Check Inode usage. If 100%, you cannot create files even with free GBs.
* `iostat -xz 1` - Monitor disk utilization (`%util`). Values > 80% indicate I/O saturation.
* `lsblk -f` - View block devices, UUIDs, and mount points.
* `smartctl -a /dev/sdX` - Check physical disk health (S.M.A.R.T. errors).
* `quota -s` - Check disk quota for current user. An asterisk `*` means quota is exceeded.
* `du -h /var/log 2>/dev/null | sort -rh | head -10` - Find largest log directories.
* `du -h /home/admin/* 2>/dev/null | sort -rh` - Check disk usage per file in home directory.

## 4. System Limits & Stability
* `ulimit -a` - Check system-imposed limits (Max open files, max processes).
* `dmesg -T | tail -n 50` - Check kernel ring buffer for OOM Killer (Out of Memory) or Hardware errors.
* `journalctl -p 3 -xb` - View only high-priority errors from the current boot.

## 5. Memory Deep Dive
* `free -mw` - Detailed memory stats in MB (Wide mode shows buffers/cache separately).
* `cat /proc/meminfo` - Low-level memory stats (check for HugePages or Slab leaks).
* `slabtop -o` - Identify kernel-level memory consumption (Slab cache).

## 6. File Descriptors
Every process has numbered file descriptors - open files, sockets, and pipes. Standard descriptors: `0` = stdin, `1` = stdout, `2` = stderr. Everything above is opened by the process itself.
* `lsof <file>` - Show which process and descriptor number holds a specific file open.
* `lsof -p <PID>` - List all open file descriptors for a specific process.
* `ls -la /proc/<PID>/fd/` - View all open descriptors for a process as symlinks to actual files.
* `exec N>&-` - Close file descriptor `N` in the current bash session.

**Note:** If a file is deleted but a process still holds it open, disk space is not freed until the descriptor is closed. Use `lsof | grep deleted` to find such cases.

## 7. Sparse Files & Disk Space
A sparse file has a large logical size but only consumes disk space for blocks that contain actual data. Blocks of zeros are not physically written to disk - the OS simply records their position.
* `truncate -s <size> <file>` - Create or resize a file to a specific size as a sparse file without allocating real disk blocks.
* `du -h <file>` - Show actual disk usage (how much space is physically allocated on disk).
* `du -h --apparent-size <file>` - Show apparent size (logical size as reported by the filesystem).

**Note:** For a sparse file these two values will differ significantly. To convert a regular file to sparse: clear its contents first with `> <file>`, then restore logical size with `truncate`.