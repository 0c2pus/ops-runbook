# Bash Scripting & Automation

Essential patterns for automating repetitive tasks and manipulating file structures.

## 1. Variables & Parameter Expansion
Useful for renaming files or extracting parts of strings without external tools.
* `VAR="part-01.tmp"`
* `${VAR#part-}` - **Remove prefix:** results in `01.tmp`.
* `${VAR%*.tmp}` - **Remove suffix:** results in `part-01`.
* `${VAR/old/new}` - **Replace first match:** find "old" and replace with "new".
* `${VAR//old/new}` - **Replace all matches.**

## 2. Advanced File Splitting
* `split -n 10 body.tmp part-` - Split a file into exactly 10 chunks.
* `split -d -a 2` - Use numeric suffixes (`00, 01...`) with a fixed length of 2 digits.

## 3. Control Structures (Loops)
Automate actions over multiple files or items.

### For Loop (One-liner)
`for f in *.txt; do echo "Processing $f"; cat header.txt "$f" > "new_$f"; done`

### For Loop (Script block)
```bash
for f in part-*; do
    num=${f#part-}
    echo "Header" > "data-$num.csv"
    cat "$f" >> "data-$num.csv"
done
```

## 4. Input/Output Redirection

- `>` - **Overwrite:** Create or replace a file with new content.
- `>>` - **Append:** Add content to the end of an existing file (crucial for rebuilding files).
- `|` - **Pipe:** Pass the output of one command as input to another.

## 5. Sequence & Arithmetic

- `seq 1 10` - Generate numbers from 1 to 10.
- `$(command)` - **Command substitution:** Save the output of a command into a variable (e.g., `HEADER=$(head -n 1 file.csv)`).
