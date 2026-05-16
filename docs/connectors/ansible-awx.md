# Ansible (AWX)

The Ansible AWX connector integrates with AWX (open-source Ansible Tower) to discover managed infrastructure and launch job templates.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Controller URL | Yes | AWX URL (e.g. `https://awx.example.com`) |
| API Token | Yes | AWX API token |

## Ingest

Discovers inventories, managed hosts, job templates, and recent job run history.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `launch_job` | Launch a job template by ID, with optional extra vars | Cancel the job |
| `cancel_job` | Cancel a running job | N/A |
| `run_adhoc_command` | Run an ad-hoc Ansible module across hosts in an inventory | N/A |
| `sync_inventory` | Trigger an inventory sync | N/A |
| `update_host_variables` | Update Ansible variables for a specific host | Restore previous variables |

!!! note
    For running playbooks without an AWX server, see [Ansible (Local)](ansible-local.md) (SSM transport) or [Ansible (Remote)](ansible-remote.md) (direct SSH).
