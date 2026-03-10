# Scenario: "Bata" - Finding Secrets in the Virtual Filesystem

## 🚩 Issue
A password left by a "spy" needs to be retrieved from the `/proc/sys` directory. 
The file's content starts with the prefix `secret:`.
The goal is to extract the password and save it to `/home/admin/secret.txt`.

## 🔍 Investigation
- The `/proc` directory is a virtual filesystem containing kernel and system information.
- Many files in `/proc/sys` are restricted. Searching as a non-root user (`admin`) results in numerous `Permission denied` errors.
- Found the target file: `/proc/sys/kernel/core_pattern` contains the string `secret:excalibur`.

## 🛠 Resolution

### Step 1: Locating the File
To find the file while ignoring access errors, I used `grep` with error redirection:
```bash
grep -r "secret:" /proc/sys 2>/dev/null
```
_Result:_ `/proc/sys/kernel/core_pattern:secret:excalibur`

### Step 2: Extracting the Password (Automated methods)
Instead of manual copy-pasting, the password can be extracted using text-processing tools:

**Option A: Using `cut`**
```bash
grep -rh "secret:" /proc/sys 2>/dev/null | cut -d':' -f2 > /home/admin/secret.txt
```

- `-d':'`: Sets the delimiter to a colon.
- `-f2`: Selects the second field (the password).

**Option B: Using `awk`**
```bash
grep -rh "secret:" /proc/sys 2>/dev/null | awk -F':' '{print $2}' > /home/admin/secret.txt
```

- `-F':'`: Defines the field separator as a colon.    
- `{print $2}`: Outputs the content after the first colon.

**Option C: Using Bash Parameter Expansion (Fastest)**
```bash
LINE=$(grep -rh "secret:" /proc/sys 2>/dev/null)
echo "${LINE#secret:}" > /home/admin/secret.txt
```

## ✅ Verification
- Checked the MD5 hash of the result: `md5sum /home/admin/secret.txt`
- Expected: `a7fcfd21da428dd7d4c5bb4c2e2207c4`
- Result: **Match!**

## 💡 Lessons Learned
- **Virtual Filesystems:** Files in `/proc` aren't real files on disk but interfaces to the kernel.
- **Error Handling:** Redirecting `stderr` to `/dev/null` (`2>/dev/null`) is crucial when searching through system directories without root privileges.
- **String Cleaning:** Using `cut` or `awk` with specific delimiters allows for clean data extraction from structured logs or config files.