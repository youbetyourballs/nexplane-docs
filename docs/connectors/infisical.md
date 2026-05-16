# Infisical

The Infisical connector manages secrets in Infisical workspaces.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Infisical Base URL | Yes | e.g. `https://app.infisical.com` |
| API Token | Yes | Infisical API token |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `rotate_secret` | Generate a new value for a secret in an Infisical workspace. Stores the old value for rollback. Pass `new_value` explicitly or omit to auto-generate. | Restore previous secret value |
