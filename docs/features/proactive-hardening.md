# Proactive Hardening

Proactive hardening is Nexplane's scheduled, recurring baseline enforcement layer. Unlike reactive operations triggered by incidents or alerts, proactive hardening runs on a defined schedule to detect and remediate configuration drift before it becomes a risk. Each action produces a change request that follows the normal lifecycle — plan, approve, execute — so every enforcement action is auditable and reversible.

## SSH CA Rotation

Nexplane manages SSH certificate authority rotation as a seven-phase fleet-wide operation. When an SSH CA rotation CR executes, the platform generates a new CA keypair, distributes the new trusted CA public key to every host in scope, verifies each host accepted it, then revokes the old CA.

**Phases:**

1. Preflight — enumerate all hosts in scope, verify SSH connectivity, check the current CA key and `TrustedUserCAKeys` configuration
2. Snapshot — capture the current `TrustedUserCAKeys` file on every host in the execution context
3. Generate — generate a new CA keypair (default: `ed25519`) on the designated CA host
4. Distribute — push the new CA public key to `TrustedUserCAKeys` on every host; both old and new CAs are trusted simultaneously during this window
5. Verify — confirm every host reports the new CA public key in its `TrustedUserCAKeys` file
6. Revoke — remove the old CA public key from `TrustedUserCAKeys` on every host
7. Report — emit per-host results and record the new CA key fingerprint in the execution result

**Rollback:** Restores the original `TrustedUserCAKeys` file on all hosts from the snapshots captured in phase 2.

See [ssh_ca_rotation](../change-types/proactive-hardening.md#ssh-ca-rotation) for the full parameter reference.

## AWS IAM Role Baseline

The IAM role baseline change type scans your AWS account for stale and overprivileged roles. Stale is defined as roles with no recorded usage in the last N days (configurable, default 90). Overprivileged is flagged based on wildcard permissions on sensitive service namespaces.

The scan produces a findings list. When `generate_remediations` is enabled (default), each finding is accompanied by a proposed remediation CR — a targeted `iam_role_remediate` change request that the operator can review and approve. The audit CR itself makes no changes to IAM.

See [aws_iam_role_baseline](../change-types/proactive-hardening.md#aws-iam-role-baseline) for the full parameter reference.

## Privileged Account Audit

The privileged account audit queries Active Directory for members of Domain Admins, Enterprise Admins, and Schema Admins. Each account is evaluated for:

- MFA registration status
- Days since last interactive logon
- Whether the account is flagged as a service account with admin rights

Accounts that fail any check are included in the findings list. When `generate_remediations` is enabled (default), the platform produces a batch of proposed remediation CRs — account disables, MFA enforcement actions, or group membership removals — that the operator reviews and approves before execution.

See [privileged_account_audit](../change-types/proactive-hardening.md#privileged-account-audit) for the full parameter reference.

## Scheduling Proactive Hardening

All three actions can be scheduled to run on a recurring basis using Recurring Jobs. A typical configuration runs SSH CA rotation monthly, IAM baseline scans nightly, and privileged account audits weekly.

See [Recurring Jobs](recurring-jobs.md) for configuration details.
