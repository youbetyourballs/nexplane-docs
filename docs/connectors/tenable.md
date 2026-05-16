# Tenable Connector

The Tenable connector uses the `pytenable` SDK to manage scans and import vulnerability findings.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Access Key | Yes | Tenable API access key |
| Secret Key | Yes | Tenable API secret key |

## Ingest

Imports vulnerability findings from completed scans. Findings are matched to Nexplane assets by IP/hostname and can auto-generate draft remediation change requests based on your remediation policies.

## Capabilities

| Action | Description |
|--------|-------------|
| Launch scan | Start a scan immediately |
| Pause scan | Pause a running scan |
| Resume scan | Resume a paused scan |
| Export findings | Export findings from a completed scan |
| Trigger remediation verification | Re-scan a specific asset after remediation to verify the finding is resolved |

See [Vulnerability Remediation](../features/vulnerability-remediation.md) for how findings flow into Nexplane.
