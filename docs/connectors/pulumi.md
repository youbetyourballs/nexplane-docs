# Pulumi

The Pulumi connector discovers and manages Pulumi stacks via the Pulumi Cloud API.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| API Token | Yes | Pulumi Cloud API token |
| Organization | Yes | Pulumi organization name |

## Ingest

Discovers Pulumi stacks (name, project, last update, resource count), stack resources, and stack update history.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `cancel_update` | Cancel an in-progress stack update | N/A |
| `refresh_stack` | Refresh stack state from the cloud provider | N/A |
