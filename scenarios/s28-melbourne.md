# Scenario: Melbourne - WSGI with Gunicorn

## 🚩 Issue

`curl http://localhost` returns nothing. The request chain is: `curl → Nginx → Gunicorn → wsgi.py`. All three components need to work correctly for the response to reach the client.

## 🔍 Investigation

### Step 1: Check services and ports
```bash
systemctl status nginx.service
systemctl status gunicorn.service
sudo ss -tulpn
```

_Observation: Nginx is `inactive (dead)`. Port 80 is not listening. Gunicorn is running but nothing serves HTTP._

### Step 2: Start Nginx and retest
```bash
sudo systemctl start nginx.service
curl -s http://localhost
```

_Observation: Returns `502 Bad Gateway` - Nginx is up but cannot reach Gunicorn._

### Step 3: Check Nginx proxy configuration
```bash
cat /etc/nginx/sites-available/default
```

_Observation: `proxy_pass http://unix:/run/gunicorn.socket` - note `.socket` extension._

### Step 4: Check actual socket name
```bash
ls /run/gunicorn*
cat /etc/systemd/system/gunicorn.service
```

_Observation: Gunicorn creates `/run/gunicorn.sock` (`.sock`) but Nginx looks for `/run/gunicorn.socket` (`.socket`) - name mismatch._

### Step 5: Fix socket name in Nginx config and retest
```bash
# Fixed: proxy_pass http://unix:/run/gunicorn.sock;
sudo systemctl restart nginx
curl --unix-socket /run/gunicorn.sock http://localhost
```

_Observation: curl to socket returns nothing - problem is deeper._

### Step 6: Inspect wsgi.py
```bash
cat /home/admin/wsgi.py
```

_Observation: `Content-Length: '0'` but actual body `Hello, world!` is 13 bytes. Client receives header saying body is 0 bytes and ignores the content._

## ✅ Root Cause

Two separate issues:

1. Nginx `proxy_pass` pointed to `/run/gunicorn.socket` but Gunicorn created `/run/gunicorn.sock` - socket name mismatch caused 502.
2. `wsgi.py` declared `Content-Length: '0'` while returning 13 bytes of content - client ignored the body entirely.

## 🛠 Resolution

**Step 1:** Start Nginx:
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

**Step 2:** Fix socket name in Nginx config:
```bash
sudo vim /etc/nginx/sites-available/default
# Change: proxy_pass http://unix:/run/gunicorn.socket;
# To:     proxy_pass http://unix:/run/gunicorn.sock;
sudo systemctl restart nginx
```

**Step 3:** Fix Content-Length in wsgi.py:
```bash
vim /home/admin/wsgi.py
# Change: ('Content-Length', '0')
# To:     ('Content-Length', '13')
```

**Step 4:** Restart Gunicorn to apply wsgi.py changes:
```bash
sudo systemctl restart gunicorn
```

**Step 5:** Verify:
```bash
curl -s http://localhost
```

_Returns `Hello, world!`._

## 💡 Lessons Learned

- `502 Bad Gateway` means Nginx is alive but cannot reach the backend - always check the proxy_pass path and verify the socket/port actually exists.
- Unix socket names must match exactly between the server that creates them and the proxy that connects to them. A `.sock` vs `.socket` difference is enough to break the chain.
- `Content-Length` must match the actual byte length of the response body. A mismatch causes the client to silently ignore content.
- After editing application code, always restart the application server - `reload` is not always supported, use `restart` instead.
- Trace the full request chain step by step: test each hop independently (`curl --unix-socket`) to isolate where the failure occurs.