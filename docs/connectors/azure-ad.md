# Azure AD (Graph API)

The Azure AD connector provides direct Microsoft Graph API access for core user lifecycle operations. For the full Entra ID feature set (sessions, licenses, Teams, conditional access discovery), see the [Microsoft Entra ID connector](entra-id.md).

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Tenant ID | Yes | Azure AD tenant ID |
| Client ID | Yes | App registration client ID |
| Client Secret | Yes | App registration client secret |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `disable_user` | Set `accountEnabled=false` and revoke all active sessions (by UPN or object ID) | Enable the user |
| `create_user` | Provision a new Azure AD user via Microsoft Graph | Delete the user |
