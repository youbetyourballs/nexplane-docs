# PostgreSQL

The PostgreSQL connector connects directly to PostgreSQL instances for credential rotation.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Host | Yes | PostgreSQL server hostname or IP |
| Port | No | PostgreSQL port (default: 5432) |
| Database Name | No | Database to connect to (default: postgres) |
| Admin Username | Yes | Admin user with `ALTER USER` privileges |
| Admin Password | Yes | Admin password |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `rotate_postgres_password` | Generate a secure random password and set it on the specified PostgreSQL user via `ALTER USER` | Restore previous password |

**Major version upgrades:** `db_major_version_upgrade` supports PostgreSQL major version upgrades with pre-upgrade compatibility scan and `pg_dump`-based rollback. See [OS & Application Upgrades](../change-types/os-upgrades.md#database-major-version-upgrade).

!!! note
    For full database administration (user provisioning, permission grants, audit logging), the [Nexplane Agent](nexplane-agent.md) `dbadmin` package supports PostgreSQL on managed hosts.
