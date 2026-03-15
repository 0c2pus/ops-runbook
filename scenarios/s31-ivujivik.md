# Scenario: Ivujivik - Parlez-vous Français?

## 🚩 Issue

Find the Electoral District with the largest number of Rejected Ballots where Population is less than 100,000. The CSV file may be corrupted.

## 🔍 Investigation & Resolution

### Step 1: Inspect file for corruption
```bash
cat -A table_tableau11.csv | head -2
```

_Observation: `^M$` at end of every line - Windows line endings (`\r\n`) which will break awk field parsing._

### Step 2: Fix line endings
```bash
tr -d '\r' < table_tableau11.csv > new.csv
```

_Verify fix:_
```bash
cat -A new.csv | head -2
```

_Observation: Only `$` at line ends - Windows `^M` removed._

### Step 3: Find column indexes
```bash
head -n1 new.csv | tr ',' '\n' | nl
```

_Observation: Column 2 = District Name, Column 4 = Population, Column 9 = Rejected Ballots._

### Step 4: Filter and sort
```bash
awk -F"," '$4 < 100000 {print $2, $9, $4}' new.csv | sort -n -k 2
```

_Observation: Last row has the highest Rejected Ballots value._

### Step 5: Save the result
```bash
echo "Montcalm" > ~/mysolution
md5sum ~/mysolution
```

## 💡 Lessons Learned

- Always check for Windows line endings (`^M`) with `cat -A` before processing CSV files - they silently break awk and other tools.
- Never redirect `tr` output to the same input file - the file gets cleared before reading starts.
- `head -n1 file | tr ',' '\n' | nl` is the fastest way to find column indexes in a CSV before writing awk commands.
- In awk, conditions go before `{print}`: `awk '$4 < 100000 {print}'` filters rows before outputting them.