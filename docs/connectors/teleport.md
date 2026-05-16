# Teleport CE

The Teleport connector manages user access via the `tctl` CLI on self-hosted Teleport CE clusters.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Teleport Proxy Address | Yes | e.g. `teleport.example.com:3025` |
| Auth Token | No | Auth token for `tctl` |
| Path to tctl binary | No | Path to tctl (default: resolved from PATH) |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `teleport_lock_user` | Create a Teleport lock for a user via `tctl`, preventing all access for the specified TTL (default: 1h) | Delete the lock |
