# Active Directory Operations

Tier-zero Active Directory operations affect domain-wide security posture and require domain admin credentials. All operations are logged and require approval before execution. These change types cover the highest-blast-radius, lowest-frequency AD operations — functional level upgrades, trust relationships, group policy, password policies, stale account cleanup, and domain controller lifecycle management.

## Domain Functional Level Upgrade

**Change type:** `ad_domain_functional_level_upgrade`

Raises the Active Directory domain or forest functional level. Before applying, the executor runs a preflight that inventories all domain controllers and verifies every DC supports the target level. If any DC cannot support the target level, the operation is blocked.

**Phases:**

1. Preflight — inventory all DCs and verify all support the target functional level
2. Raise level — raise domain or forest functional level

!!! warning "Irreversible"
    Raising the forest or domain functional level cannot be undone. Active Directory provides no supported mechanism to lower the functional level once raised. Ensure all domain controllers support the target functional level before proceeding.

**Parameters:** `target_level` (required — e.g. `Win2019`, `Win2025`, `WinThreshold` for 2016), `scope` (`domain` or `forest`, default: `domain`), `domain_name`, `dry_run` (runs preflight DC inventory only, no level change applied)

**Rollback:** Not available.

**Connector:** Active Directory

---

## Forest/Domain Trust

**Change type:** `ad_trust_create`

Creates an Active Directory forest or domain trust relationship and validates Kerberos authentication across the trust boundary. Supports one-way and two-way trusts, and both external and forest trust types.

**Phases:**

1. Preflight — verify DNS resolution and network connectivity between source and target domains
2. Create trust — establish the trust relationship via `New-ADTrust` or `netdom`
3. Validate Kerberos — confirm Kerberos authentication works across the trust boundary

**Parameters:** `source_domain` (required), `target_domain` (required), `trust_type` (`external` or `forest`, default: `external`), `trust_direction` (`one_way_inbound`, `one_way_outbound`, or `two_way`, default: `two_way`), `target_domain_admin_user` (required), `target_domain_admin_password` (required), `dry_run` (connectivity preflight only)

**Rollback:** Remove the trust relationship using `Remove-ADTrust` on both sides of the trust boundary.

**Connector:** Active Directory

---

## Group Policy Deployment

**Change type:** `ad_gpo_deploy`

Creates and links a Group Policy Object with a staged rollout: the policy is linked to a pilot OU first, then extended to production OUs after validation. This staged approach limits blast radius — a misconfigured policy affects only the pilot OU until the operator approves the production link.

**Phases:**

1. Preflight — verify the GPO name is unique and the pilot OU exists
2. Create GPO — create the GPO and apply the specified registry, security, or script settings
3. Link pilot — link the GPO to the pilot OU and wait for policy application
4. Validate pilot — validate the GPO applied correctly on machines in the pilot OU
5. Link production — link the GPO to production OUs after pilot validation passes

**Parameters:** `domain_name` (required), `gpo_name` (required), `gpo_settings_json` (required — JSON describing registry/security/script settings), `pilot_ou` (required — OU distinguished name), `target_ous` (optional array of OU DNs for production rollout), `dry_run`

**Rollback:** Remove all GPO links and delete the GPO using `Remove-GPLink` and `Remove-GPO`.

**Connector:** Active Directory

---

## Fine-Grained Password Policy

**Change type:** `ad_pso_manage`

Creates, updates, or deletes an Active Directory Password Settings Object (PSO) and applies it to security groups or users. For update and delete operations, the executor snapshots the existing PSO state before making changes so rollback can restore the previous configuration exactly.

**Phases:**

1. Preflight — verify the precedence value is unique and target groups/users exist
2. Snapshot existing — capture current PSO state for rollback (update and delete operations only)
3. Apply PSO — create, update, or delete the PSO
4. Apply subjects — apply the PSO to the specified groups and users

**Parameters:** `domain_name` (required), `pso_name` (required), `operation` (`create`, `update`, or `delete`, default: `create`), `precedence` (required — lower values take priority, must be unique), `min_password_length` (default: 14), `password_history_count` (default: 24), `max_password_age_days` (default: 90), `lockout_threshold` (default: 5), `applies_to` (optional array of group or user distinguished names), `dry_run`

**Rollback:** For `create`: remove the PSO. For `update`: restore previous PSO settings from the captured pre-change snapshot. For `delete`: re-create the PSO from the captured pre-change state.

**Connector:** Active Directory

---

## Stale Computer Account Cleanup

**Change type:** `ad_stale_computer_cleanup`

Identifies and remediates stale Active Directory computer accounts using a two-phase lifecycle: accounts inactive beyond the threshold are disabled first (not deleted), and accounts that have been disabled for the retention period are then eligible for deletion. Running with `dry_run` produces a candidate report without making any changes.

**Phases:**

1. Inventory — query AD for stale computer accounts based on `lastLogonTimestamp` and `pwdLastSet`; accounts inactive beyond `inactive_days_threshold` (default: 90 days) are flagged
2. Disable stale — disable computer accounts that are inactive beyond the threshold
3. Delete expired — delete accounts that have been in the disabled state beyond `disabled_retention_days` (default: 30 days)

**Parameters:** `domain_name` (required), `inactive_days_threshold` (default: 90), `disabled_retention_days` (default: 30), `target_ou` (optional OU DN to scope the search), `disable_only` (skip deletion phase, default: false), `dry_run` (report candidates only, no changes made)

**Rollback:** Re-enable all accounts disabled during this run. Deleted accounts cannot be recovered via rollback — use the AD Recycle Bin (if enabled) or backup restore.

**Connector:** Active Directory

---

## Domain Controller Parallel Upgrade

**Change type:** `ad_dc_parallel_upgrade`

Upgrades a domain controller using the Microsoft-prescribed parallel paradigm: provision a new EC2 instance running the target Windows Server version, promote it as a DC, wait for AD replication to converge, transfer FSMO roles, validate the new DC, then demote the old DC. This approach keeps the domain continuously available — there is always a healthy DC online throughout the operation.

The executor provisions the new DC instance directly using the AWS EC2 API, using the same subnet and security groups as the source DC. A Windows Server AMI is resolved from AWS SSM Parameter Store for the specified target version.

**Phases:**

1. Preflight — verify WinRM connectivity to the source DC, confirm the domain is healthy, query current FSMO role holders, check DNS and replication health, and resolve the target Windows Server AMI
2. Provision new DC — launch a new EC2 instance, install the AD Domain Services role, run DCPromo to join the new instance as a domain controller
3. Verify replication — poll `repadmin /showrepl` until replication converges with zero errors (configurable timeout, default: 30 minutes)
4. Transfer FSMO roles — move all five FSMO roles (PDC Emulator, RID Master, Infrastructure Master, Domain Naming Master, Schema Master) to the new DC using `Move-ADDirectoryServerOperationMasterRole`
5. Validate new DC — LDAP probe, DNS resolution, `dcdiag` checks (advertising, fsmocheck, kccevent, services)
6. Demote old DC — run `Uninstall-ADDSDomainController` on the source DC
7. Verify domain health — final `dcdiag` sweep and Directory Service event log check on the new DC

**Parameters:** `new_dc_asset_id` (required), `new_dc_hostname` (required — FQDN or IP), `domain_name` (required), `target_os_version` (optional — e.g. `2025`), `transfer_fsmo` (default: true), `demote_old_dc` (default: true), `dry_run` (runs preflight only)

**Rollback:** Three-case rollback based on how far the operation progressed:

- **Before FSMO transfer:** Demote and terminate the new DC. Source DC remains authoritative and no state change has occurred.
- **After FSMO transfer, before source demotion:** Seize FSMO roles back to the source DC using `ntdsutil`, then demote and terminate the new DC.
- **After source demotion:** Cannot be automatically reversed. The execution result records the source EC2 instance ID so an operator can re-promote it manually or restore from an AMI snapshot.

**Connector:** Active Directory (WinRM) + AWS (EC2 provisioning)
