# Checkov

The Checkov connector scans IaC repositories for security misconfigurations and secrets using the Checkov static analysis tool.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| IaC Repository Path | Yes | Path to the IaC repository inside the backend container (e.g. `/app/iac-repos/myrepo`) |
| Framework | No | `terraform`, `cloudformation`, `kubernetes`, `arm`, or `all` (default: all) |

## Capabilities

All Checkov actions are read-only ingest:

| Action | Description |
|--------|-------------|
| `scan_iac` | Run Checkov against the repo path; return findings with check ID, severity, resource, file, and remediation guideline |
| `scan_secrets` | Run Checkov secrets detection across all files |
| `get_compliance_summary` | Return pass/fail counts by compliance framework (CIS, NIST, PCI-DSS) |

Findings are imported as Nexplane vulnerability findings and flow into the vulnerability remediation pipeline.
