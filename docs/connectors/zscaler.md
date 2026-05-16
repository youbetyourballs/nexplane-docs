# Zscaler

The Zscaler connector manages Zscaler Internet Access (ZIA) users, URL filtering policies, and IP block lists.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Cloud | Yes | Zscaler cloud (e.g. `zscaler.net`) |
| API Key | Yes | ZIA API key |
| Username | Yes | ZIA admin username |
| Password | Yes | ZIA admin password |

## Ingest

Discovers ZIA users, URL filtering policies, network locations, and ZPA application segments.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `block_url` | Add a URL to the custom block category | Remove from block list |
| `block_ip` | Add an IP address to the deny list | Remove from deny list |
| `suspend_user` | Suspend a ZIA user account | Activate the user |
| `activate_user` | Reactivate a suspended ZIA user | Suspend the user |
| `update_url_category` | Add or remove URLs from a custom URL category | Restore previous category |
