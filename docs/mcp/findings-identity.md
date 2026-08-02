# Findings & Identity

## Findings

Vulnerability and security findings surface from connected scanners. These tools let agents triage, assign, and act on findings. Each finding tracks lifecycle status, exploitability validation, and linked remediation CRs.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `list_findings` | List vulnerability findings for the org. Filter by status (open, actionable, remediating, resolved, etc.), severity (critical/high/medium/low), CVE ID, or asset ID. Returns summary fields — use `get_finding` for full detail. | `status`, `severity`, `cve_id`, `asset_id`, `limit` |
| `get_finding` | Get full detail for a single finding including PoC result, verification status, linked CRs, and SLA countdown. Use this after `list_findings` to get depth on a specific item. | `finding_id` |
| `update_finding_status` | Update a finding's lifecycle status. Updates Nexplane metadata only — does not touch external systems. Valid values: open, accepted_risk, false_positive, exploitability_pending, actionable, remediating, verifying, resolved. | `finding_id`, `status` |
| `assign_finding` | Assign a finding to a Nexplane user by their user ID. Updates Nexplane metadata only. | `finding_id`, `user_id` |
| `accept_risk` | Mark a finding as risk-accepted with a stated reason and expiry date (ISO 8601 format). The finding reappears on the SLA dashboard at expiry for re-review. | `finding_id`, `reason`, `expires_at` |
| `mark_false_positive` | Close a finding as a false positive. No SLA credit is given. Not reversible without re-ingesting the finding from the scanner. | `finding_id` |
| `trigger_poc_validation` | Trigger a PoC validation run for this finding. If the CVE is on the CISA KEV catalog, the finding is immediately marked exploited. Otherwise creates a `vuln_poc_validate` CR in draft state. Returns the result and asset context bundle for planning. | `finding_id`, `asset_id` |
| `get_poc_result` | Get the current PoC validation result for a finding. Result values: exploited, not_exploited, inconclusive, no_poc_available, or null if not yet run. | `finding_id` |
| `challenge_exploitability` | Submit an exploitability challenge explaining why this CVE is not exploitable in your environment. Disabled for CISA KEV findings — those cannot be challenged. | `finding_id`, `reason` |
| `trigger_verification` | Trigger a scanner re-probe to verify remediation actually removed the vulnerability. Returns the verification result and asset context bundle. Result values: resolved, still_vulnerable, inconclusive. | `finding_id` |
| `get_verification_result` | Get the latest scanner verification probe result for a finding. | `finding_id` |
| `list_finding_change_requests` | List all Change Requests linked to a finding (patch, mitigation, poc_validate, verify) with their current status. Use this to track remediation progress from the finding. | `finding_id` |

!!! note "Finding lifecycle"
    Status updates via `update_finding_status`, `accept_risk`, and `mark_false_positive` modify Nexplane metadata only — they do not create or modify external systems. Remediation actions (patches, mitigations) are initiated via `trigger_poc_validation`, which may create a CR in draft state.

## Identity

Identity tools inspect user access across connected identity systems. They surface user profiles, account correlations, group memberships, and identity-related security findings (stale accounts, over-privilege, orphaned credentials).

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `list_identities` | List identity profiles for the org. Optionally filter by source IdP connector or stale status. Returns summary fields — use `get_identity` for full detail. | `source_connector_id`, `is_stale`, `limit` |
| `get_identity` | Get full identity profile detail including all linked accounts across connectors. | `identity_id` |
| `list_identity_findings` | List security findings associated with an identity (stale accounts, over-privilege, orphaned credentials). Filter by status (open/actionable/remediating/etc.). | `identity_id`, `status`, `limit` |
| `get_identity_graph` | Get the identity graph for a profile — all linked accounts, group memberships (from raw_attributes), and stale account flags. Use this to understand the full footprint of an identity across systems. | `identity_id` |
| `list_access_reviews` | List access review campaigns with status and completion rate. Filter by status (pending/in_progress/completed/overdue). | `status`, `limit` |
| `get_access_review` | Get full access review detail including pending decisions and completion rate. | `review_id` |

!!! note "Identity correlation"
    Identities are correlated across connectors using email, username, or other attributes. A single identity profile may link accounts from Okta, GitHub, AWS, Kubernetes, and other IdP systems. Use `get_identity_graph` to visualize these relationships.
