# Wiz Connector

The Wiz connector imports cloud security findings and misconfigurations.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Client ID | Yes | Wiz API client ID |
| Client Secret | Yes | Wiz API client secret |
| Endpoint | Yes | Wiz API endpoint URL |

## Ingest

Imports Wiz security findings — misconfigurations, vulnerabilities, toxic combinations, and network exposure issues. Findings are matched to Nexplane cloud assets and flow into the vulnerability remediation pipeline.

See [Vulnerability Remediation](../features/vulnerability-remediation.md) for how findings flow into Nexplane.
