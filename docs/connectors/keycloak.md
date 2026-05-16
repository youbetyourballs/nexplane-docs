# Keycloak

The Keycloak connector manages user accounts in Keycloak via the Admin REST API.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| (Connection fields configured per-deployment) | | |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `keycloak_disable_user` | Disable a Keycloak user account and revoke all active sessions | Re-enable the account |
