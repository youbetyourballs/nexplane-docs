# IaC Change Types

IaC change types integrate Terraform, Ansible, and Helm as tracked, approval-gated Nexplane change requests with plan diffs, rollback, and audit trails.

## Terraform

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `terraform_local_apply` | Run `terraform apply` inside the backend container with AWS credentials injected | `terraform destroy` with the same configuration |
| `terraform_apply` | Run `terraform apply` against an external Terraform workspace (two-phase: plan → approval → apply) | State restore |

**Plan diff:** The `terraform plan` output renders in the CR detail view with green (add) / red (destroy) coloring before approval.

## Ansible

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `ansible_local_playbook` | Run an Ansible playbook via SSM transport using `community.aws.aws_ssm` connection plugin | Defined per-playbook via rollback tasks |
| `ansible_playbook` | Run an Ansible playbook on a remote inventory | Defined per-playbook |

**Preflight:** `--check` mode runs automatically before the full playbook, showing what would change.

**Transport (local):** The backend container runs Ansible. The `session-manager-plugin` forwards execution to target EC2 instances via SSM — no direct SSH needed. `inventory_content` can be provided inline to override the target list.

## Helm

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `helm_upgrade` | Run `helm upgrade --atomic` on a Kubernetes release | `helm rollback` to the previous revision |
| `helm_rollback` | Explicit rollback to a previous Helm revision | N/A |

**Atomic flag:** `--atomic` causes Helm to automatically roll back the release if the deployment fails its health check within the configured timeout.

## Plan Diff Rendering

For all IaC change types, the plan output (Terraform diff, Ansible check output, Helm diff) renders in the change request detail view with syntax highlighting. Approvers see exactly what will change before signing off.

See [IaC Orchestration](../features/iac-orchestration.md) for full workflow details.
