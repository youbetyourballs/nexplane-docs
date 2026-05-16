# Falco

The Falco connector manages Falco runtime security rules on EC2 instances via AWS SSM.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| EC2 Instance ID | Yes | SSM target instance ID (e.g. `i-0abc123456789def0`) |
| AWS Region | No | AWS region (default: us-east-1) |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `update_policy` | Write a Falco local rule to `/etc/falco/falco_rules.local.yaml` and restart the Falco service | Remove the rule and restart Falco |

**Parameters:** rule name, condition expression, priority (WARNING/ERROR/CRITICAL), output format string.
