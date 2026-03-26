# Scenario: Bekasi - Supervisor is Still Around

## 🚩 Issue
nginx is running on port 443, but `curl -k https://bekasi` returns `Failed to start the server. Please check the setup` instead of `Hello SadServers!`

## 🔍 Investigation

### Step 1: Check curl and nginx
```bash
curl -k https://bekasi
systemctl status nginx
```
_Note: nginx is running normally, the error is returned by the Python app._

### Step 2: Check supervisor and the app process
```bash
systemctl status supervisor
sudo supervisorctl status
```
_Note: supervisor is running, bekasi process is in RUNNING state._

### Step 3: Run the app manually
```bash
cd /home/admin/bekasi
bin/python wsgi.py
# in second terminal:
curl http://127.0.0.1:5000
```
_Note: returns `Hello SadServers!` manually - different result than under supervisor._

### Step 4: Find environment variables
```bash
cat ~/bekasi/bekasi.py   # check_env() checks for BEKASI_SERVER and BEKASI_USER
cat ~/.bashrc            # variables are defined here for admin user
```
_Note: supervisor does not read ~/.bashrc on startup - variables are unavailable to the process._

### Step 5: Check supervisor config
```bash
cat /etc/supervisor/conf.d/uwsgi.conf
```
_Note: `environment` directive is missing._

## ❌ What Didn't Work
Looking for the issue in nginx and bekasi.ini - the problem was in the process execution environment.

## ✅ Root Cause
Supervisor launches uwsgi in an isolated environment without variables from `~/.bashrc`. `BEKASI_SERVER` and `BEKASI_USER` were defined only for the admin user but were not passed to the supervisor process via the `environment` directive.

## 🛠 Resolution
Add to `/etc/supervisor/conf.d/uwsgi.conf`:
```bash
environment=BEKASI_SERVER="bekasi.sadservers.com",BEKASI_USER="admin"
```
Reload supervisor configuration:
```bash
sudo supervisorctl reread
sudo supervisorctl update
```

## 💡 Lessons Learned
- Supervisor does not read `~/.bashrc` - environment variables must be explicitly passed via `environment=` in the config file.
- If the app works manually but not as a service - first suspect: difference in execution environment.
- `supervisorctl reread` + `update` is the correct way to apply supervisor config changes without restarting the whole service.
- To debug supervisor processes: `sudo supervisorctl`, then `?` for available commands, `tail -f <process>` shows process logs.