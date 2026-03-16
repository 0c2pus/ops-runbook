# Scenario: Paris - Where is my Webserver?

## 🚩 Issue

A Flask web server runs on `localhost:5000` with a password. `curl localhost:5000` returns `Unauthorized` and credentials are unknown.

## 🔍 Investigation

### Step 1: Check the service and port
```bash
systemctl status flaskapp.service
curl localhost:5000
```

_Observation: Service is running. curl returns `Unauthorized` - but checking journalctl shows the server returns HTTP 200, not 401. This means `Unauthorized` is the actual page content, not an HTTP error._

### Step 2: Check available files
```bash
ls -la /home/admin/
cat /etc/systemd/system/flaskapp.service
```

_Observation: `webserver.py` exists but has `rwxrwx---` permissions with `root:root` ownership - unreadable without sudo. Service file has no environment variables or EnvironmentFile directive._

### Step 3: Identify the trick

_Observation: The server returns different content based on the HTTP User Agent header. curl and wget are blocked by the application code - they receive `Unauthorized`. Browser-like User Agents receive the real content._

## ✅ Root Cause

The Flask application checks the `User-Agent` header of incoming requests. Requests from `curl` or `wget` are blocked and receive `Unauthorized`. Requests with a browser-like User Agent receive the actual password page.

## 🛠 Resolution

Send the request with a browser User Agent:
```bash
curl -A "Mozilla/5.0" localhost:5000
```

_Returns: `Welcome! Password is FDZPmh5AX3oiJt`_

Save the solution:
```bash
echo "FDZPmh5AX3oiJt" > ~/mysolution
```

## 💡 Lessons Learned

- The `User-Agent` HTTP header identifies the client making the request. Servers can respond differently based on this value.
- `curl` sends `curl/7.x.x` as its User Agent by default. Use `-A` flag to override it: `curl -A "Mozilla/5.0" <url>`.
- In real L2 work: if curl fails but a browser works, always check if the server blocks requests by User Agent - common in WAF rules and CDN configs.
- `Unauthorized` as page content with HTTP 200 is different from HTTP 401 Unauthorized - always check the actual HTTP status code, not just the text.