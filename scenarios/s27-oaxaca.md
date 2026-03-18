# Scenario: Oaxaca - Close an Open File

## 🚩 Issue

The file `/home/admin/somefile` is open for writing by a process. The file must be closed without killing the process.

## 🔍 Investigation

### Step 1: Find which process holds the file open
```bash
lsof /home/admin/somefile
```

_Observation: bash process with a specific PID holds the file open on file descriptor `77`._

### Step 2: Identify if it's the current session
```bash
echo $$
```

_Observation: Current bash PID matches the PID from lsof - the file was opened by the current shell session via `exec 66> /home/admin/somefile` in a startup script._

## ✅ Root Cause

The current bash session opened `/home/admin/somefile` and assigned it file descriptor `77`. Since it's the same process, the descriptor can be closed directly from within the current shell without elevated privileges.

## 🛠 Resolution

Close the file descriptor directly in the current bash session:
```bash
exec 77>&-
```

Verify:
```bash
lsof /home/admin/somefile
```

_Returns nothing - file descriptor is closed._

## 💡 Lessons Learned

- `lsof <file>` shows which process and which file descriptor number holds a file open.
- `exec N>&-` closes file descriptor `N` in the current bash session. `&` refers to a descriptor, `-` means close.
- If the process holding the file is the current shell - no elevated privileges or external tools are needed.
- If the process were a different PID, `gdb` would be needed to close the descriptor remotely - but it requires installation and sudo access.