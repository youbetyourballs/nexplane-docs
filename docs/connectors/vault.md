# HashiCorp Vault Connector

The Vault connector uses the `hvac` Python library to interact with HashiCorp Vault. It supports secret rotation, token revocation, and policy management for KV v2, database secrets engines, and token auth methods.

## Credential Fields

| Field | Type | Required | Description |
|---|---|---|---|
| Name | string | Yes | Display name for this connector (e.g., `prod-vault`) |
| Vault Address | string | Yes | Vault server URL (e.g., `https://vault.internal:8200`) |
| Token | string | Yes | Vault token with appropriate policies |
| CA Certificate | string | No | PEM-encoded CA certificate for TLS validation |
| Namespace | string | No | Vault namespace (Vault Enterprise only) |

## Supported Actions

| Action | Description | Rollback |
|---|---|---|
| Rotate KV Secret | Writes a new version of a KV v2 secret | Restore the previous version |
| Delete KV Secret Version | Soft-deletes a specific secret version | Undelete the version |
| Revoke Token | Revokes a Vault token by accessor | No rollback (revoked tokens cannot be restored) |
| Rotate Database Credentials | Triggers Vault to rotate dynamic database credentials | No rollback (Vault manages the credential lifecycle) |
| Update Policy | Replaces a Vault policy document | Restore the previous policy document |

## Minimum Permissions Required

The Vault token used by Nexplane must have a policy that grants:

For discovery:
```hcl
path "secret/metadata/*" {
  capabilities = ["list", "read"]
}

path "auth/token/lookup-self" {
  capabilities = ["read"]
}

path "sys/policies/acl" {
  capabilities = ["list"]
}
```

For KV secret rotation:
```hcl
path "secret/data/*" {
  capabilities = ["read", "create", "update"]
}

path "secret/metadata/*" {
  capabilities = ["read", "list"]
}
```

For token revocation:
```hcl
path "auth/token/revoke-accessor" {
  capabilities = ["update"]
}

path "auth/token/accessors" {
  capabilities = ["list"]
}
```

## Known Limitations

- The connector supports KV v2 only. KV v1 is not supported due to its lack of versioning (which is required for rollback).
- Token revocation is irreversible. Nexplane does not offer rollback for token revoke operations.
- Dynamic secrets (database, AWS, PKI) cannot be rotated on demand through Nexplane -- Nexplane triggers Vault's built-in rotation endpoint. The resulting credential is managed by Vault's lease system.
- Vault Enterprise namespaces are supported by setting the Namespace field, but cross-namespace operations are not supported.
- The Nexplane Vault token should have a TTL long enough to cover the connector's usage. Nexplane does not automatically renew the token -- set up token renewal separately or use a long-lived token with an appropriate policy.
