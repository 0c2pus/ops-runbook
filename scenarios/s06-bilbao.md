# Scenario: Bilbao - Troubleshooting Kubernetes Pod Scheduling

## 🚩 Issue
An Nginx deployment is failing to start. The pod remains in the `Pending` state, and the service (Load Balancer) is unreachable.

## 🔍 Investigation

### Step 1: Initial Health Check
Verified the service status and pod state:
```bash
kubectl get all
```
**Observation:** Pod `nginx-deployment-xxx` is in `Pending` status. Service `nginx-service` is active but has no healthy endpoints to route traffic to.

### Step 2: Deep Dive with Describe
Inspected the pod's lifecycle events:
```bash
kubectl describe pod <pod_name>
```
**Key Error in Events:**
`Warning FailedScheduling: 0/2 nodes are available: 1 node(s) didn't match Pod's node affinity/selector, 1 node(s) had untolerated taint.`

### Step 3: Node Labels Verification
Checked if any nodes satisfy the `nodeSelector` requirement (`disk=ssd`) specified in `manifest.yml`:
```bash
kubectl get nodes --show-labels
```
**Result:** None of the available nodes have the `disk=ssd` label. Additionally, nodes were in `NotReady` or restricted states.

## ✅ Root Cause
1. **Unmet Node Selector**: The pod was restricted to nodes with `disk=ssd`, which did not exist in the cluster.
2. **Resource Over-requesting**: The manifest requested `2000Mi` of RAM, which exceeded the available capacity of the nodes

## 🛠 Resolution
### Step 1: Edit the Manifest
Modified `manifest.yml` to align with cluster capabilities:
* Removed/Commented out the `nodeSelector` block.
* Reduced `resources.requests.memory` and `resources.limits.memory` from `2000Mi` to `100Mi`.

### Step 2: Apply Changes
```bash
kubectl apply -f manifest.yml
```

### Step 3: Verification
```bash
kubectl get pods  # Status changed to Running
curl 10.43.216.196 # Returns Nginx Welcome Page
```

## 💡 Lessons Learned
* **Scheduling Constraints**: `nodeSelector` is a "hard" constraint. If no nodes match, the pod will stay `Pending` indefinitely.
* **Resource Discipline**: Always check node capacity (`kubectl describe node`) before setting high resource requests in deployments.
* **Status NotReady**: Pods cannot be scheduled on nodes that are in the `NotReady` state.