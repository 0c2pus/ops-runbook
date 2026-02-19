# Network Diagnostics & Connectivity Troubleshooting

This guide covers tools for verifying network configurations, testing connectivity, and identifying port-level issues.

## 1. Local Interface & Routing
Check local IP addresses and the routing table.
* `ip addr show` — Display all network interfaces and assigned IP addresses.
* `ip route show` — View the system's routing table (check the default gateway).
* `nmcli device status` — Quick status of network interfaces (managed by NetworkManager).

## 2. Connectivity Testing
Verify if a remote host or service is reachable.
* `ping -c 4 <hostname_or_IP>` — Check ICMP reachability and latency.
* `traceroute <hostname>` — Trace the path packets take to a network host.
* `mtr <hostname>` — A combination of ping and traceroute for real-time path analysis.

## 3. Port & Service Verification
Check if specific services are listening or reachable through firewalls.
* `ss -tulpn` — List all active listening TCP/UDP ports and their PIDs.
* `nc -zv <host> <port>` — (Netcat) Scan a specific port to see if it's open (e.g., `nc -zv google.com 443`).
* `telnet <host> <port>` — Alternative way to test connectivity to a specific port.
* `nmap -p <port> <host>` — Advanced port scanning (if installed).

## 4. DNS Troubleshooting
Resolve hostnames to IP addresses and verify DNS records.
* `dig <hostname>` — Detailed DNS lookup (Query, Answer, and Authority sections).
* `nslookup <hostname>` — Simple DNS query tool.
* `host <hostname>` — Quick DNS resolution.
* `cat /etc/resolv.conf` — Check configured DNS nameservers.

## 5. Traffic & Statistics
Monitor active connections and network load.
* `netstat -ant` — View all active TCP connections.
* `tcpdump -i eth0` — Capture and analyze network packets (requires sudo).
* `iftop` — Real-time bandwidth usage by host (if installed).