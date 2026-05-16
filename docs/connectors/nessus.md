# Nessus Essentials

The Nessus connector runs vulnerability scans via the Nessus REST API.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Nessus URL | Yes | e.g. `https://nessus-host:8834` |
| Username | Yes | Nessus username |
| Password | Yes | Nessus password |

## Capabilities

| Action | Description |
|--------|-------------|
| `run_scan` | Create and launch a Nessus scan, poll to completion (up to 30 min), return vulnerability findings. `policy_id` defaults to Basic Network Scan template. |

Findings flow into the [Vulnerability Remediation Pipeline](../features/vulnerability-remediation.md).
