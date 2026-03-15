# jq - JSON Processing in the CLI

Essential tool for parsing, filtering, and extracting data from JSON files and API responses directly in the terminal.

## 1. Basic Output
* `jq 'keys' file.json` - List all top-level keys of a JSON object. Essential first step when exploring an unknown JSON structure.
* `jq '.' file.json` - Pretty-print the entire JSON file with formatting.
* `jq '.[0]' file.json` - Access the first element of a top-level array.
* `jq '.[]' file.json` - Iterate over all elements of a top-level array.
* `jq '.fieldName' file.json` - Extract a specific field from a top-level object.

## 2. Nested Access
* `jq '.Volumes[]' file.json` - Iterate over all elements of a nested array.
* `jq '.Volumes[].VolumeId' file.json` - Extract a specific field from each element of a nested array.
* `jq '.Volumes[].Attachments[]' file.json` - Access a nested array inside each element.

## 3. Filtering with select()
`select()` keeps only elements where the condition is `true`:
```bash
# Single condition
jq '.Volumes[] | select(.VolumeType == "gp3")' file.json

# Multiple conditions
jq '.Volumes[] | select(.VolumeType == "gp3" and .Size < 64 and .Iops < 1500 and .Throughput > 300)' file.json
```

**Comparison operators:** `==`, `!=`, `<`, `>`, `<=`, `>=`

**Note:** Date strings in ISO 8601 format (`2025-09-29T...`) are lexicographically sortable — string comparison works correctly for dates.

## 4. Pipes
jq supports pipes just like Linux shell:
```bash
# Extract field after filtering
jq '.Volumes[] | select(.VolumeType == "gp3") | .VolumeId' file.json

# Combine with shell pipes
cat file.json | jq '.Volumes[] | .VolumeId'
```

## 5. Common Patterns
```bash
# Count elements in an array
jq '.Volumes | length' file.json

# Check if array is empty
jq '.Volumes[] | select(.Attachments == [])' file.json

# Extract nested field from filtered results
jq '.Volumes[] | select(.Size < 64) | .Attachments[].InstanceId' file.json
```