# Scenario: Minneapolis - CSV Data Manipulation & Automation

## 🚩 Issue
Need to break a large CSV file (`data.csv`, 306KB) into exactly 10 smaller files (approx. 30KB each). 
Requirements:
- Each file must include the original header.
- Naming convention: `data-00.csv`, `data-01.csv`, ..., `data-09.csv`.
- Files must be under 32KB.

## 🔍 Investigation
- File size: 306KB. Splitting into 10 parts results in ~30.6KB per file + header size, fitting the <32KB limit.
- Using standard `split` alone is insufficient as it doesn't replicate headers or follow the specific naming format.

## 🛠 Resolution

Executed the following Bash automation:

```bash
# 1. Extract and store the CSV header
HEADER=$(head -n 1 data.csv)

# 2. Create a temporary body file (excluding the header)
tail -n +2 data.csv > body.tmp

# 3. Split the body into 10 equal parts with numeric suffixes
split -n 10 -d -a 2 body.tmp part-

# 4. Reconstruct files with headers using a loop
for f in part-*; do 
    num=${f#part-}
    echo "$HEADER" > "data-$num.csv"
    cat "$f" >> "data-$num.csv"
    rm "$f" # Cleanup temporary chunks
done

# 5. Final cleanup
rm body.tmp
```

## ✅ Verification
Ran `/home/admin/agent/check.sh`.
**Result:** All 10 files created correctly with headers and valid naming.

## 💡 Lessons Learned

- **Bash Parameter Expansion:** Using `${var#prefix}` is an efficient way to clean up filenames inside loops.
- **Redirection Logic:** Using `>` to initialize a file with a header and `>>` to append data.
- **Split Command:** The `-n` flag is superior to `-b` when an exact number of output files is required.