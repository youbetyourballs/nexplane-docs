# ServiceNow Connector

The ServiceNow connector syncs Nexplane change requests with ServiceNow change management.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Instance URL | Yes | ServiceNow instance URL (e.g., `https://acme.service-now.com`) |
| Username | Yes | ServiceNow username |
| Password | Yes | ServiceNow password |

## Capabilities

Nexplane change requests are synced as ServiceNow change records. Status transitions in Nexplane update the ServiceNow record state. Approval decisions in ServiceNow can be reflected back to Nexplane.
