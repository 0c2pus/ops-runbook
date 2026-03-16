# Network Diagnostics & Connectivity Troubleshooting

Tools for verifying network configurations, DNS, SSL, and traffic flow.

## 1. Interface & Routing
* `ip -c addr` — Display IP addresses with color highlights for easier reading.
* `ip route show` — View the routing table and default gateway.
* `ethtool <interface>` — Check physical link speed and duplex mode (1000Mb/s vs 100Mb/s).

## 2. Advanced Connectivity & API Testing (curl)
* `curl -Iv https://example.com` — Show HTTP response headers and SSL certificate details.
* `curl -Lo /dev/null -s -w "%{http_code}\n" <URL>` — Quickly check HTTP status code only.
* `curl -x http://proxy:8080 <URL>` — Test connectivity through a specific proxy server.
* `curl -A "Mozilla/5.0" <url>` - Send request with a custom User Agent string. Use when a server blocks curl but responds to browsers.
* `curl -v <url>` — Verbose mode: shows request headers including User-Agent being sent.

## 3. Port & Service Discovery
* `ss -tulpn` — List all listening TCP/UDP ports with PIDs (Replacement for `netstat`).
* `nc -zv <host> <port>` — Test if a remote port is reachable (TCP).
* `nc -uzv <host> <port>` — Test if a remote port is reachable (UDP).
* `nmap -sV -p <port> <host>` — Detect service version running on a specific port.

## 4. DNS & SSL Verification
* `dig +short <hostname>` — Quick IP resolution.
* `dig @8.8.8.8 <hostname>` — Test resolution using a specific DNS server (e.g., Google).
* `openssl s_client -connect <host>:443 -showcerts` — Inspect the full SSL certificate chain.
* `openssl x509 -in cert.crt -noout -enddate` — Check if a certificate has expired.

## 5. Traffic Analysis & Latency
* `mtr -rw <hostname>` — Combined Ping + Traceroute report (shows packet loss at each hop).
* `tcpdump -i any port 80 -nn -v` — Capture traffic on port 80 (without resolving hostnames for speed).
* `conntrack -L` — (Requires sudo) List all active network connections tracked by the firewall.