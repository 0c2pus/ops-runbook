# Scenario: Lhasa - Arithmetic Mean Calculation

## 🚩 Issue
The task is to find the average score from the second column of `/home/admin/scores.txt`.
**Requirement:** The result must have exactly two decimal places, truncated (not rounded). 
Example: `21.349` must become `21.34`.

## 🔍 Investigation

### Step 1: Data Verification
Checked the structure of the file:
```bash
head -n 5 /home/admin/scores.txt
```
_Observation: The file has two space-separated columns. Target data is in the second column ($2)._

### Step 2: Tool Selection
Standard `printf "%.2f"` rounds the numbers (e.g., 84.389 becomes 84.39). Since the requirement is truncation, I will use `awk` with a mathematical shift:
1. Multiply by 100.
2. Get the integer part (`int`).
3. Divide by 100.

## ✅ Root Cause
Data processing task requiring specific formatting (truncation) that standard rounding utilities do not provide by default.

## 🛠 Resolution
Executed the following `awk` command to calculate the mean and format the output:
```bash
awk '{sum+=$2} END {avg=sum/NR; printf "%.2f\n", int(avg*100)/100}' /home/admin/scores.txt > ~/solution
```
### Command Breakdown:
* `sum+=$2`: Adds the value of the second column to the sum variable for each line.
* `NR`: Built-in variable representing the total number of lines processed.
* `int(avg*100)/100`: Truncates the number to two decimal places.
* `printf "%.2f"`: Ensures trailing zeros are present (e.g., `33.1` becomes `33.10`).

## 💡 Lessons Learned
* **AWK Efficiency**: `awk` is the most effective tool for field-based math in a Linux environment.
* **Truncation vs Rounding**: In technical support, "precision" often means following the exact rule (truncation) rather than standard mathematical rounding.