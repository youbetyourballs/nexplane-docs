# Incident Response

Nexplane includes pre-built, fast-path change workflows for common incident types. IR playbooks are launched from a single click and run through an expedited approval path.

## Playbooks

### Host Isolation

Cuts off all network access to a compromised host while preserving management connectivity:

- Flushes iptables/nftables (Linux) or Windows Firewall rules
- Allows only the management CIDR and the Nexplane control plane
- Records the pre-isolation network state for rollback
- Dispatched via the Nexplane Agent — no SSH required

### Account Lockdown

Simultaneously disables a compromised account across all connected identity systems:

- Active Directory
- Okta
- Microsoft Entra ID
- Google Workspace
- GitHub
- Slack

All actions run concurrently. Failure in one system does not block the others. The change request records the per-system result.

### Evidence Preservation

Collects forensic data from a host *before* any remediation runs:

- Auth logs (`/var/log/auth.log`, `journald`)
- Auditd records
- Active network connections (`netstat`)
- ARP cache
- Running process state

All collected files are packed into a tar.gz and uploaded to S3 via a pre-signed URL. The forensic bundle is stored in the `forensic_bundles` table and downloadable from the **Incident Response** tab in the UI.

### Phishing Response

Responds to a confirmed phishing incident:

- Blocks the sender domain at the email gateway
- Forces a password reset for all affected users
- Revokes active sessions across identity providers
- Forces MFA re-enrollment

## Expedited Approval

IR change requests bypass standard approval wait times. Users with the `ir_responder` role can approve and execute IR CRs immediately, even during a change freeze window (with mandatory justification logged to the audit trail).

## Forensic Bundles

Forensic evidence collected during Evidence Preservation is stored in the database and downloadable as a ZIP from the IR tab. Bundle metadata includes the target host, collection timestamp, file manifest, and S3 storage path.
