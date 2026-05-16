# Splunk Connector

The Splunk connector ingests events and runs searches.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Host | Yes | Splunk server hostname or IP |
| Token | Yes | Splunk HEC or REST API token |
| Port | No | Splunk API port (default: 8089) |

## Capabilities

| Action | Description |
|--------|-------------|
| Event ingest | Forward Nexplane audit events to Splunk via HEC |
| Search query | Run a Splunk search query and import results |

Change request executions and audit events can be forwarded to Splunk in real time for SIEM correlation.
