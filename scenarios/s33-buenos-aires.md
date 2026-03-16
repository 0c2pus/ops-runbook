# Scenario: Buenos Aires - Kubernetes Pod Crashing

## 🚩 Issue

Pod `logshipper` is in `CrashLoopBackOff` state. It cannot read logs from the `logger` pod due to insufficient RBAC permissions.

## 🔍 Investigation

### Step 1: Check pod states
```bash
sudo kubectl get pods
sudo kubectl get pods -o wide
```

_Observation: `logger` is running. `logshipper` is in `CrashLoopBackOff` with multiple restarts._

### Step 2: Check logshipper logs
```bash
sudo kubectl logs logshipper-597f84bf4f-6ssjq
```

_Observation: HTTP 403 Forbidden error:_
```bash
User "system:serviceaccount:default:logshipper-sa" cannot get resource "pods/log" in API group "" in the namespace "default"
```

_`logshipper` uses a service account `logshipper-sa` which lacks permission to read pod logs._

### Step 3: Find existing RBAC resources
```bash
sudo kubectl get clusterroles,clusterrolebindings | grep logshipper
```

_Observation: `logshipper-cluster-role` and `logshipper-cluster-role-binding` exist._

### Step 4: Inspect the ClusterRole
```bash
sudo kubectl describe clusterrole logshipper-cluster-role
```

_Observation: The role allows only `list` verb for `pods/log`. The application needs `get` verb to read logs. The annotation says: "Think about what verbs you need to add."_

## ✅ Root Cause

`logshipper-cluster-role` only granted `list` permissions on `pods/log`, `pods`, and `namespaces`. The application requires `get` verb on `pods/log` to read log content from the `logger` pod.

## 🛠 Resolution

Edit the ClusterRole to add `get` verb:
```bash
sudo kubectl edit clusterrole logshipper-cluster-role
```

Add `- get` under verbs for all resources:
```yaml
rules:
- apiGroups:
  - ""
  resources:
  - namespaces
  - pods
  - pods/log
  verbs:
  - list
  - get
```

Verify the change:
```bash
sudo kubectl describe clusterrole logshipper-cluster-role
```

Wait 1-2 minutes for Kubernetes to apply the RBAC change, then verify:
```bash
sudo kubectl get pods
```

_`logshipper` transitions from `CrashLoopBackOff` to `Running`._

## 💡 Lessons Learned

- `CrashLoopBackOff` means a pod starts, crashes, and Kubernetes keeps restarting it. Always check logs first to find the root cause.
- In Kubernetes, RBAC controls what service accounts can do. A pod uses its service account to make API calls - if permissions are missing, the pod fails.
- `ClusterRole` defines permissions. `ClusterRoleBinding` assigns those permissions to a service account. They are separate from the pod definition and can be changed without touching the pod.
- `list` and `get` are different verbs: `list` retrieves a collection, `get` retrieves a specific resource. Both are often needed together.
- RBAC changes take 1-2 minutes to propagate in Kubernetes - always wait before concluding the fix did not work.