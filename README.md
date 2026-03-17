# Troubleshooting Lab & Ops Runbook

This repository serves as a knowledge base for Linux system administration, monitoring, and technical incident resolution. It documents diagnostic workflows, engineering cheatsheets, and detailed post-mortems of real-world scenarios from the SadServers platform.

## 📌 Repository Structure

### 📂 [Cheatsheets](./cheatsheets/) — Universal Instructions

A collection of verified algorithms for rapid system diagnostics:

* [Linux Diagnostics](./cheatsheets/linux-diagnostics.md) — Analysis of system load (CPU, RAM, IO) and service states.
* [Network Tools](./cheatsheets/network-tools.md) — Verification of network connectivity, DNS, and port status.
* [Text Processing](./cheatsheets/text-processing.md) — CLI tools for searching, modifying, extracting, and analyzing text data (logs, configs, CSV).
* [Docker Handbook](./cheatsheets/docker-handbook.md) — Diagnostics for containerized applications.
* [SQL Reference](./cheatsheets/sql-reference.md) — Essential queries for data integrity checks and database troubleshooting.
* [Kubernetes (K8s)](./cheatsheets/k8s-basics.md) — A guide to diagnose and resolve issues within Kubernetes clusters.
* [Bash Automation](./cheatsheets/bash-automation.md) — Scripting essentials: loops, variables, and automated file manipulation.
* [SSL Certificates](./cheatsheets/ssl-certificates.md) — SSL/TLS Management: a comprehensive guide for inspecting, generating, and verifying certificates using OpenSSL, including Nginx integration workflows.
* [Server review guide](./cheatsheets/server-review-guide.md) — Server Reconnaissance: A step-by-step guide to discovering a server's purpose, hardware saturation, and application architecture from scratch.
* [Iptables](./cheatsheets/iptables.md) — Firewall & traffic management: port forwarding, NAT rules, filtering, and rule persistence.
* [Cron](./cheatsheets/cron.md) — Task scheduling: crontab management, schedule syntax, output logging, and debugging silent failures.
* [Systemd](./cheatsheets/systemd.md) — Service & unit management: listing, status checking, controlling services and timers, and reading logs with journalctl.
* [Git Reference](./cheatsheets/git-reference.md) — Version control & repository diagnostics: history inspection, commit comparison, and bisect for finding breaking changes.
* [Linux Users & Groups](./cheatsheets/users-and-groups.md) — User and group management: creating users and groups, managing memberships, file ownership, chmod, and filesystem attributes.
* [jq Reference](./cheatsheets/jq-reference.md) — JSON processing in the CLI: field access, array iteration, filtering with select(), and extracting nested data.

### 📂 [Scenarios](./scenarios/) — Incident Analysis (Post-Mortems)

Hands-on cases of resolving issues on live systems. Each scenario includes symptoms, investigation steps, root cause analysis, and dead ends encountered during investigation:

<details>
<summary>Easy Level</summary>

* [s01-saint-john.md](./scenarios/s01-saint-john.md) — Process Management: terminating a rogue logging process filling the disk.
* [s02-saskatoon.md](./scenarios/s02-saskatoon.md) — Filesystem: identifying and reclaiming exhausted disk space.
* [s03-taipei.md](./scenarios/s03-taipei.md) — Text Processing: analyzing large-scale log files.
* [s04-lhasa.md](./scenarios/s04-lhasa.md) — Data Processing: calculating a truncated two-decimal average from structured text data.
* [s05-bucharest.md](./scenarios/s05-bucharest.md) — Database Connectivity: troubleshooting a PostgreSQL authentication configuration issue.
* [s06-bilbao.md](./scenarios/s06-bilbao.md) — Kubernetes Pod Scheduling: troubleshooting a pod scheduling issue due to node selector and resource constraints.
* [s07-apia.md (Pro)](./scenarios/s07-apia.md) — Text Processing: identifying a unique file among identical ones by size and extracting the added word using diff.
* [s08-gitega.md (Pro)](./scenarios/s08-gitega.md) — Git: locating the first bad commit in a repository using git bisect binary search.
* [s09-minneapolis.md](./scenarios/s09-minneapolis.md) — Bash Automation: splitting CSV data into multiple chunks while preserving headers and specific naming conventions.
* [s10-saint-paul.md](./scenarios/s10-saint-paul.md) — Data Engineering: efficiently merging hundreds of CSV files into a single dataset using stream processing.
* [s11-bata.md](./scenarios/s11-bata.md) — System Inspection: navigating the `/proc` virtual filesystem and isolating data using stream filters (`grep`, `cut`, `awk`).
* [s12-geneva.md](./scenarios/s12-geneva.md) — Web Security: Identifying and renewing an expired SSL certificate while maintaining identical metadata and updating Nginx configuration.
* [s13-tokamachi.md](./scenarios/s13-tokamachi.md) — IPC: diagnosing and fixing a named pipe writer that blocks due to buffer saturation.
* [s14-yokohama.md (Pro)](./scenarios/s14-yokohama.md) — Linux Users & Permissions: creating a shared group, managing cross-user file access, and setting append-only file attributes with chattr.
* [s15-fukuoka.md (Pro)](./scenarios/s15-fukuoka.md) — Nginx & Permissions: fixing directory and file access rights along a symlink chain to resolve 403 and 404 errors.
* [s16-kampot.md](./scenarios/s16-kampot.md) — Networking: redirecting local port 80 to an application port using iptables NAT OUTPUT chain.
* [s17-rio-de-janeiro.md (Pro)](./scenarios/s17-rio-de-janeiro.md) — Java & systemd: diagnosing a Jenkins startup failure caused by an incompatible Java version and switching to the correct one via update-alternatives.
* [s18-nuuk.md (Pro)](./scenarios/s18-nuuk.md) — SSH: restoring correct directory permissions on ~/.ssh to fix publickey authentication failure.
* [s19-cairo.md](./scenarios/s19-cairo.md) — systemd & Firewall: enabling a disabled health check timer and removing a hidden iptables DROP rule blocking local traffic.
* [s20-alexandria.md](./scenarios/s20-alexandria.md) — Cron & Bash: diagnosing a silently failing backup job caused by a stale lock file and a misconfigured crontab entry.
* [s21-kortenberg.md (Pro)](./scenarios/s21-kortenberg.md) — Linux Permissions: diagnosing and permanently fixing a corrupted umask setting in /etc/profile that caused all new files to be created with no permissions.
* [s22-hamburg.md (Pro)](./scenarios/s22-hamburg.md) — JSON & AWS: filtering AWS EBS volume data using jq to identify a specific volume by multiple criteria.
</details>

<details>
<summary>Medium Level</summary>

* [s23-manhattan.md](./scenarios/s23-manhattan.md) — Disk & PostgreSQL: diagnosing a failed database service caused by a 100% full disk volume and a misleading parent systemd unit status.
* [s24-tokyo.md](./scenarios/s24-tokyo.md) — Apache & Firewall: unblocking port 80 via iptables and fixing file permissions causing a 403 Forbidden response.
* [s25-capetown.md](./scenarios/s25-capetown.md) — Nginx & systemd: fixing a syntax error in site config and raising a deliberately low LimitNOFILE that caused 500 errors.
* [s26-salta.md](./scenarios/s26-salta.md) — Docker: fixing Dockerfile bugs (wrong port and entrypoint filename) and resolving a port conflict with nginx to run a Node.js container.
* [s27-oaxaca.md](./scenarios/s27-oaxaca.md) — Linux File Descriptors: closing an open file descriptor in a running process without killing it using exec and lsof.
* [s28-melbourne.md (Pro)](./scenarios/s28-melbourne.md) — Nginx & Gunicorn: fixing a broken WSGI request chain caused by a socket name mismatch and incorrect Content-Length header.
* [s29-kihei.md](./scenarios/s29-kihei.md) — Disk Space: converting a large file to a sparse file to free real disk blocks while preserving the logical file size required by the application.
* [s30-unimak-island.md (Pro)](./scenarios/s30-unimak-island.md) — JSON: filtering nested station data using jq select with boolean and numeric conditions.
* [s31-ivujivik.md (Pro)](./scenarios/s31-ivujivik.md) - CSV Processing: fixing Windows line endings, identifying column indexes, and filtering with awk conditions to find a specific electoral district.
* [s32-paris.md](./scenarios/s32-paris.md) — HTTP: bypassing a User-Agent based block in a Flask application by spoofing a browser User-Agent with curl -A.
* [s33-buenos-aires.md](./scenarios/s33-buenos-aires.md) - Kubernetes RBAC: fixing a CrashLoopBackOff caused by missing `get` verb on `pods/log` in a ClusterRole.
* [s34-tarifa.md (Pro)](./scenarios/s34-tarifa.md) — Docker Compose & HAProxy: fixing load balancing by correcting a port mismatch and adding a missing shared network between HAProxy and a backend nginx container.
</details>

### 📂 [Templates](./templates/) — Incident Documentation

Standardized templates for incident reporting and post-mortem documentation:

* [incident-report-template.md](./templates/incident-report-template.md) — Blank IR template following L2/L3 support standards.
* [incident-report-example.md](./templates/incident-report-example.md) — Filled example based on a real SadServers scenario.

## 🛠 Troubleshooting Methodology

I follow a standardized L2/L3 support cycle when resolving incidents:

1. **Identification:** Symptom collection and impact assessment.
2. **Investigation:** Utilizing system utilities to localize the issue.
3. **Root Cause Analysis (RCA):** Identifying the primary cause of the failure.
4. **Resolution:** Implementing the fix and verifying system health.
5. **Documentation:** Recording the solution to prevent recurrence.