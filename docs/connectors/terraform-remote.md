# Terraform (Remote) Connector

The Terraform Remote connector executes plans against an external Terraform workspace (Terraform Cloud, Terraform Enterprise, or a self-hosted backend).

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Workspace URL | Yes | Terraform workspace URL |
| Token | Yes | Terraform API token |

## How It Works

**Two-phase plan → apply:**

1. Nexplane triggers a plan run in the external workspace
2. The plan output is fetched and stored as the CR's blast radius preview
3. The operator reviews the plan in the CR detail view
4. After approval, Nexplane triggers the apply run
5. Apply status is polled until completion

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `terraform_apply` | Two-phase plan→approve→apply via external workspace | State restore from pre-apply state |

## Plan Diff

The Terraform plan output renders in the CR detail view with green (add) / red (destroy) coloring. Approvers see the exact resource changes before signing off.
