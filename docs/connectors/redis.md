# Redis Connector

The Redis connector uses `redis-py` to connect to Redis instances. It supports AUTH password rotation and selective key management for security remediation scenarios.

## Credential Fields

| Field | Type | Required | Description |
|---|---|---|---|
| Name | string | Yes | Display name for this connector (e.g., `prod-redis`) |
| Host | string | Yes | Redis hostname or IP |
| Port | integer | No | Redis port (default: 6379) |
| Password | string | No | Redis AUTH password (leave blank if AUTH is not configured) |
| Database | integer | No | Redis database index (default: 0) |
| Use TLS | boolean | No | Connect using TLS (default: false) |
| CA Certificate | string | No | PEM-encoded CA certificate for TLS validation |

## Supported Actions

| Action | Description | Rollback |
|---|---|---|
| Rotate AUTH Password | Sets a new Redis AUTH password via `CONFIG SET requirepass` | Restore the old password (stored encrypted in the change record) |
| Flush Volatile Keys | Deletes all keys with a TTL (volatile keys only, not persistent keys) | No rollback (deleted keys cannot be recovered) |
| Delete Key Pattern | Deletes keys matching a specific pattern | No rollback |

## Minimum Permissions Required

The Redis AUTH password must belong to an account with:

- `CONFIG` command access (for password rotation)
- `KEYS` or `SCAN` command access (for discovery and key operations)

For Redis 6+ with ACL support:

```
ACL SETUSER nexplane on ><password> ~* +CONFIG +KEYS +SCAN +TTL +DEL +INFO
```

## Known Limitations

- Redis AUTH is instance-wide. Rotating the AUTH password immediately disconnects all clients using the old password. Coordinate password rotation with application deployments.
- Redis 6+ ACL-based users are not yet supported as distinct identities. The connector treats the AUTH password as a single credential for the instance.
- `Flush Volatile Keys` uses `SCAN` to iterate keys and check TTL. On large keyspaces, this can take significant time and impact Redis performance. Use during low-traffic windows.
- The connector connects to a single Redis instance. Redis Cluster and Redis Sentinel are not currently supported.
- There is no discovery for Redis -- the connector does not enumerate keys. Key information is provided by the user when creating a change request.
