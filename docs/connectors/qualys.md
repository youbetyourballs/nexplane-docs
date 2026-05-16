# Qualys Connector

The Qualys connector imports vulnerability findings and launches scans.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Username | Yes | Qualys username |
| Password | Yes | Qualys password |
| API URL | Yes | Qualys API URL (e.g., `https://qualysapi.qualys.com`) |

## Ingest

Imports vulnerability findings from Qualys scan results. Findings are matched to Nexplane assets by IP/hostname and flow into the vulnerability remediation pipeline.

## Capabilities

| Action | Description |
|--------|-------------|
| Launch scan | Start a Qualys vulnerability scan |
| Import findings | Import completed scan results |

See [Vulnerability Remediation](../features/vulnerability-remediation.md) for how findings flow into Nexplane.
