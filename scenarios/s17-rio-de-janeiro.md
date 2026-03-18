# Scenario: Rio de Janeiro - Do We Have Another Option?

## 🚩 Issue

Jenkins service is failing to start. `systemctl status jenkins.service` shows `inactive (dead)`.

## 🔍 Investigation

### Step 1: Check service logs
```bash
journalctl -u jenkins.service | tail
```

_Observation: Key error:_
```bash
jenkins: failed to find a valid Java installation
```
_Service crashes immediately and restarts repeatedly until hitting the restart limit._

### Step 2: Verify Java exists
```bash
which java
ls -la /usr/bin/java
ls -la /etc/alternatives/java
```

_Observation: `/usr/bin/java` is a symlink chain pointing to `/usr/lib/jvm/temurin-8-jdk-amd64/bin/java`. Java exists but the version may be incompatible._

### Step 3: Check Java version compatibility
```bash
jenkins --version
java -version
```

_Observation: Error reveals the incompatibility:_
```bash
UnsupportedClassVersionError: compiled by class file version 55.0, this runtime only recognizes up to 52.0
```
_Java 8 (`52.0`) is active but Jenkins requires at least Java 11 (`55.0`)._

### Step 4: Check available Java versions
```bash
ls /usr/lib/jvm/
```

_Observation: Java 21 (`java-21-openjdk-amd64`) is installed alongside Java 8._

## ❌ What Didn't Work

- Starting Jenkins without fixing Java version - service crashed immediately with `UnsupportedClassVersionError`.

## ✅ Root Cause

Jenkins was compiled for a newer Java version than the one currently active on the system. Java 8 was set as the system default via `update-alternatives`, but this version of Jenkins requires Java 11 or higher. Java 21 was already installed but not selected.

## 🛠 Resolution

**Step 1:** Switch system Java to version 21:
```bash
sudo update-alternatives --config java
# Select Java 21 from the menu
```

**Step 2:** Start Jenkins:
```bash
sudo systemctl start jenkins.service
```

**Step 3:** Verify:
```bash
curl -s localhost:8888/login | grep Jenkins | head -n1
```

_Returns HTML containing `Sign in - Jenkins`._

## 💡 Lessons Learned

- `failed to find a valid Java installation` from Jenkins means Java version mismatch, not that Java is missing entirely.
- `UnsupportedClassVersionError` with class file versions is a reliable indicator of Java version incompatibility. Class file version `52.0` = Java 8, `55.0` = Java 11, `61.0` = Java 17, `65.0` = Java 21.
- `update-alternatives --config java` is the correct way to switch between multiple Java versions on Debian/Ubuntu - it updates the symlink chain system-wide without touching individual files.
- Always check `ls /usr/lib/jvm/` before installing a new Java version - the required version may already be installed but not selected.