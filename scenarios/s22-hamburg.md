# Scenario: Hamburg - Find the AWS EC2 Volume

## 🚩 Issue

A JSON file `aws-volumes.json` contains descriptions of many AWS EBS volumes. One specific volume must be identified by its characteristics and its `InstanceId` saved to `~/mysolution`.

**Search criteria:**
- `VolumeType`: `gp3`
- `CreateTime`: before `2025-09-30`
- `Size`: less than `64`
- `Iops`: less than `1500`
- `Throughput`: greater than `300`

## 🔍 Investigation & Resolution

### Step 1: Inspect the JSON structure
```bash
head -n 30 aws-volumes.json
```

_Observation: File contains a `Volumes` array. Each volume has fields: `VolumeType`, `CreateTime`, `Size`, `Iops`, `Throughput`, `Attachments`._

### Step 2: Filter volumes using jq
```bash
jq '.Volumes[] | select(.CreateTime < "2025-09-30" and .Size < 64 and .Iops < 1500 and .Throughput > 300 and .VolumeType == "gp3")' aws-volumes.json
```

_Observation: Two volumes match. One has `"Attachments": []` - not attached to any instance. The other has a populated `Attachments` array with an `InstanceId`. Since the task asks for `InstanceId`, only the attached volume is valid._

### Step 3: Save the result
```bash
echo "i-371822c092b2470da" > ~/mysolution
md5sum ~/mysolution
```

## 💡 Lessons Learned

- `jq` is the standard tool for parsing JSON in Linux - essential for working with AWS CLI, Kubernetes, and API responses.
- String date comparison works in jq because ISO 8601 format (`2025-09-29T...`) is lexicographically sortable.
- Always read the full requirements before filtering - "find the InstanceId" implied the volume must be attached to an instance, which eliminated one of two matching results.
- An empty `Attachments: []` means the volume exists but is not mounted to any instance.