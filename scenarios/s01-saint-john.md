# Scenario: Saint John - Terminate a Rogue Logging Process

## 🚩 Issue
A developer created a testing program that is continuously writing to `/var/log/bad.log` and filling up the disk. The task is to identify and terminate this process without deleting the log file.

## 🔍 Investigation

### Step 1: Confirm the issue
Check if the log file is indeed growing in real-time:
```bash
tail -f /var/log/bad.log
```

### Step 2: Identify the process
Identify which process has the file open for writing:
```bash
sudo lsof /var/log/bad.log
```
_Note: This command shows the PID (Process ID) of the application using the file.

### Step 3: Verify the process_
Check the details of the identified PID to ensure it's the correct target:
```bash
ps -fp <PID>
```

### ✅ Root Cause
A rogue background process was identified as the source of continuous writes to the log file. The process was no longer needed for testing but remained active.

### 🛠 Resolution
1. Found the PID using lsof.
2. Terminated the process using:
    `sudo kill <PID>`
3. Confirmed the file size stopped growing by running ls -lh /var/log/bad.log twice with a delay.

### 💡 Lessons Learned
* lsof (List Open Files) is the essential tool for linking files to processes.
* Do not delete files that are being written to; always stop the process first to avoid disk descriptor issues.