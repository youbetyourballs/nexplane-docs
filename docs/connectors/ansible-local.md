# Ansible (Local) Connector

The Ansible Local connector runs Ansible playbooks inside the Nexplane backend container, targeting EC2 instances via AWS SSM transport. No direct SSH connection to target hosts is required.

## Credential Fields

None — this connector uses the credentials from the linked AWS connector.

## How It Works

1. The operator submits a CR with a playbook (YAML content)
2. Nexplane runs the playbook in `--check` mode first (dry-run preflight)
3. The check output renders in the CR detail view
4. After approval, Nexplane runs the full playbook
5. Ansible connects to target instances via `community.aws.aws_ssm` + `session-manager-plugin` — no SSH port needed

## Capabilities

| Action | Description |
|--------|-------------|
| `ansible_local_playbook` | Run an Ansible playbook via SSM transport |

## Custom Inventory

Set `inventory_content` in the CR parameters to provide a custom Ansible inventory, overriding the default (all registered EC2 instances matching the target asset).

## Supported Transport

The backend container includes:
- Ansible + `community.aws` collection
- AWS `session-manager-plugin`
- Tailscale (for reaching instances in private VPCs via the tailnet)
