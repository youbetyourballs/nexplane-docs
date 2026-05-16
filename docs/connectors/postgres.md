# PostgreSQL Connector

The PostgreSQL connector uses `psycopg2` to connect to PostgreSQL instances and manage database users, roles, and privileges. It supports password rotation, privilege management, and read-only user creation.

## Credential Fields

| Field | Type | Required | Description |
|---|---|---|---|
| Name | string | Yes | Display name for this connector (e.g., `prod-postgres`) |
| Host | string | Yes | PostgreSQL hostname or IP |
| Port | integer | No | PostgreSQL port (default: 5432) |
| Database | string | Yes | Database name to connect to (typically `postgres` for admin operations) |
| Username | string | Yes | PostgreSQL superuser or admin role |
| Password | string | Yes | Password for the admin account |
| SSL Mode | string | No | `disable`, `require`, `verify-ca`, or `verify-full` (default: `require`) |
| CA Certificate | string | No | PEM-encoded CA certificate for TLS validation |

## Supported Actions

| Action | Description | Rollback |
|---|---|---|
| Rotate User Password | Runs `ALTER USER ... PASSWORD '...'` | No rollback (old password is not stored) |
| Revoke All Privileges | Runs `REVOKE ALL ON ALL TABLES IN SCHEMA ... FROM ...` | Re-grant the revoked privileges |
| Create Read-Only User | Creates a new role with SELECT on all tables in a schema | Drop the created role |
| Lock User Account | Runs `ALTER USER ... NOLOGIN` | `ALTER USER ... LOGIN` |
| Unlock User Account | Runs `ALTER USER ... LOGIN` | `ALTER USER ... NOLOGIN` |
| Drop User | Drops a PostgreSQL role (must have no owned objects) | No rollback |

## Minimum Permissions Required

The admin account must be a superuser or have the following privileges:

- `CREATEROLE` for creating and dropping roles
- `GRANT OPTION` on the privileges being granted/revoked
- Member of `pg_read_all_data` (PostgreSQL 14+) for discovery

For discovery only, a read-only account with `pg_monitor` membership is sufficient.

## Known Limitations

- Password rotation does not affect active database connections. Existing connections remain open with the old credentials until the connection is closed or the session times out.
- Revoking privileges using `REVOKE ALL` revokes permissions granted directly to the user. Privileges inherited through role membership are not affected.
- The connector connects to a single PostgreSQL instance. For multi-instance setups (RDS read replicas, Aurora clusters), create one connector per endpoint.
- `DROP USER` fails if the user owns any database objects. Transfer or drop the owned objects before dropping the user.
- The connector does not support Row-Level Security (RLS) policy management. RLS policies must be managed directly.
