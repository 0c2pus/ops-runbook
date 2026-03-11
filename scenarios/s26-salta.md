# Scenario: Salta - Docker Container Won't Start

## 🚩 Issue

A Node.js web application in `/home/admin/app` needs to run in a Docker container and be accessible on port `8888`. The existing container exited 3 years ago with status `Exited (1)`.

## 🔍 Investigation

### Step 1: Inspect the application directory
```bash
ls -la /home/admin/app/
cat Dockerfile
```

_Observation: Dockerfile has two bugs:_
- _`EXPOSE 8880` - wrong port, should be `8888`_
- _`CMD ["node", "serve.js"]` - wrong filename, should be `server.js`_

### Step 2: Check existing containers and images
```bash
sudo docker ps -a
sudo docker images
```

_Observation: Image `app:latest` exists but the container exited with error code 1 - caused by the bugs in Dockerfile._

### Step 3: Build fixed image and attempt to run
```bash
sudo docker build -t web-app .
sudo docker run -p 8888:8888 web-app
```

_Observation: Error - port 8888 already in use._

### Step 4: Find what is using port 8888
```bash
sudo ss -tulpn | grep 8888
```

_Observation: nginx is listening on port 8888._

## ✅ Root Cause

Three separate issues:

1. `EXPOSE 8880` in Dockerfile - wrong port number.
2. `CMD ["node", "serve.js"]` - wrong filename, correct file is `server.js`.
3. nginx was running and occupying port 8888, preventing the container from binding to it.

## 🛠 Resolution

**Step 1:** Fix the Dockerfile:
```dockerfile
# Change:
EXPOSE 8880
CMD [ "node", "serve.js" ]

# To:
EXPOSE 8888
CMD [ "node", "server.js" ]
```

**Step 2:** Stop and disable nginx:
```bash
sudo systemctl stop nginx
sudo systemctl disable nginx
```

**Step 3:** Build the corrected image:
```bash
sudo docker build -t web-app .
```

**Step 4:** Run the container with port mapping:
```bash
sudo docker run -p 8888:8888 web-app
```

**Step 5:** Verify:
```bash
curl localhost:8888
```

_Returns `Hello World!`._

## 💡 Lessons Learned

- `EXPOSE` in Dockerfile is documentation only - it does not publish the port to the host. Port mapping (`-p host:container`) is always required at `docker run`.
- Always verify the CMD filename matches the actual file in the directory - a typo in the entrypoint causes `Exited (1)` with no obvious error message.
- Before running a container on a specific port, check if the port is free with `ss -tulpn | grep <port>`.
- `systemctl stop` stops the service now. `systemctl disable` prevents it from starting on reboot. Both are needed for a permanent fix.