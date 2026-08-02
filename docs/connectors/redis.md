# Redis

The Redis connector manages Redis AUTH passwords.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Host | Yes | Redis server hostname or IP |
| Port | No | Redis port (default: 6379) |
| Current Auth Password | No | Existing requirepass value (required if AUTH is enabled) |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `rotate_redis_password` | Rotate the Redis `requirepass` via `CONFIG SET`. Stores the old password in the execution result for rollback. | Restore previous password |

**Major version upgrades:** `db_major_version_upgrade` supports Redis major version upgrades. See [OS & Application Upgrades](../change-types/os-upgrades.md#database-major-version-upgrade).
