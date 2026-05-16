# RunZero Connector

The RunZero connector performs network asset discovery.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| API Key | Yes | RunZero API key |
| Organization ID | Yes | RunZero organization ID |

## Ingest

Discovers network assets (hosts, services, open ports) and syncs them into Nexplane asset inventory. Useful for finding unmanaged or shadow IT assets that are not tracked in cloud provider inventories.
