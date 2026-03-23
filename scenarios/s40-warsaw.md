# Scenario: Warsaw - Prometheus Can't Scrape the Webserver

## 🚩 Issue

A Golang application exposes `/metrics` endpoint on port 9000 but `curl localhost:9000/metrics` returns nothing and `curl localhost:9000` returns `404 Not Found`.

## 🔍 Investigation

### Step 1: Check running containers and ports
```bash
docker ps -a
```

_Observation: Two containers running - `golang-app` on port 9000 and `prometheus` on port 9090. Both are up._

### Step 2: Test endpoints
```bash
curl localhost:9000/metrics
curl localhost:9090/metrics
```

_Observation: Port 9090 (Prometheus) returns metrics correctly. Port 9000 returns nothing - the application endpoint is broken._

### Step 3: Inspect application source code
```bash
cat /home/admin/app/main.go
```

_Observation: The `/metrics` route is registered with `Methods("POST")`:_
```go
router.Handle("/metrics", promhttp.Handler()).Methods("POST")
```

_`curl` sends GET requests by default. The endpoint only accepts POST - all GET requests return 404._

## ✅ Root Cause

The `/metrics` HTTP route was registered with `Methods("POST")` instead of `Methods("GET")`. Prometheus scrapes metrics using GET requests, so all scrape attempts resulted in 404 Not Found.

## 🛠 Resolution

**Step 1:** Fix `main.go` - change `POST` to `GET`:
```go
router.Handle("/metrics", promhttp.Handler()).Methods("GET")
```

**Step 2:** Rebuild and restart the container:
```bash
cd /home/admin/app
docker compose up -d --build
```

**Step 3:** Verify:
```bash
curl localhost:9000/metrics
```

_Returns Prometheus metrics with HTTP 200._

## 💡 Lessons Learned

- Prometheus scrapes metrics using HTTP GET - endpoints registered with POST will always return 404 to Prometheus.
- `docker compose restart` does not rebuild images - use `docker compose up -d --build` when application code changes.
- When debugging a web endpoint, always check the HTTP method in the route definition - a method mismatch causes silent 404 errors.
- Source code inspection is often faster than network debugging - reading `main.go` immediately revealed the bug.