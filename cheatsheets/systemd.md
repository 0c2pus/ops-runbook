# systemd — Service & Unit Management

Essential commands for managing, inspecting, and debugging systemd units (services, timers, slices) on Linux systems.

## 1. Listing & Discovering Units
* `systemctl list-units` - List all currently loaded and active units.
* `systemctl list-units | grep <name>` - Filter units by name to find a specific service.
* `systemctl list-units --all` - Include inactive and failed units in the list.
* `systemctl list-timers` - List all active timers with next trigger time.

## 2. Checking Unit Status
* `systemctl status <name>` - Show status, recent logs, and PID of a unit.
* `systemctl status <name>.service` - Explicitly target a service unit.
* `systemctl status <name>.timer` - Explicitly target a timer unit.
* `systemctl is-enabled <name>` - Check if a unit is enabled for autostart on boot.

**Warning:** `active (exited)` does not mean the service is healthy — always verify the actual process exists with `ps aux | grep <name>`.

## 3. Controlling Units
* `systemctl start <name>` - Start a unit immediately.
* `systemctl stop <name>` - Stop a running unit.
* `systemctl restart <name>` - Stop and start a unit (applies config changes).
* `systemctl reload <name>` - Reload config without stopping the process (if supported).
* `systemctl enable <name>` - Enable autostart on boot (survives reboot).
* `systemctl disable <name>` - Remove from autostart.
* `systemctl enable --now <name>` - Enable and start in one command.

## 4. Reading Logs (journalctl)
* `journalctl -u <name>` - Show all logs for a specific unit.
* `journalctl -u <name> -n 50` - Show last 50 log lines for a unit.
* `journalctl -u <name> -f` - Follow live log output for a unit.
* `journalctl -p 3 -xb` — Show only critical errors from the current boot.

## 5. Common Failure Patterns
| Symptom | What to check |
|---|---|
| `active (exited)` | Real process may not exist - check `ps aux` |
| `failed` | Check `journalctl -u <name>` for root cause |
| Service starts but crashes | Check logs with `journalctl -u <name> -n 100` |
| Timer not firing | Check `systemctl list-timers` and `is-enabled` |

## 6. Resource Limits
* `ulimit -a` - Check all resource limits for the current session. Key value: `open files (-n)`.
* `cat /proc/$(pgrep <service> | head -1)/limits | grep "open files"` - Check the actual open file limit for a running process.
* `cat /proc/sys/fs/file-max` - Check the system-wide maximum allowed open files.

**Increasing limits for a systemd service:**

Edit the unit override file:
```bash
sudo systemctl edit <service>.service
```

Add the following content:
```bash
[Service]
LimitNOFILE=65536
```

Apply changes and verify:
```bash
sudo systemctl daemon-reload
sudo systemctl restart <service>.service
cat /proc/$(pgrep <service> | head -1)/limits | grep "open files"
```

**Note:** Prefer `systemctl edit` over modifying the unit file directly - it creates `/etc/systemd/system/<service>.service.d/override.conf` which survives package updates.