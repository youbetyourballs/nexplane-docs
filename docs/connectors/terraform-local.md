# Terraform (Local) Connector

The Terraform Local connector runs `terraform apply` directly inside the Nexplane backend container. AWS credentials are injected from the configured AWS connector — no separate credential management.

## Credential Fields

None — this connector uses the credentials from the linked AWS connector.

## How It Works

1. The operator submits a CR with Terraform HCL configuration
2. Nexplane runs `terraform init` and `terraform plan` inside the backend container
3. The plan output (diff) renders in the CR detail view with green/red coloring
4. After approval, Nexplane runs `terraform apply`
5. State is stored inside the container (ephemeral per-CR)

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `terraform_local_apply` | Run `terraform apply` in the backend container | `terraform destroy` with the same configuration |

## Supported Resources

Any resource supported by the AWS Terraform provider — S3 buckets, security groups, IAM policies, EC2 instances, VPCs, etc.

## Limitations

- State is ephemeral per CR — not suitable for long-lived infrastructure managed across multiple CRs
- One apply at a time — concurrent local Terraform runs are queued
- Backend container must have network access to AWS APIs

For persistent Terraform state and multi-CR workflows, use the [Terraform (Remote)](terraform-remote.md) connector.
