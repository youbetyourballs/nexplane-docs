# GCP Connector

The GCP connector uses the Google Cloud Python client libraries to interact with GCP services. It supports asset discovery across IAM, Compute Engine, and Cloud Storage, and can execute credential rotation and compute operations.

## Credential Fields

| Field | Type | Required | Description |
|---|---|---|---|
| Name | string | Yes | Display name for this connector (e.g., `prod-gcp`) |
| Service Account JSON | string (JSON) | Yes | Full JSON key file for a GCP service account |
| Project ID | string | Yes | GCP project ID (e.g., `my-project-123`) |

The service account JSON is the file you download from GCP when creating a service account key. Paste the entire JSON content into the field.

## Supported Actions

| Action | Description | Rollback |
|---|---|---|
| Rotate Service Account Key | Creates a new key, disables the old one | Re-enable old key, disable new key |
| Delete Service Account Key | Deletes a disabled service account key | No rollback |
| Disable Service Account | Disables a GCP service account | Re-enable the service account |
| Enable Service Account | Re-enables a disabled service account | Disable the service account |
| Modify Firewall Rule | Updates an ingress or egress firewall rule | Restore the previous rule configuration |
| Stop GCE Instance | Stops a running Compute Engine instance | Start the instance |
| Start GCE Instance | Starts a stopped Compute Engine instance | Stop the instance |

## Minimum Permissions Required

Assign the following roles to the service account:

For discovery:
- `roles/iam.securityReviewer`
- `roles/compute.viewer`
- `roles/storage.objectViewer`

For full change execution, additionally:
- `roles/iam.serviceAccountKeyAdmin`
- `roles/compute.instanceAdmin.v1`
- `roles/compute.securityAdmin`

## Known Limitations

- GCP service account keys are global and not region-scoped. The `Project ID` field must match the project where the service account lives.
- Firewall rule modifications apply at the network level and affect all instances in the network that match the rule's target tags or service accounts.
- GCP does not support reactivating a deleted service account key. The rollback for key rotation re-enables a disabled key -- deleted keys cannot be recovered.
- The connector currently supports single-project deployments. Multi-project (organization-level) asset discovery is on the roadmap.
