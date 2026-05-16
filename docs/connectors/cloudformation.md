# AWS CloudFormation

The AWS CloudFormation connector manages stacks and detects infrastructure drift.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Access Key ID | Yes | AWS IAM access key |
| Secret Access Key | Yes | AWS IAM secret key |
| Region | Yes | AWS region (default: us-east-1) |
| Session Token | No | Temporary session token |

## Ingest

Discovers all CloudFormation stacks (name, status, template URL, outputs) and stack resources (logical/physical IDs, type, status).

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `detect_stack_drift` | Initiate drift detection on a stack | N/A |
| `get_drift_results` | Retrieve drift detection results | N/A |
| `create_change_set` | Create a CloudFormation change set from a template | Delete the change set |
| `execute_change_set` | Execute a change set to update the stack | Rollback via stack rollback |
| `delete_stack` | Delete a CloudFormation stack and all its resources | Not available (destructive) |
| `update_termination_protection` | Enable or disable stack termination protection | Reverse the setting |
