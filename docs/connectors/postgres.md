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

!!! note
    For full database administration (user provisioning, permission grants, audit logging), the [Nexplane Agent](nexplane-agent.md) `dbadmin` package supports PostgreSQL on managed hosts.
