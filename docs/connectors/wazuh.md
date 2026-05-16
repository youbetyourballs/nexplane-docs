# Wazuh

The Wazuh connector registers new agents with a Wazuh manager via the Wazuh REST API.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Wazuh API URL | Yes | e.g. `https://wazuh.example.com:55000` |
| API Username | Yes | Wazuh API username (e.g. `wazuh-wui`) |
| API Password | Yes | Wazuh API password |
| Verify SSL | No | Verify SSL certificate |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `deploy_agent` | Register a new Wazuh agent by name. Returns the `agent_id` assigned by the manager. | N/A |
