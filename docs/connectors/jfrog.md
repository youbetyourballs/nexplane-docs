# JFrog Xray

The JFrog Xray connector scans artifacts for security vulnerabilities and imports policy violations.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| JFrog Base URL | Yes | JFrog platform URL |
| Username | Yes | JFrog username |
| Password or API Token | Yes | Password or API token |

## Capabilities

| Action | Description |
|--------|-------------|
| `scan_artifact` | Trigger an Xray security scan of a specific artifact and return violations/CVEs |
| `sync_violations` | Pull all Xray policy violations and import as Nexplane vulnerability findings |
