# SSH Connector

The SSH connector uses the Paramiko library to connect to Linux/Unix hosts over SSH. Unlike a general-purpose SSH client, the connector does not execute arbitrary shell commands. All operations are typed command templates validated before execution.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Hostname | Yes | Target host hostname or IP |
| Username | Yes | SSH username |
| Private Key | Yes | SSH private key (PEM format) |
| Port | No | SSH port (default: 22) |

## Capabilities

| Action | Description |
|--------|-------------|
| Execute command template | Run a command from the connector's approved template list |
| Check service status | Query the status of a named systemd service |
| Tail logs | Retrieve the last N lines from a log file |

Freeform shell commands are blocked unconditionally by the safety engine. All permitted operations must be defined as typed templates.

!!! tip "Consider using the Nexplane Agent"
    For hosts you control, the [Nexplane Agent](nexplane-agent.md) provides a richer set of typed commands, self-registration, and outbound-only connectivity (no inbound SSH port required).
