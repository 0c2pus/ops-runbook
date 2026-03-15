# Scenario: Kihei - Surely Not Another Disk Space Scenario

## 🚩 Issue

`/home/admin/kihei` program panics on startup. Disk is 93% full. The `/home/admin/datafile` (5GB) cannot be deleted.

## 🔍 Investigation

### Step 1: Run the program and check disk space
```bash
/home/admin/kihei
df -hT
```

_Observation: Program panics with `exit status 1`. Disk is 93% full with only 576MB free._

### Step 2: Find what occupies disk space
```bash
sudo du -h /home/admin/* 2>/dev/null | sort -rh
```

_Observation: `datafile` is 5GB and is the only large file. It cannot be deleted per the scenario rules._

### Step 3: Read the test script to understand requirements
```bash
cat /home/admin/agent/check.sh
```

_Observation: Test verifies that `datafile` is exactly `5368709120` bytes. The file must exist with its original size but doesn't need real disk content._

## ✅ Root Cause

`datafile` occupies 5GB of real disk space leaving insufficient room for `kihei` to run. The solution is to convert it to a sparse file - it will report the correct size to the OS but not actually consume disk blocks.

## 🛠 Resolution

**Step 1:** Add write permission to datafile:
```bash
sudo chmod o+w /home/admin/datafile
```

**Step 2:** Clear the file content (this frees the actual disk blocks):
```bash
echo "" > /home/admin/datafile
```

**Step 3:** Extend to original size as a sparse file:
```bash
truncate -s 5368709120 /home/admin/datafile
```

**Step 4:** Verify disk space is freed and file size is correct:
```bash
df -hT
ls -la /home/admin/datafile
```

**Step 5:** Run the program:
```bash
/home/admin/kihei
```

_Returns `Done.`_

## 💡 Lessons Learned

- A sparse file reports a large logical size but only consumes disk space for blocks that contain actual data. Blocks of zeros are not written to disk.
- `truncate -s <size> <file>` sets the logical file size without allocating real disk blocks - creating a sparse file.
- To convert a regular file to sparse: clear its contents first (freeing real blocks), then use `truncate` to restore the logical size.
- Real-world use cases for sparse files: VM disk images, database files, and container layers - they pre-declare size without consuming space upfront.