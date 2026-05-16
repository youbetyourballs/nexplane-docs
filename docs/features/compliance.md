# Compliance & Governance

Nexplane automates CIS benchmark enforcement, detects configuration drift, enforces change freeze windows, and collects audit evidence — all as tracked, audited change requests.

## CIS Benchmark Campaigns

Audit hosts against CIS benchmark controls, then apply remediation:

**Audit controls covered:**

- Filesystem permissions and mount options
- Kernel sysctl hardening parameters
- SSH configuration
- PAM configuration
- auditd rules
- SELinux / AppArmor policy
- Network parameters

**Campaign flow:**

1. Audit phase — agent collects per-control pass/fail results
2. Nexplane displays a before/after compliance score
3. Remediation CRs are generated for failed controls
4. After execution, a post-remediation audit confirms the score improvement

## Drift Detection

A weekly scheduled scan compares each host's current configuration against its baseline snapshot. When drift is detected (a previously-passing control is now failing), Nexplane creates a draft remediation CR automatically.

Drift alerts are visible in the **Compliance** section of the UI. Configure scan frequency and alert thresholds via `GET/PUT /compliance/baselines/`.

## Change Freeze Enforcement

Declare a freeze window to block all non-emergency changes during critical periods (release windows, audits, holidays):

- When a freeze is active, the approve and execute endpoints return `423 Locked`
- Emergency bypass requires the `ir_responder` role and a mandatory `X-Bypass-Justification` header
- All bypass attempts — whether successful or not — are recorded in the immutable audit trail

Freeze windows are created via `POST /compliance/freeze-windows/` or in **Settings → Compliance**.

## Audit Evidence Collection

Request evidence for a specific compliance control (SOC2, PCI DSS, ISO 27001):

1. Create a `collect_evidence` change request specifying the control ID and target hosts
2. The Nexplane Agent collects the relevant config files and command outputs
3. Nexplane packages everything as a downloadable ZIP

Evidence ZIPs are stored in the database and downloadable from the Compliance section. Each collection is timestamped and linked to the approved change request that authorized it.

Endpoint: `GET /compliance/evidence/{id}/download`
