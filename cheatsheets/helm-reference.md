# Helm - Kubernetes Package Manager

Helm is a package manager for Kubernetes. Charts are YAML templates with variables replaced by values.yaml at deploy time.

## 1. Chart & Release Management
* `helm list` - List all deployed releases with status and revision.
* `helm install <release-name> <chart-path>` - Deploy a chart for the first time.
* `helm upgrade <release-name> <chart-path> -f <values-file>` - Apply changes to an existing release. Always pass `-f` explicitly to ensure updated values are used.
* `helm uninstall <release-name>` - Remove a deployed release and all its resources.
* `helm rollback <release-name> <revision>` - Roll back to a previous revision.

## 2. Inspecting Releases
* `helm get values <release-name>` - Show values currently used by a release.
* `helm get manifest <release-name>` - Show rendered YAML that Helm applied to the cluster.
* `helm history <release-name>` - Show revision history of a release.
* `helm status <release-name>` - Show current status of a release.

## 3. Key Concepts
- **Chart** - A folder with YAML templates and `values.yaml`.
- **Release** - A deployed instance of a chart with specific values.
- **values.yaml** - Default configuration values for the chart.
- Variables in templates use `{{ .Values.key }}` syntax.

## 4. Common Pitfalls
- `helm upgrade` without `-f values.yaml` uses previously stored values - always pass `-f` explicitly when you changed the values file.
- ConfigMaps must be mounted as volumes in `deployment.yaml` to be visible inside containers - creating them in Kubernetes is not enough.
- YAML indentation errors cause `mapping values are not allowed` - check spacing carefully.