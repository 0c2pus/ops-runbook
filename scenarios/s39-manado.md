# Scenario: Manado - How Much Do You Press?

## 🚩 Issue

The file `/home/admin/names` (35147 bytes) must be compressed to under 9400 bytes. The file can be modified without deleting any content.

## 🔍 Investigation

### Step 1: Try gzip on original file
```bash
gzip -k names
ls -la names.gz
```

_Observation: `names.gz` = 16207 bytes - too large._

### Step 2: Try xz on original file
```bash
xz -k names
ls -la names.xz
```

_Observation: `names.xz` = 15116 bytes - still too large._

### Step 3: Optimize file before compression

_Observation: The file contains names - sorting puts identical or similar strings together which helps compression algorithms find more repeating patterns._
```bash
sort names > new_names
```

### Step 4: Compress sorted file with xz
```bash
xz -k new_names
ls -la new_names.xz
```

_Observation: `new_names.xz` = 9328 bytes - under the 9400 limit._

## ✅ Root Cause

The file was not optimally ordered for compression. Sorting the content groups similar strings together, allowing `xz` to find more repeating patterns and achieve better compression ratio.

## 🛠 Resolution
```bash
sort names > new_names
xz -k new_names
cp new_names.xz ~/solution/names.xz
```

Verify size:
```bash
ls -lh ~/solution/names.xz
```

_Returns 9328 bytes - under the 9400 limit._

## 💡 Lessons Learned

- Compression ratios depend on file content order - sorting similar data together significantly improves compression.
- `xz` provides better compression than `gzip` at the cost of more CPU time. For maximum compression use `xz`, for speed use `gzip`.
- `-k` flag preserves the original file in both `gzip` and `xz`.
- "Modify without deleting" means reordering is allowed - sorting is not deletion.