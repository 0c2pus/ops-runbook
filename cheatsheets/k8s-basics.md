# Kubernetes (K8s) Troubleshooting Essentials

A guide for L2 Support Engineers to diagnose and resolve issues within Kubernetes clusters.

## 1. Cluster Status & Resource Overview
* `kubectl get all` - Show all resources (Pods, Services, Deployments) in the current namespace.
* `kubectl get nodes` - Check if nodes are in `Ready` or `NotReady` state.
* `kubectl get pods -o wide` - List pods with more details, including Node IP and Pod IP.
* `kubectl get events --sort-by='.lastTimestamp'` - See a chronological list of events in the cluster (critical for finding sudden failures).
* `kubectl get nodes --show-labels` - Show all nodes with their assigned labels (critical for nodeSelector issues).
* `kubectl describe node <node_name>` - Detailed info about a node, including available capacity (CPU/RAM) and Taints.

## 2. Pod Diagnostics (The First Line of Defense)
* `kubectl describe pod <pod_name>` - **Most Important:** Shows detailed status, lifecycle events, and error messages (e.g., ImagePullBackOff, CrashLoopBackOff).
* `kubectl logs <pod_name>` - View the standard output (STDOUT) logs from the container.
* `kubectl logs <pod_name> --previous` - View logs from a previous, crashed instance of a pod.
* `kubectl exec -it <pod_name> -- /bin/bash` - Open an interactive shell inside a running container for direct inspection.

## 3. Service & Connectivity
* `kubectl get svc` - List all services and their ClusterIP/ExternalIP.
* `kubectl describe svc <service_name>` - Check if the service has active **Endpoints** (if Endpoints is `<none>`, the service selector doesn't match any pods).
* `kubectl port-forward <pod_name> 8080:80` - Forward a local port to a pod's port to test connectivity bypass-ing the LoadBalancer.

## 4. Workload Management & Fixes
* `kubectl apply -f <file.yml>` - Apply changes from a manifest file.
* `kubectl edit deployment <deployment_name>` - Edit a resource's configuration on the fly in the default text editor.
* `kubectl rollout restart deployment <deployment_name>` - Restart all pods in a deployment (standard way to refresh a service).
* `kubectl delete pod <pod_name>` - Manually kill a pod (K8s will automatically recreate it if it's part of a Deployment).

## 5. Common Pod States for L2 to Know
* **Pending:** The cluster cannot schedule the pod (e.g., lack of CPU/RAM on nodes).
* **CrashLoopBackOff:** The application starts but immediately crashes (check logs).
* **ImagePullBackOff:** Incorrect image name or lack of permissions to the registry.
* **Terminating:** A pod is stuck in deletion, often due to a finalizer or unresponsive node.

## 6. RBAC - Role Based Access Control
RBAC controls what service accounts (used by pods) are allowed to do in the cluster. Missing permissions cause 403 Forbidden errors in pod logs.

* `kubectl get roles,rolebindings,clusterroles,clusterrolebindings` - List all RBAC resources in the cluster.
* `kubectl describe clusterrole <name>` - Show what resources and verbs a ClusterRole permits.
* `kubectl describe clusterrolebinding <name>` - Show which service account a ClusterRole is assigned to.
* `kubectl edit clusterrole <name>` - Edit a ClusterRole to add or remove permissions.

**Common verbs:** `get`, `list`, `watch`, `create`, `update`, `patch`, `delete`.
Note: `list` retrieves a collection, `get` retrieves a specific resource - both are often needed together.

**Debugging RBAC errors:**
A 403 error in pod logs like:
`User "system:serviceaccount:default:myapp-sa" cannot get resource "pods/log"` means the ServiceAccount is missing the `get` verb on `pods/log` in its ClusterRole.