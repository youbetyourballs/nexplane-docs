# Kubernetes Connector

The Kubernetes connector uses the official Python Kubernetes client to interact with clusters.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Kubeconfig | Yes | kubeconfig YAML (paste the full kubeconfig content) |
| Context | No | Kubernetes context to use (defaults to the current context in the kubeconfig) |

## Ingest

Discovers pods, deployments, services, namespaces, and secrets metadata.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `restart_deployment` | Restart all pods in a deployment (rolling restart) | N/A |
| `scale_deployment` | Scale a deployment up or down | Restore previous replica count |
| `apply_network_policy` | Apply a Kubernetes NetworkPolicy | Delete the policy |
| `update_rbac` | Update a RoleBinding or ClusterRoleBinding | Restore previous binding |
| `rotate_secret` | Update a Kubernetes Secret value | Restore previous secret value |
| `helm_upgrade` | Upgrade a Helm release | `helm rollback` |
| `helm_rollback` | Roll back a Helm release to a previous revision | N/A |
| `k8s_cluster_upgrade` | Major version upgrade for EKS, GKE, AKS, and self-managed kubeadm clusters. Enforces +1 minor version maximum per upgrade. Scans for removed API usage before starting. | Node pools can be rolled back independently; control plane upgrade is partially irreversible after etcd migration. |
