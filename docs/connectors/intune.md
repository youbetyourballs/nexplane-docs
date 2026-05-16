# Microsoft Intune

The Microsoft Intune connector discovers managed devices and manages device policies via the Microsoft Graph API.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Tenant ID | Yes | Azure AD tenant ID |
| Client ID | Yes | App registration client ID |
| Client Secret | Yes | App registration client secret |

## Ingest

Discovers all Intune-managed devices with compliance state, OS version, and last sync timestamp.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `sync_device` | Trigger an immediate MDM sync on a device to pull latest policy | N/A (read-only trigger) |
| `push_script` | Deploy a PowerShell script to a device or group via Intune | Delete the deployed script |
| `delete_script` | Remove a previously deployed Intune script | N/A |
| `check_compliance` | Return current compliance state and failing policies for a device | N/A (read-only) |
