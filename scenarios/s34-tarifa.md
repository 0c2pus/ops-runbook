# Scenario: Tarifa - Between Two Seas

## 🚩 Issue

HAProxy is load-balancing two nginx containers but `curl localhost:5000` only returns responses from `nginx_0`. `nginx_1` is never reached.

## 🔍 Investigation

### Step 1: Test the endpoint
```bash
curl localhost:5000
```

_Observation: Always returns `hello there from nginx_0` - no load balancing._

### Step 2: Inspect haproxy.cfg
```bash
cat haproxy.cfg
```

_Observation: HAProxy is configured with `roundrobin` balance and points to `nginx_0:80` and `nginx_1:80`._

### Step 3: Check nginx configs
```bash
cat custom-nginx_0.conf
cat custom-nginx_1.conf
```

_Observation: `nginx_0` listens on port `80` but `nginx_1` listens on port `81`. HAProxy points both to port `80` - `nginx_1` is unreachable._

### Step 4: Check docker-compose networks
```bash
cat docker-compose.yml
```

_Observation: Two separate problems:_
1. _`nginx_1` listens on port `81` but HAProxy config points to port `80`_
2. _`nginx_1` is on `backend_network` only. HAProxy is on `frontend_network` only. They cannot communicate._

## ✅ Root Cause

Two separate issues prevented load balancing:

1. Port mismatch - `haproxy.cfg` pointed to `nginx_1:80` but nginx_1 listens on port `81`.
2. Network isolation - `nginx_1` and `haproxy` were on different Docker networks with no shared network between them.

## 🛠 Resolution

**Step 1:** Fix port in `haproxy.cfg`:
```bash
server nginx_1 nginx_1:81 check
```

**Step 2:** Add `backend_network` to HAProxy in `docker-compose.yml`:
```yaml
haproxy:
  networks:
    - frontend_network
    - backend_network
```

**Step 3:** Apply changes:
```bash
docker compose up -d
```

**Step 4:** Verify load balancing:
```bash
curl localhost:5000
curl localhost:5000
```

_Returns `hello there from nginx_0` and `hello there from nginx_1` alternately._

## 💡 Lessons Learned

- In Docker Compose, containers can only communicate if they share at least one network. Containers on different networks are completely isolated.
- `docker compose restart` does NOT apply changes from `docker-compose.yml` - it only restarts containers. Use `docker compose up -d` to apply config changes.
- Always verify port numbers match between the proxy config and the actual service config - a port mismatch causes silent failures where the proxy simply skips the backend.