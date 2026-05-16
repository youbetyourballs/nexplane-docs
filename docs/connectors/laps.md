# Microsoft LAPS

The Microsoft LAPS connector retrieves and rotates LAPS-managed local administrator passwords via the Microsoft Graph API.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Tenant ID | Yes | Azure AD tenant ID |
| Client ID | Yes | App registration client ID |
| Client Secret | Yes | App registration client secret |

## Ingest

Discovers all devices with LAPS enabled and their last password rotation timestamp.

## Capabilities

| Action | Description |
|--------|-------------|
| `get_local_password` | Retrieve the current LAPS-managed local administrator password for a device. Password is returned in the CR result — treat this CR as sensitive. |
| `rotate_local_password` | Force an immediate LAPS password rotation on a device. The previous password is immediately invalidated. |
