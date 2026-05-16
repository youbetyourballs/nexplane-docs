# Backup & Recovery Change Types

Backup and recovery change types create, verify, and restore backups, and orchestrate DR failover.

## Backup

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `create_backup` | Create a restic backup to S3 (Linux/Windows via agent) or an EBS/RDS snapshot (AWS connector) | Delete the backup |
| `verify_backup` | Restore an RDS snapshot to a temporary instance, run a health query, then terminate the temp instance | N/A |
| `restore_files` | Restic restore with path filter and SHA256 checksum verification | N/A |

**Connector:** Nexplane Agent (`backup` package) for host-level backups; AWS connector for EBS/RDS snapshots.

## DR Failover

**Change type:** `dr_failover`

Updates Route53 weighted routing to shift traffic to the DR site. Records per-step RTO timestamps and reports actual vs target RTO in the execution results.

**Connector:** AWS

**Rollback:** Shift traffic back to the primary site by restoring original Route53 weights.

## Scheduled Reboot

| Change Type | Description |
|-------------|-------------|
| `scheduled_reboot` | Gracefully reboot a host at a scheduled time within a maintenance window |

**Connector:** Nexplane Agent (`reboot` package)

**Commands:**
- `graceful_reboot` — drain connections, notify services, reboot
- `verify_post_reboot` — check named services are healthy after reboot

Scheduled reboots respect [maintenance windows](../features/fleet-operations.md#maintenance-windows) — the CR is queued if submitted outside a window and executes automatically when the next window opens.
