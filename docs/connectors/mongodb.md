# MongoDB

The MongoDB connector manages MongoDB user credentials.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Connection URI | Yes | MongoDB connection URI (e.g. `mongodb://admin:password@localhost:27017`) |
| Auth Database | No | Authentication database (default: admin) |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `rotate_mongodb_password` | Rotate a MongoDB user's password via the `updateUser` command | Restore previous password |

**Major version upgrades:** `db_major_version_upgrade` supports MongoDB major version upgrades with `mongodump`-based rollback. See [OS & Application Upgrades](../change-types/os-upgrades.md#database-major-version-upgrade).
