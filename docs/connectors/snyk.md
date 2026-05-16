# Snyk Connector

The Snyk connector imports vulnerability findings from Snyk for code, container, and infrastructure-as-code scans.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| API Token | Yes | Snyk API token |
| Organization ID | Yes | Snyk organization ID |

## Ingest

Imports Snyk vulnerability findings (CVEs, license issues, misconfigurations) and syncs them into the Nexplane vulnerability pipeline. Findings are matched to assets by project name or repository.

See [Vulnerability Remediation](../features/vulnerability-remediation.md) for how findings flow into Nexplane.
