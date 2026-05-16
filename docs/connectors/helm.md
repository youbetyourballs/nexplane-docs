# Helm Connector

The Helm connector manages Kubernetes application releases via the Helm CLI.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Kubeconfig | Yes | kubeconfig YAML |
| Default Namespace | No | Default namespace for releases |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `helm_upgrade` | Run `helm upgrade --atomic` on a release | `helm rollback` to previous revision |
| `helm_rollback` | Explicitly roll back a release to a previous revision | N/A |

**Atomic flag:** `--atomic` causes Helm to automatically roll back the release if the deployment fails its health check within the configured timeout.

The plan diff (`helm diff`) renders in the CR detail view before approval, showing exactly which resources will change.
