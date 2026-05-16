# Chef InSpec

The Chef InSpec connector integrates with Chef Automate to run compliance scans and manage compliance profiles.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Chef Automate URL | Yes | e.g. `https://automate.example.com` |
| API Token | Yes | Chef Automate API token |

## Ingest

Discovers Chef-managed nodes, available InSpec compliance profiles, and latest compliance scan results per node.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `run_compliance_scan` | Trigger a compliance scan on a node against a specific profile | N/A |
| `assign_profile` | Assign a compliance profile to a node | Unassign the profile |
