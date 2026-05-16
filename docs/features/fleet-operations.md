# Fleet Operations

Fleet operations let you coordinate changes across many hosts safely — with batch control, abort thresholds, health checks between batches, and rollback on failure.

## Rolling Restart

Restart a service across a fleet N hosts at a time:

- Configure `batch_size` (how many hosts per batch) and `abort_threshold` (maximum failure rate before halting)
- A health check command runs between batches — if it fails, the restart stops
- If the failure rate across any batch exceeds the abort threshold, the campaign halts and triggers rollback on completed hosts

## Canary Config Push

Deploy a config change to one host first, verify it, then roll to the rest:

1. Push config to the canary host
2. Run a verification command against the canary
3. If the verification passes, roll config to all remaining hosts
4. If verification fails, rollback restores the original file on the canary host

The original file is backed up before being replaced. Rollback restores it exactly.

## Bulk File Distribution

Push files to a fleet in a single change request:

- CA certificates
- `authorized_keys` files
- Config files
- Scripts

Files are distributed to all target hosts via the Nexplane Agent. Each host reports success/failure independently.

## Fleet Health Check

Before major operations (patch campaigns, rolling restarts), run a health check across all target hosts:

- Disk usage
- System load
- Named service status
- Pending-reboot flag (Windows: Windows Update pending reboot; Linux: `/var/run/reboot-required`)

Hosts failing the health check are flagged before the operation begins. You can abort or proceed with only healthy hosts.

## Maintenance Windows

Define cron-scheduled windows during which approved change requests are allowed to execute:

- Approved CRs arriving outside a maintenance window are queued automatically
- When the window opens, queued CRs execute in order
- Emergency CRs with the `ir_responder` role bypass window restrictions

Configure maintenance windows in **Settings → Maintenance Windows** or via `POST /maintenance-windows/`.
