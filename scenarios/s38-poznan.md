# Scenario: Poznań - Helm Chart Issue in Kubernetes

## 🚩 Issue

A Helm chart `web_chart` deploys nginx but shows the default nginx page instead of the custom HTML. Also needs 3 replicas instead of 1.

## 🔍 Investigation

### Step 1: Inspect chart structure
```bash
ls ~/web_chart/templates/
cat ~/web_chart/templates/configmap.yaml
cat ~/web_chart/templates/deployment.yaml
cat ~/web_chart/values.yaml
```

_Observation: `configmap.yaml` contains custom HTML. `deployment.yaml` has no `volumes` or `volumeMounts` - ConfigMap is never mounted into the nginx container. `values.yaml` has `replicaCount: 1`._

### Step 2: Check current release
```bash
helm list
helm get values web-chart
```

_Observation: Release `web-chart` is deployed with `replicaCount: 1`._

## ✅ Root Cause

Two issues:

1. `deployment.yaml` does not mount the ConfigMap as a volume at `/usr/share/nginx/html` - nginx serves its default page instead.
2. `replicaCount` is `1` in `values.yaml` instead of `3`.

## 🛠 Resolution

**Step 1:** Add `volumeMounts` and `volumes` to `deployment.yaml`:
```yaml
containers:
  - name: {{ .Chart.Name }}
    ...
    volumeMounts:
      - mountPath: /usr/share/nginx/html
        name: {{ .Release.Name }}-cm-index-html
volumes:
- name: {{ .Release.Name }}-cm-index-html
  configMap:
    name: {{ .Release.Name }}-cm-index-html
    items:
      - key: index.html
        path: index.html
```

**Step 2:** Change `replicaCount` in `values.yaml`:
```yaml
replicaCount: 3
```

**Step 3:** Apply changes with explicit values file:
```bash
helm upgrade web-chart ~/web_chart -f ~/web_chart/values.yaml
```

**Step 4:** Verify:
```bash
kubectl get pods
POD_IP=$(kubectl get pods -n default -o jsonpath='{.items[0].status.podIP}')
curl -s "${POD_IP}"
```

_Returns `Welcome SadServers` page from all 3 pods._

## 💡 Lessons Learned

- Helm is a package manager for Kubernetes. Charts are templates with variables replaced by `values.yaml` at deploy time.
- A ConfigMap must be explicitly mounted as a volume in `deployment.yaml` to be visible inside the container - creating it in Kubernetes is not enough.
- `helm upgrade` without `-f values.yaml` uses previously stored values, not the current file. Always pass `-f` explicitly to ensure updated values are applied.
- YAML indentation errors cause `mapping values are not allowed` errors - always check spacing carefully in Kubernetes manifests.