# Palo Alto Connector

The Palo Alto connector uses the `pan-os-python` library to manage firewall rules and security policies on Palo Alto Networks firewalls.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Hostname | Yes | Firewall hostname or IP |
| Username | Yes | Admin username |
| Password | Yes | Admin password |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| Create address object | Add an address object | Delete the object |
| Create security rule | Add a security policy rule | Delete the rule |
| Delete security rule | Remove a security policy rule | Recreate the rule |
| Block IP | Add an IP to a block-list address group | Remove from group |
| Manage zones | Create or modify security zones | Restore previous zone config |
| Commit | Push the candidate configuration to the running configuration | N/A |

Changes on Palo Alto firewalls are staged in the candidate configuration. Nexplane automatically commits after each change unless `auto_commit: false` is set in the CR parameters.
