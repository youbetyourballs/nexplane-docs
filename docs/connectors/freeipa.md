# FreeIPA

The FreeIPA connector manages user accounts via the FreeIPA JSON-RPC API.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| FreeIPA Server URL | Yes | e.g. `https://ipa.example.com` |
| Admin Username | Yes | FreeIPA admin username |
| Admin Password | Yes | FreeIPA admin password |
| Verify SSL | No | Verify SSL certificate |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `freeipa_disable_user` | Disable a FreeIPA user account | Re-enable the account |
