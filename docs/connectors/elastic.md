# Elastic Security

The Elastic Security connector syncs security alerts and manages detection rules via the Elasticsearch and Kibana APIs.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Elasticsearch URL | Yes | e.g. `http://elastic.example.com:9200` |
| Username | Yes | Elasticsearch username |
| Password | Yes | Elasticsearch password |
| Kibana URL | No | e.g. `http://elastic.example.com:5601` (required for detection rules) |
| Verify SSL | No | Verify SSL certificate |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `sync_alerts` | Pull security alerts from Elastic Security and sync as Nexplane findings (configurable time range and min severity) | N/A |
| `create_detection_rule` | Create a KQL detection rule in Kibana Security | Delete the rule |
