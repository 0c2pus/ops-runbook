# Scenario: Saskatoon - Counting IPs from Web Server Logs

## 🚩 Issue
A web server access log file is located at `/home/admin/access.log`. Each line represents an HTTP request starting with the requester's IP address. The goal is to identify the IP address with the highest number of requests and save it to `/home/admin/highestip.txt`.

## 🔍 Investigation

### Step 1: Analyze the log format
View the first few lines of the log to confirm the IP is in the first column:
```bash
head -n 5 /home/admin/access.log
```

## Step 2: Extract, Count, and Sort
To find the most frequent IP, we need to:
1. Extract the first column.
2. Sort the IPs so identical ones are adjacent.
3. Count occurrences.
4. Sort numerically in descending order.
   
Execution:
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -n 1
```

Command Breakdown:
* `awk '{print $1}'`: Extracts the first field (IP address) from each line.
* `sort`: Sorts the IPs alphabetically (required for `uniq`).
* `uniq -c`: Filters unique lines and prefixes them with the number of occurrences.
* `sort -nr`: Sorts the result **N**umerically in **R**everse order (highest count first).
* `head -n 1`: Takes the top result.

_Result: The IP `66.249.73.135` appeared 482 times._

## ✅ Root Cause
Analytical task to identify the most active client/bot from access logs.

## 🛠 Resolution
1. Executed the pipeline to find the top IP.
2. Wrote the identified IP to the required file:
```bash
echo "66.249.73.135" > /home/admin/highestip.txt
```
3. Verified the solution with the provided SHA1 checksum:
```bash
sha1sum /home/admin/highestip.txt
```

## 💡 Lessons Learned
* **Text Processing Power:** Combining small Linux utilities (`awk`, `sort`, `uniq`) allows for powerful data analysis without complex scripts.
* **The Importance of** `sort` **before** `uniq`: `uniq` only detects duplicate lines that are adjacent, so sorting beforehand is mandatory.
* **Log Analysis:** This method is a standard way to detect potential DoS attacks or aggressive web scrapers in a real-world L2 support environment.