# HashiCorp Vault Connector

The HashiCorp Vault connector uses the `hvac` Python library to interact with Vault secrets engines and auth methods.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Vault URL | Yes | Vault server URL (e.g., `https://vault.acme.com:8200`) |
| Token | Yes | Vault token with appropriate policies (or AppRole credentials) |
| Namespace | No | Vault namespace (Vault Enterprise only) |

## Ingest

Discovers KV v2 secret paths and policies.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| Rotate KV secret | Write a new version of a KV v2 secret | Restore the previous version |
| Revoke token | Revoke a Vault token | N/A |
| Dynamic credentials | Generate short-lived credentials from database secrets engine | N/A |
| Update policy | Add or update a Vault policy | Restore previous policy |

Vault's KV v2 versioning means rollback can restore an exact prior secret version — not just a new value.
