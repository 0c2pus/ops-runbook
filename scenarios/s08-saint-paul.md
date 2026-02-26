# Scenario: Saint Paul - Merging Multiple CSV Files

## 🚩 Issue
Need to merge 338 CSV files named `polldayregistrations_enregistjourduscrutin?????.csv` into a single `all.csv` file. 
Requirements:
- Only one header line at the top.
- Maintain all data from all files.
- Files can be in any order.

## 🔍 Investigation
- Each of the 338 files contains a header and data rows.
- Directly concatenating files (`cat *.csv > all.csv`) would result in 338 headers inside the final file, which is incorrect.
- Creating the output file in the same directory using a wildcard loop (`for f in *.csv`) can cause a recursion trap where `all.csv` appends to itself.

## 🛠 Resolution

### Optimized Approach (Stream Processing)
Instead of a slow loop, used `tail` with the "quiet" flag to process all files in a single stream:

```bash
# 1. Extract header from the first file
head -n 1 polldayregistrations_enregistjourduscrutin00001.csv > all.csv

# 2. Append data from all files, skipping the first line of each
# -q: Quiet mode (suppresses file headers in output)
# -n +2: Starts output from the second line
tail -q -n +2 polldayregistrations_enregistjourduscrutin*.csv >> all.csv
```

### Alternative Approach (Safe Loop)
If a loop is necessary, use a temporary extension to avoid recursion:
```bash
for f in pollday*.csv; do
    tail -n +2 "$f" >> all.tmp
done
mv all.tmp all.csv
```

## ✅ Verification
- Checked line count: `wc -l all.csv`
- Verified header: `head -n 1 all.csv`
- Ran `check.sh` provided by the environment.
    
## 💡 Lessons Learned
- **Recursion Traps:** Always be careful when creating an output file in the same directory where you are using wildcards (`*`).
- **Efficient Streaming:** `tail -q` is much faster for merging large numbers of files than a `for` loop, as it reduces the number of process spawns.