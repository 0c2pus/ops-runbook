# Scenario: Taipei - Port Knocking Bypass

## 🚩 Issue
The HTTP server on port 80 is unreachable (`Connection refused`). Initial checks suggest the service is hidden behind a Port Knocking mechanism, which requires a trigger packet to open the firewall.

## 🔍 Investigation

### Step 1: Connectivity and Service Status
Confirmed that port 80 is closed:
```bash
curl localhost
# Result: curl: (7) Failed to connect to localhost port 80: Connection refused
```

## Step 2: System Reconnaissance
Instead of blind port scanning, I checked for active knocking daemons or configurations:
```bash
# Checking for the knockd process
ps aux | grep knockd
```
_Result: Confirmed that `knockd` is running._
   
## Step 3: Analyzing Knocking Logic
Examined the configuration file to find the required sequence:
```bash
cat /etc/knockd.conf
```
_Observation: The configuration specifies that a SYN packet to port 8080 triggers the opening of port 80._

## ✅ Root Cause
The web server was protected by a Port Knocking daemon. Access to port 80 was restricted by `iptables` rules until a specific "knock" (SYN packet) was received on the designated trigger port (8080).

## 🛠 Resolution
1. Sent a targeted SYN packet to the trigger port identified in the configuration:
```bash
nc -zv localhost 8080
```
2. Verified that the firewall rule was updated and port 80 became accessible:
```bash
curl localhost
```

## 💡 Lessons Learned
* **Reconnaissance over Brute Force:** Checking process lists (`ps`) and configuration files (`/etc/`) is more efficient and stealthier than full port scans.
* **Mechanism of Port Knocking:** Understanding how `knockd` interacts with `iptables` to manage dynamic access control.
* **Diagnostic Tools:** Using `nc` (Netcat) as a lightweight tool to send trigger packets without the overhead of a full port scanner.