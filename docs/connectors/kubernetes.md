# Kubernetes Connector

The Kubernetes connector uses the official Python Kubernetes client to interact with a Kubernetes cluster. It supports secret rotation, service account token management, RBAC inspection, and deployment operations.

## Credential Fields

| Field | Type | Required | Description |
|---|---|---|---|
| Name | string | Yes | Display name for this connector (e.g., `prod-k8s`) |
| Kubeconfig | string | Yes | Full kubeconfig YAML content. Paste the output of `kubectl config view --raw` |
| Context | string | No | Kubeconfig context to use. Defaults to the current-context in the kubeconfig |
| Namespace | string | No | Limit discovery to a specific namespace. Defaults to all namespaces |

## Supported Actions

| Action | Description | Rollback |
|---|---|---|
| Rotate Kubernetes Secret | Updates the `data` field of a Secret with new values | Restore the previous secret data |
| Delete Kubernetes Secret | Deletes a Kubernetes Secret | No rollback |
| Rotate Service Account Token | Deletes existing token secrets, triggering new token generation | No rollback (old token is gone once deleted) |
| Patch Deployment | Updates a deployment's image, replicas, or environment variables | Restore previous deployment spec |
| Scale Deployment | Sets the replica count on a deployment | Restore the previous replica count |
| Cordon Node | Marks a node as unschedulable | Uncordon the node |
| Uncordon Node | Marks a node as schedulable | Cordon the node |

## Minimum Permissions Required

The kubeconfig service account should have a ClusterRole (or Role for namespace-scoped operation) with:

For discovery:
```yaml
rules:
- apiGroups: [""]
  resources: ["pods", "services", "secrets", "serviceaccounts", "namespaces"]
  verbs: ["get", "list"]
- apiGroups: ["apps"]
  resources: ["deployments", "daemonsets", "statefulsets"]
  verbs: ["get", "list"]
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["roles", "rolebindings", "clusterroles", "clusterrolebindings"]
  verbs: ["get", "list"]
```

For full change execution, add:
```yaml
- apiGroups: [""]
  resources: ["secrets", "serviceaccounts"]
  verbs: ["create", "update", "delete", "patch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["update", "patch"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["patch"]
```

## Known Limitations

- The kubeconfig is stored encrypted in Nexplane. If the cluster's API server certificate changes, the kubeconfig may need to be updated.
- Service account token rotation deletes the existing token Secret and waits for the token controller to create a new one. This can take up to 60 seconds. Applications using the old token will fail during this window.
- Secret rotation updates the secret in-place. Pods that mount the secret as a volume will receive the updated value automatically (subject to kubelet sync delay, typically up to 2 minutes). Pods that use the secret as environment variables must be restarted to pick up the new values.
- The connector does not support exec-based kubeconfig authentication (e.g., `aws eks get-token`). Use a static service account token or client certificate kubeconfig.
