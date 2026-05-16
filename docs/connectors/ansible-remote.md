# Ansible (Remote) Connector

The Ansible Remote connector runs Ansible playbooks against a remote inventory using standard SSH transport.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Inventory | Yes | Ansible inventory file path or content |
| Private Key | Yes | SSH private key for connecting to target hosts |
| Become Password | No | sudo/become password if required |

## Capabilities

| Action | Description |
|--------|-------------|
| `ansible_playbook` | Run an Ansible playbook against a remote inventory |

**Preflight:** `--check` mode runs automatically before the full playbook.

For targeting EC2 instances without direct SSH access, use [Ansible (Local)](ansible-local.md) instead.
