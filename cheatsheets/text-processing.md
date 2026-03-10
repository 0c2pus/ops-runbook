# Text Processing & Stream Editing (grep, sed, awk, cut)

This handbook contains essential tools for filtering, transforming, and analyzing text data (logs, configurations, CSVs) directly from the CLI.

## 1. Searching & Filtering (grep)
The primary tool for finding specific data patterns within files.
* `grep "pattern" file` — Display lines containing the pattern.
* `grep -i "pattern" file` — Case-insensitive search.
* `grep -v "pattern" file` — Invert match: show lines that do NOT contain the pattern.
* `grep -E "p1|p2" file` — Extended Regex: search for multiple patterns simultaneously.
* `grep -r "pattern" /path/` — Recursive search through all files in a directory.
* `grep -A 5 -B 2 "Error" file` — Context search: show 5 lines **After** and 2 lines **Before** the match (critical for Stack Traces).
* `grep -c "pattern" file` — Count the number of lines that match the pattern.

## 2. Stream Editing & Substitution (sed)
Used for modifying text "on the fly" or performing bulk replacements.
* `sed 's/old/new/g' file` — Replace all occurrences of "old" with "new" in the output.
* `sed -i 's/old/new/g' file` — **In-place edit:** Save changes directly to the file.
* `sed -n '5,10p' file` — Print only lines 5 through 10.
* `sed '/pattern/d' file` — Delete all lines matching the pattern from the output.

## 3. Data Extraction & Analytics (awk)
A powerful tool for processing columns and performing arithmetic operations.
* `awk '{print $1, $3}' file` — Print specific columns (default delimiter: space/tab).
* `awk -F":" '{print $1}' /etc/passwd` — Define a custom field separator (e.g., `:` for system files or `,` for CSV).
* `awk '/pattern/ {print $2}' file` — Find lines with a pattern and output only their 2nd column.
* `awk '{sum+=$1} END {print sum}' file` — Calculate the total sum of the 1st column.
* `awk '{sum+=$1} END {print sum/NR}' file` — Calculate the arithmetic mean (NR = Number of Records/lines).
* `awk 'length($0) > 100' file` — Output only lines longer than 100 characters.

## 4. Sorting, Uniques & Formatting (sort, uniq, cut, tr)
* `cut -d',' -f1,3 file.csv` — Extract specific fields (1st and 3rd) from a delimited file.
* `tr '[:lower:]' '[:upper:]'` — Convert text to uppercase.
* `sort -n` — Numerical sort.
* `sort -r` — Reverse sort.
* `sort -u` — Sort and remove duplicates.
* `sort | uniq -c | sort -nr` — **The Analytics Pipeline:** Count unique occurrences and sort them from highest to lowest.

## 5. File Inspection & Statistics
* `wc -l file` — Count the total number of lines.
* `head -n 20 file` — Display the first 20 lines.
* `tail -f /var/log/syslog` — Follow log updates in real-time.
* `tail -q -n +2 *.csv >> merged.csv` — Merge multiple files skipping headers. `-q` ensures file names aren't printed between chunks.
* `stat file` — Detailed file information (size, permissions, last modified).

## 6. File Comparison (diff)
* `diff file1 file2` - Compare two files line by line. Lines with `<` are from file1, `>` from file2.
* `diff -u file1 file2` - Unified format: shows context with `-` for removed and `+` for added lines.
* `diff -q file1 file2` - Quick check: reports only whether files differ, no details.
* `diff -rq dir1/ dir2/` - Recursively compare two directories and report which files differ.

**Finding a unique file by size before comparing:**
* `ls -lS /path/` - List files sorted by size (largest first). A file with different size is a starting point for investigation.