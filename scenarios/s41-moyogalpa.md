# Scenario: Moyogalpa - Security Snag

## 🚩 Issue

A Golang webapp service fails to start and serve HTTPS content. `curl https://webapp:7000/users.html` cannot resolve host.

## 🔍 Investigation

### Step 1: Find the service
```bash
sudo systemctl list-units | grep -i web
systemctl status webapp.service
```

_Observation: `webapp.service` exists but logs show:_
```bash
open /home/webapp/pki/server.crt: permission denied
```

### Step 2: Check file permissions
```bash
sudo ls -la /home/webapp/pki/
ls -la /home/webapp/static-files/
```

_Observation: `pki/` owned by `root:root` with `700` - `webapp` user cannot enter. `static-files/` owned by `admin:admin` - `webapp` cannot read._

### Step 3: Check SSL certificate CN
```bash
echo QUIT | openssl s_client -connect localhost:7000 -showcerts 2>/dev/null | openssl x509 -noout -subject -ext subjectAltName
```

_Observation: Certificate issued for `127.0.10.1`, not `127.0.0.1`._

## ✅ Root Cause

Five separate issues:

1. `pki/` directory and certificates inaccessible to `webapp` user.
2. `static-files/` owned by `admin` - webapp cannot read HTML files.
3. `webapp` hostname not in `/etc/hosts`.
4. CA certificate not trusted by system.
5. AppArmor profile missing rules for `static-files/` access.

## 🛠 Resolution

**Step 1:** Fix certificate permissions:
```bash
sudo chown -vR webapp:webapp /home/webapp/pki
sudo chmod -v 0600 /home/webapp/pki/server.crt /home/webapp/pki/server.pem
```

**Step 2:** Fix static files permissions:
```bash
sudo chown -R webapp:webapp /home/webapp/static-files
sudo chmod -R 0640 /home/webapp/static-files/*
```

**Step 3:** Add CA to system trust store:
```bash
sudo cp /home/webapp/pki/CA.crt /usr/local/share/ca-certificates/CA.crt
sudo chmod 0644 /usr/local/share/ca-certificates/CA.crt
sudo update-ca-certificates
```

**Step 4:** Add correct hostname to `/etc/hosts`:
```bash
echo '127.0.10.1 webapp' | sudo tee -a /etc/hosts
```

**Step 5:** Fix AppArmor profile - add to `/etc/apparmor.d/usr.local.bin.webapp`:
```bash
/home/webapp/static-files/ r,
/home/webapp/static-files/* r,
```

Reload profile:
```bash
sudo apparmor_parser -r /etc/apparmor.d/usr.local.bin.webapp
```

**Step 6:** Verify:
```bash
curl https://webapp:7000/users.html
```

## 💡 Lessons Learned

- When `permission denied` persists despite correct file permissions - check AppArmor. It enforces access at kernel level independently of standard Linux permissions.
- SSL certificate CN/SAN must match the hostname used in the request. Use `openssl s_client` to inspect what hostname the certificate expects.
- `127.0.0.1` and `127.0.10.1` are different addresses - always verify the exact IP in the certificate SAN field.
- `sudo echo "text" >> file` does not work - use `echo "text" | sudo tee -a file` instead.