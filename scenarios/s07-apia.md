# Scenario: Apia - Needle in a Haystack
## 🚩 Issue

In `/home/admin/data` there are multiple files with identical content except one - a single word was added to it. The task is to identify that word and save it to `/home/admin/solution`.

## 🔍 Investigation

### Step 1: Find the unique file by size
```bash
ls -lS /home/admin/data/
```

_Observation: All files are the same size except `file76.txt` which is slightly larger - it contains an extra word._

### Step 2: Compare the unique file with another file
```bash
diff -u file76.txt file77.txt
```

_Observation: One line differs between the two files. Visual inspection of the diff output reveals the added word._

## ❌ What Didn't Work

- `diff --word-diff` - not available in this version of diff.
- Automated extraction of the specific word from diff output would require a complex pipeline - visual inspection was used instead.

## ✅ Root Cause

One file in the directory had an extra word inserted into its content, making it slightly larger than all other files.

## 🛠 Resolution

**Step 1:** Identify the unique file by size:
```bash
ls -lS /home/admin/data/
```

**Step 2:** Compare with any other file using diff:
```bash
diff -u /home/admin/data/file76.txt /home/admin/data/file77.txt
```

**Step 3:** Save the identified word to the solution file:
```bash
echo "eureka" > /home/admin/solution
```

**Step 4:** Verify:
```bash
md5sum /home/admin/solution
```

## 💡 Lessons Learned

- When looking for a unique file among many, file size is the fastest first filter - `ls -lS` sorts by size and immediately highlights outliers.
- `diff -u` shows the differing lines with context. For small files visual inspection is practical, but for large-scale automation a pipeline with `grep`, `awk`, or `comm` would be needed.
- `comm` is an alternative to diff for comparing sorted files line by line and is sometimes easier to pipe into further processing.