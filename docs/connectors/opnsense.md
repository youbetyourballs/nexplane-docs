# OPNsense

The OPNsense connector manages firewall rules on OPNsense routers via the OPNsense REST API.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| OPNsense Base URL | Yes | e.g. `https://opnsense.example.com` |
| API Key | Yes | OPNsense API key |
| API Secret | Yes | OPNsense API secret |
| Verify SSL | No | Verify SSL certificate (default: false) |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `update_firewall_rule` | Create or update a firewall filter rule and apply it | Delete the rule |
| `block_host` | Block an IP address by creating an alias and a block rule | Remove the alias and rule |
