# OS & Application Upgrades

OS and application upgrade change types orchestrate version transitions for operating systems, runtimes, databases, and Kubernetes clusters. Each type enforces a pre-flight gate, takes a snapshot or backup before making changes, and provides a structured rollback path. For single-instance in-place upgrades, rollback restores from the snapshot taken before the upgrade began. For parallel migration types, rollback reverses the traffic cutover so the source instance resumes serving while the destination is discarded.

---

## OS Major Version Upgrade

**Change type:** `os_upgrade`

Upgrades a Linux host to a new major OS version in-place. The executor blocks on a pre-flight check that validates the target version path and catalogues running services, takes an EBS snapshot (mandatory on production assets), runs the distribution upgrade, then verifies that services listed in the pre-flight report are still running. If the upgrade fails at any point, the executor automatically restores from the snapshot without requiring operator intervention.

**Phases:**

1. **Preflight** — agent checks that the target version is a valid upgrade path, lists running services, and estimates migration time. Blocked if the path is unsupported.
2. **Snapshot** — EBS snapshot of the instance is taken. Mandatory on production assets; can be skipped on non-production with `skip_snapshot: true`.
3. **Upgrade** — distribution upgrade executes on the host (e.g. `do-release-upgrade` on Ubuntu, `dnf system-upgrade` on RHEL/CentOS). Timeout is 2 hours.
4. **Verify** — agent confirms the OS version matches the target and that the services recorded in preflight are responding. Auto-rollback from snapshot fires if verification fails.

**Rollback:** Full — snapshot is restored automatically on upgrade failure; operator-initiated rollback also restores from the same snapshot.

**Connector:** Nexplane Agent

---

## Linux Parallel Upgrade

**Change type:** `linux_parallel_upgrade`

Migrates a Linux instance to a pre-provisioned destination instance at a higher OS version using rsync, then cuts over traffic. The source instance is not modified — it continues running until decommission. Because the source is kept intact until after cutover succeeds, rollback is a traffic reversal with no data loss. Rollback capability becomes irreversible only after the source decommission job fires (default: 24 hours post-cutover).

**Phases:**

1. **Preflight** — verifies that both source and destination agents are reachable, that `sync_paths` exist on the source, and that cutover preconditions are met (EIP allocation, ALB target group ARN, DNS zone, or static IP config).
2. **Snapshot** — EBS snapshot of the source is taken if running on EC2. On-premises hosts skip snapshot with a logged warning.
3. **Pre-sync** — rsync runs one or more times (configurable via `pre_sync_runs`) to drain the delta between source and destination before the maintenance window.
4. **Verify dest health** — agent confirms the destination is healthy (TCP ports, optional health check command).
5. **Cutover** — final rsync run, then traffic is swapped: EIP re-association, ALB target group swap, DNS record update, or static IP change. Source is stopped.
6. **Decommission scheduling** — source is held for `decommission_after_hours` (default 24) before termination, preserving the rollback window.

**Rollback:** Full while source is still running — reverses the traffic cutover and restarts the source. Becomes irreversible after source decommission fires.

**Connector:** Nexplane Agent

---

## Windows Parallel Migration

**Change type:** `windows_parallel_migration`

Migrates a Windows Server instance to a new instance at a higher OS version. Inventory is captured from the source (installed software, services, scheduled tasks, registry keys), data is transferred via robocopy, and then traffic is cut over via EIP, DNS, or ENI reassignment. The source is stopped but not terminated for 24 hours, giving a clean rollback window. Windows features must be pre-installed on the destination — the executor handles data and configuration, not feature installation.

**Phases:**

1. **Preflight** — verifies agents on both instances are reachable, confirms the OS upgrade path is supported, validates cutover method preconditions, and reads the current DNS TTL if DNS cutover is selected (so it can be restored on rollback).
2. **Snapshot** — EBS snapshot of the source is taken if running on EC2. Skipped with a warning for on-premises hosts.
3. **Inventory** — `win_inventory` runs on the source to collect installed software, services, scheduled tasks, and the specified registry keys. Hostname references are recorded for post-sync replacement.
4. **Sync** — robocopy transfers data and configuration from source to destination. Hostname string replacements are applied on the destination after sync.
5. **Health check** — verifies the destination agent responds, confirms robocopy succeeded, and validates hostname replacements were applied.
6. **Cutover** — traffic is swapped (EIP re-association, DNS update, or ENI re-attachment). Source is stopped. Decommission is scheduled for `decommission_after_hours` (default 24).

**Rollback:** Full while source is still running — reverses the cutover (re-associates EIP, restores DNS, or re-attaches ENI), starts the source, and verifies agent responds within 300 seconds. Becomes irreversible after source decommission fires.

**Connector:** Nexplane Agent

---

## Kubernetes Cluster Upgrade

**Change type:** `k8s_cluster_upgrade`

Upgrades a Kubernetes cluster by exactly one minor version. Supports EKS, GKE, AKS, and kubeadm clusters (auto-detected by default). The executor enforces the +1 minor version constraint and scans for removed API violations before the irreversible control-plane step. Node pools are upgraded in rolling or blue-green fashion and can be independently rolled back to their previous image.

**Phases:**

1. **Preflight** — validates the target version is exactly one minor ahead of current, scans all workloads for removed API violations, checks node resource headroom, and detects the cluster type. Blocked if removed API violations are found or version path is invalid.
2. **Control plane upgrade** — upgrades the Kubernetes control plane via the managed API (EKS, GKE, AKS) or `kubeadm`. Timeout is 30 minutes. **This step is irreversible** — Kubernetes does not support control plane downgrade.
3. **Node pool rolling upgrade** — each node pool is upgraded in sequence: nodes are drained (respecting PodDisruptionBudgets), the node image is replaced, and the node is verified healthy before proceeding. `max_unavailable` controls concurrency (default: 1). Timeout per drain is controlled by `drain_timeout_seconds` (default: 300).
4. **Post-upgrade health check** — cluster-level health check confirms API server, node readiness, and core workloads are healthy after all node pools have been upgraded.

**Rollback:** Partial — node pools are restored to their previous image in FILO order. The control plane upgrade is irreversible; if the failure occurs during the control plane step, manual intervention is required.

**Connector:** Nexplane Agent

---

## Database Major Version Upgrade

**Change type:** `db_major_version_upgrade`

Upgrades a database engine to a new major version. Supports PostgreSQL (dump-restore or in-place), MySQL (in-place), and MongoDB (sequential feature compatibility version bumps). A snapshot or dump is taken before any changes, providing a full rollback artifact. On RDS, the executor uses the managed upgrade API; on self-hosted instances, it connects via the Nexplane Agent. Skipping the snapshot is blocked on production-tagged assets.

**Phases:**

1. **Preflight** — verifies agent reachability, confirms the source and target versions are compatible, and checks for blocking conditions (active connections, replication lag, unsupported extensions). Returns a `preflight_blocked` status if any check fails.
2. **Snapshot** — for RDS instances, an RDS snapshot is taken; for self-hosted PostgreSQL/MySQL, a dump is written to S3; for MongoDB, a filesystem snapshot is taken. `skip_snapshot` is rejected on production assets.
3. **Upgrade** — engine-specific upgrade runs: PostgreSQL uses `pg_upgrade` (in-place) or dump-and-restore to a new instance; MySQL runs `mysql_upgrade`; MongoDB advances through each feature compatibility version hop in the chain sequentially, one major version at a time.
4. **Verify** — connects to the upgraded database, checks the reported version matches the target, runs the optional `health_check_url` probe, and confirms the database accepts queries.

**Rollback:** Full — restores from the snapshot or dump taken in phase 2. RDS restores from the managed snapshot; self-hosted instances restore from the S3 dump. MongoDB multi-hop upgrades restore from the initial pre-upgrade snapshot.

**Connector:** Nexplane Agent
