# Database Change Types

Database change types manage users, permissions, and configuration across PostgreSQL, MySQL, MSSQL (via the Nexplane Agent), and RDS (via the AWS connector).

## User Provisioning

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `provision_db_user` | Create a database user with scoped grants | Drop the user |
| `deprovision_db_user` | Remove a database user and revoke all grants | Not available |

**Supported databases:** PostgreSQL, MySQL, MSSQL

**Connector:** Nexplane Agent (`dbadmin` package)

## Permissions

| Change Type | Description |
|-------------|-------------|
| `db_permission_change` | Grant or revoke specific permissions for a database user |
| `configure_db_audit` | Configure database audit logging (pg_audit for PostgreSQL, general_log for MySQL, SQL Audit for MSSQL) |
| `db_connection_config` | Update connection limits and timeout settings |

## RDS (AWS)

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `rds_instance_create` | Create an RDS instance | Delete the instance |
| `rds_instance_delete` | Delete an RDS instance | Not available |
| `rds_snapshot_create` | Create an RDS snapshot | Delete the snapshot |

**Connector:** AWS

## DB Replica Promotion (AWS)

**Change type:** `promote_db_replica`

Promotes an RDS read replica to a standalone instance and updates a Route53 CNAME to point to the new primary.

!!! warning "No rollback"
    `promote_db_replica` is marked `rollback_supported: false`. A blast radius warning is required at approval. The change cannot be reversed — the replica becomes an independent instance and the original replication relationship is broken.

**Connector:** AWS

See [Credential Rotation](../features/credential-rotation.md) for `rotate_db_credentials`.
