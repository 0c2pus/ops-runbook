# Scenario: Unimak Island - Fun with Mr Jason

## 🚩 Issue

Find a specific station in `station_information.json` where `has_kiosk` is `false` and `capacity` is greater than `30`. Save the `station_id` to `~/mysolution`.

## 🔍 Investigation & Resolution

### Step 1: Inspect JSON structure
```bash
jq 'keys' station_information.json
```

_Observation: Top-level keys are `data`, `last_updated`, `ttl`, `version`._

### Step 2: Inspect a single station object
```bash
jq '.data.stations[0]' station_information.json
```

_Observation: Each station has `station_id`, `has_kiosk` (boolean), and `capacity` (integer) fields._

### Step 3: Filter by conditions
```bash
jq '.data.stations[] | select(.has_kiosk == false and .capacity > 30)' station_information.json
```

_Observation: One station matches. `station_id`: `05c5e17c-7aa9-49b7-9da3-9db4858ec1fc`_

### Step 4: Save the result
```bash
echo "05c5e17c-7aa9-49b7-9da3-9db4858ec1fc" > ~/mysolution
md5sum ~/mysolution
```

## 💡 Lessons Learned

- Always inspect JSON structure with `jq 'keys'` before filtering - it reveals top-level keys without printing the entire file.
- Boolean values in jq must be compared without quotes: `== false` not `== "false"`.
- Typos in field names cause compile errors in jq - double-check field names against the actual JSON structure.