# Windows Update for Business

The Windows Update for Business connector manages WUfB update rings via the Microsoft Graph API.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Tenant ID | Yes | Azure AD tenant ID |
| Client ID | Yes | App registration client ID |
| Client Secret | Yes | App registration client secret |

## Ingest

Discovers update rings/policies with deferral settings and assigned groups.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `create_update_ring` | Create a Windows Update deployment ring with deferral and deadline settings | Delete the ring |
| `delete_update_ring` | Remove a Windows Update deployment ring | N/A |
| `check_device_compliance` | Return update compliance state for devices — which are missing updates and by how much | N/A (read-only) |
