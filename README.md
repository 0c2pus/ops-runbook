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

### 📂 [Scenarios](./scenarios/) — Incident Analysis (Post-Mortems)

Hands-on cases of resolving issues on live systems. Each scenario includes symptoms, investigation steps, root cause analysis, and dead ends encountered during investigation:

* [s01-saint-john.md](./scenarios/s01-saint-john.md) — Process Management: terminating a rogue logging process filling the disk.
* [s02-saskatoon.md](./scenarios/s02-saskatoon.md) — Filesystem: identifying and reclaiming exhausted disk space.
* [s03-taipei.md](./scenarios/s03-taipei.md) — Text Processing: analyzing large-scale log files.

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