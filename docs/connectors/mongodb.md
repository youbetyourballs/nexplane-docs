# MongoDB Connector

The MongoDB connector uses `pymongo` to connect to MongoDB instances. It supports user password rotation, role management, and read-only user creation.

## Credential Fields

| Field | Type | Required | Description |
|---|---|---|---|
| Name | string | Yes | Display name for this connector (e.g., `prod-mongo`) |
| Connection String | string | Yes | MongoDB connection string (e.g., `mongodb://admin:password@host:27017/admin?authSource=admin`) |
| TLS CA Certificate | string | No | PEM-encoded CA certificate for TLS validation |

!!! tip "Atlas connections"
    For MongoDB Atlas, use the connection string from the Atlas UI. Include `&tls=true` and provide the Atlas CA certificate if using a private endpoint.

## Supported Actions

| Action | Description | Rollback |
|---|---|---|
| Rotate User Password | Updates the user's password in the `admin` database | No rollback (old password is not stored) |
| Revoke Role from User | Removes a role from a MongoDB user | Re-grant the role |
| Grant Role to User | Grants a role to a MongoDB user | Revoke the role |
| Create Read-Only User | Creates a new user with `read` role on a specific database | Drop the created user |
| Drop User | Drops a MongoDB user | No rollback |
| Lock User | Disables a MongoDB user account (4.4+) | Enable the user |

## Minimum Permissions Required

The admin account must have:

- `userAdmin` or `userAdminAnyDatabase` role for user management
- `clusterMonitor` role for discovery

For Atlas:
- Project Owner or a custom role with user management permissions

## Known Limitations

- Password rotation does not close active connections. Existing connections remain authenticated until they close or are killed with `db.killOp()`.
- User locking (`disableLockedUser`) requires MongoDB 4.4 or later. Earlier versions do not support this operation.
- The connector connects to a single MongoDB instance or replica set primary. MongoDB sharded clusters are supported via the `mongos` router address.
- Role management operates on roles defined in the `admin` database. Database-local roles (defined in a specific database) are not currently supported.
- Atlas Data API integration is on the roadmap. Currently the connector uses the MongoDB wire protocol directly.
