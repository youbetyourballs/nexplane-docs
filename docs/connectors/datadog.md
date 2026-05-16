# Datadog Connector

The Datadog connector imports monitors and creates alerts.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| API Key | Yes | Datadog API key |
| App Key | Yes | Datadog Application key |
| Site | No | Datadog site (e.g., `datadoghq.com` or `datadoghq.eu`) |

## Capabilities

| Action | Description |
|--------|-------------|
| Monitor ingest | Import Datadog monitors as Nexplane assets |
| Create alert | Create a Datadog monitor from a Nexplane change request |

Nexplane change request executions can create Datadog events for correlation with infrastructure metrics.
