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
* [s07-minneapolis.md](./scenarios/s07-minneapolis.md) — Bash Automation: splitting CSV data into multiple chunks while preserving headers and specific naming conventions.
* [s08-saint-paul.md](./scenarios/s08-saint-paul.md) — Data Engineering: efficiently merging hundreds of CSV files into a single dataset using stream processing.
* [s09-bata.md](./scenarios/s09-bata.md) — System Inspection: navigating the `/proc` virtual filesystem and isolating data using stream filters (`grep`, `cut`, `awk`).
* [s10-geneva.md](./scenarios/s10-geneva.md) — Web Security: Identifying and renewing an expired SSL certificate while maintaining identical metadata and updating Nginx configuration.
* [s11-tokamachi.md](./scenarios/s11-tokamachi.md) — IPC: diagnosing and fixing a named pipe writer that blocks due to buffer saturation.
* [s12-kampot.md](./scenarios/s12-kampot.md) — Networking: redirecting local port 80 to an application port using iptables NAT OUTPUT chain.
* [s13-cairo.md](./scenarios/s13-cairo.md) — systemd & Firewall: enabling a disabled health check timer and removing a hidden iptables DROP rule blocking local traffic.
* [s14-alexandria.md](./scenarios/s14-alexandria.md) — Cron & Bash: diagnosing a silently failing backup job caused by a stale lock file and a misconfigured crontab entry.
</details>

<details>
<summary>Medium Level</summary>

* [s15-manhattan.md](./scenarios/s15-manhattan.md) — Disk & PostgreSQL: diagnosing a failed database service caused by a 100% full disk volume and a misleading parent systemd unit status.
* [s16-tokyo.md](./scenarios/s16-tokyo.md) — Apache & Firewall: unblocking port 80 via iptables and fixing file permissions causing a 403 Forbidden response.
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