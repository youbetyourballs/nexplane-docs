# Incident Response Change Types

Incident response change types are fast-path actions for active security incidents. They route through an expedited approval path and can be executed by users with the `ir_responder` role even during change freeze windows.

## Account Lockdown

**Change type:** `lockdown_account`

Simultaneously disables a compromised account across all connected identity systems:

- Active Directory
- Okta
- Microsoft Entra ID
- Google Workspace
- GitHub
- Slack

All systems are targeted concurrently. Per-system results are recorded individually — failure in one does not block the others.

**Rollback:** Re-enable the account across all systems that were successfully disabled.

## Phishing Response

**Change type:** `phishing_response`

Responds to a confirmed phishing incident:

1. Block the sender domain at the email gateway
2. Force password reset for all affected users
3. Revoke active sessions across identity providers
4. Force MFA re-enrollment

## Evidence Preservation

**Change type:** `preserve_evidence`

Collects forensic data from a host before any remediation runs:

- Auth logs (`/var/log/auth.log`, `journald`)
- Auditd records
- Active network connections
- ARP cache
- Running process state

All files are packed into a tar.gz and uploaded to S3 via a pre-signed URL.

**Always run evidence preservation before isolation or remediation.** The change is designed to be the first step in any incident response workflow.

**Connector:** Nexplane Agent (`forensics` package)

## Host Isolation

**Change type:** `isolate_host`

Flushes all network rules on the host and replaces them with a default-deny policy that allows only the management CIDR and the Nexplane control plane IP.

**Connector:** Nexplane Agent (`isolation` package)

**Rollback:** `restore_network_access` — restores the pre-isolation network state from the snapshot taken before any rules were flushed.

See [Incident Response](../features/incident-response.md) for the full playbook descriptions and forensic bundle download.
