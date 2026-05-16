# OpenVAS / Greenbone Community Edition

The OpenVAS connector runs vulnerability scans against network targets using the Greenbone Security Assistant (GSA) API.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| GSA URL | Yes | e.g. `http://openvas-host:9392` |
| Username | Yes | OpenVAS admin username |
| Password | Yes | OpenVAS admin password |

## Capabilities

| Action | Description |
|--------|-------------|
| `run_scan` | Create a target and task, launch a full OpenVAS scan, poll to completion (up to 30 min), return CVE findings |
| `import_findings` | Import OpenVAS scan findings as Nexplane vulnerability findings linked to assets |

Findings flow into the [Vulnerability Remediation Pipeline](../features/vulnerability-remediation.md).
