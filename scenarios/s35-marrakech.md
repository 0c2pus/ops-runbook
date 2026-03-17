# Scenario: Marrakech - Word Histogram

## 🚩 Issue

Find the second most frequent word in `frankestein.txt` and save it in uppercase to `~/mysolution`.

**Rules:** words are separated by spaces, newlines, or `.,:;` - case insensitive, apostrophes count as word characters.

## 🔍 Investigation & Resolution

### Step 1: Build the word frequency pipeline
```bash
cat frankestein.txt | tr 'A-Z' 'a-z' | tr ',.;:' ' ' | tr ' ' '\n' | sort | uniq -c | sort -nr | head -n 5
```

_Observation: Top results (empty lines appear first due to blank lines in the file, then):_
```bash
3904 the
2781 and
2540 of
2529 i
```

_`the` is most frequent, `and` is second._

### Step 2: Save result in uppercase
```bash
echo "AND" > ~/mysolution
```

## 💡 Lessons Learned

- Word frequency pipeline pattern:
  `tr` (normalize case) → `tr` (replace punctuation with spaces) → `tr` (spaces to newlines) → `sort` → `uniq -c` → `sort -nr`
- Empty lines in the output appear before real words when using `sort -nr` - filter them with `grep -v '^$'` before `uniq -c` for cleaner results.
- `tr 'A-Z' 'a-z'` converts to lowercase. `tr 'a-z' 'A-Z'` converts to uppercase.
- `uniq -c` counts consecutive duplicates - always `sort` before it.