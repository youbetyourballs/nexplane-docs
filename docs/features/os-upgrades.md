# OS & Application Upgrades

Nexplane orchestrates major version upgrades as audited, approval-gated change requests. The core guarantee: the platform takes a snapshot or preserves the old instance before making any irreversible change, and the rollback path is tested before the upgrade is declared complete.

## Why Managed Upgrades

Unmanaged upgrades fail silently. An in-place upgrade that leaves the OS half-migrated, or a new instance that came up missing a scheduled task, typically isn't discovered until something breaks in production. Nexplane enforces:

- **Pre-flight validation** before any change is made
- **State capture** (EBS snapshot or preserved instance) before cutting over
- **Health verification** before completing — failed verification triggers automatic rollback
- **24-hour rollback window** for parallel upgrades (old instance stopped, not terminated)

## Upgrade Methods

### In-Place (Linux)

`os_upgrade` runs the distribution's standard upgrade tool (`do-release-upgrade` on Ubuntu) on the existing host. Suitable for dev/staging. For production, prefer the parallel approach.

Rollback: EBS root volume swap from the pre-upgrade snapshot.

### Parallel Upgrade (Linux)

`linux_parallel_upgrade` provisions a new instance at the target OS version, transfers state, and performs an EIP cutover. The old instance is **stopped but not terminated** and remains available for rollback for 24 hours.

This is the preferred method for production Linux hosts because:

- The old instance is preserved intact and can be started immediately on rollback
- Cutover is a single EIP reassignment — fast and reversible
- The new instance can be fully validated before the cutover

### Parallel Migration (Windows)

`windows_parallel_migration` handles Windows hosts where in-place upgrade is too risky. It inventories the source host comprehensively before provisioning a new instance:

- Windows services and their startup configuration
- Scheduled tasks
- IIS sites, application pools, and bindings
- Environment variables
- Installed certificates
- Application configuration files
- Registry keys
- DNS records referencing the old hostname
- Hardcoded hostname references in config files

Each artifact is transferred to the new instance. DNS cutover completes the migration.

Rollback: revert DNS to point back to the old instance (kept running during migration).

## Kubernetes Cluster Upgrade

`k8s_cluster_upgrade` upgrades EKS, GKE, AKS, or kubeadm clusters. The platform enforces a maximum of +1 minor version per upgrade (e.g., 1.28 → 1.29, not 1.28 → 1.31) and scans for removed API usage before starting.

!!! warning "Control plane partial irreversibility"
    Once the control plane etcd data migrates to the new version format, the control plane cannot be downgraded. Node pools can be rolled back independently.

## Database Major Version Upgrades

`db_major_version_upgrade` handles PostgreSQL, MySQL, and MongoDB major version upgrades. A compatibility scan runs before the upgrade starts. Rollback uses `pg_dump`/`mysqldump`/`mongodump` restore from a pre-upgrade dump, with an optional EBS snapshot as a faster fallback.

## See Also

- [OS & Application Upgrades](../change-types/os-upgrades.md) — full CR type reference with parameters
