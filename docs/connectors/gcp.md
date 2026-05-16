# GCP Connector

The GCP connector uses Google Cloud Python client libraries to interact with Google Cloud services.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name (e.g., `prod-gcp`) |
| Service Account JSON | Yes | Full JSON key file for a GCP service account |
| Project ID | Yes | GCP project ID |

## Required IAM Roles

Grant the service account these roles on the project:

- `roles/compute.admin`
- `roles/iam.serviceAccountAdmin`
- `roles/iam.serviceAccountKeyAdmin`
- `roles/storage.admin`
- `roles/dns.admin`
- `roles/iam.securityAdmin` — required for IAM binding operations

## Capabilities

### Compute Engine

| Action | Description | Rollback |
|--------|-------------|---------|
| `gcp_instance_create` | Create a Compute instance | Delete the instance |
| `gcp_instance_stop` | Stop a running instance | Start the instance |
| `gcp_instance_start` | Start a stopped instance | Stop the instance |
| `gcp_instance_reboot` | Reboot an instance | N/A |
| `gcp_instance_snapshot` | Create a persistent disk snapshot | Delete the snapshot |
| `gcp_instance_delete` | Delete an instance | Not available |

### Firewall

| Action | Description | Rollback |
|--------|-------------|---------|
| `gcp_firewall_create` | Create a firewall rule | Delete the rule |
| `gcp_firewall_delete` | Delete a firewall rule | Recreate the rule |

### Storage

| Action | Description |
|--------|-------------|
| Block public access | Configure public access prevention on a bucket |

### IAM & Service Accounts

| Action | Description | Rollback |
|--------|-------------|---------|
| Disable service account | Disable a service account | Re-enable the service account |
| Rotate service account key | Create new key, delete old key | Not available — GCP does not support re-creating deleted keys |
| IAM binding | Add or remove project-level IAM bindings | Reverse the binding |

### Security Command Center

| Action | Description |
|--------|-------------|
| SCC findings ingest | Import Security Command Center findings as vulnerability findings |

### IaC

| Action | Description |
|--------|-------------|
| Terraform local | Run `terraform apply` in backend container |
| Ansible local | Run Ansible playbook via local connection to GCP instances |
