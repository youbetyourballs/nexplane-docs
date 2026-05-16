# SentinelOne Connector

The SentinelOne connector discovers endpoints and imports alerts.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| API Token | Yes | SentinelOne API token |
| Site URL | Yes | SentinelOne management console URL |

## Ingest

Discovers endpoints and their protection status. Imports threat alerts as vulnerability findings.

## Capabilities

Endpoint discovery and alert ingest. Change actions (isolation, remediation) are planned in a future release.
