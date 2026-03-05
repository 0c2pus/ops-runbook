# Scenario: Kampot - Redirect Port 80 to Application Port 20280

## 🚩 Issue

A Python app (managed by `supervisor`) runs on port `20280` and cannot be stopped or reconfigured. A legacy monitoring system expects the service on port `80`. The task is to make the app accessible on port `80` locally without touching the application configuration.

## 🔍 Investigation

### Step 1: Verify port states

Confirm which ports are open and what is running on them:
```bash
nmap -sV -p 80 localhost
nmap -sV -p 20280 localhost
```

_Observation: Port `80` is closed. Port `20280` is open and serves a Python HTTP app. The app cannot be moved - port forwarding is required._

## ❌ What Didn't Work

- `PREROUTING` chain in iptables nat table - does not intercept locally generated traffic (e.g. `curl localhost`). Only affects traffic arriving from external interfaces.
```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 20280
```

## ✅ Root Cause

The legacy monitoring system expects the service on port `80`, but the app is hardcoded to `20280`. Since the app cannot be reconfigured, traffic must be redirected at the kernel level using iptables nat rules.

`PREROUTING` only handles inbound external traffic. For local traffic generated on the same host, the `OUTPUT` chain must be used instead.

## 🛠 Resolution

Added an iptables rule in the `OUTPUT` chain to redirect local traffic from port `80` to port `20280`:
```bash
sudo iptables -t nat -A OUTPUT -p tcp --dport 80 -j REDIRECT --to-port 20280
```

Verified the fix:
```bash
curl localhost:80/accounts
```

_Returned the expected JSON response with bank account data._

## 💡 Lessons Learned

- `iptables PREROUTING` intercepts external inbound traffic only. For traffic originating on the same host, use the `OUTPUT` chain.
- To cover both external and local traffic simultaneously, apply both rules:
```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 20280
sudo iptables -t nat -A OUTPUT -p tcp --dport 80 -j REDIRECT --to-port 20280
```

- `supervisor` is a process manager that keeps services running automatically - similar to `systemd` but typically used for application-level processes.