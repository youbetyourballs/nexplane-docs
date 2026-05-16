# IaC Orchestration

Nexplane treats Terraform, Ansible, and Helm as tracked, approval-gated, auditable change types — with plan diffs, rollback, and the same safety engine applied to every other change request.

## Terraform (Local)

Runs `terraform init/plan/apply` directly inside the backend container. AWS credentials are injected from the configured AWS connector — no separate credential management needed.

**Change type:** `terraform_local_apply`

**Flow:**
1. Submit a CR with the Terraform configuration (HCL inline or referenced path)
2. Nexplane runs `terraform plan` and stores the diff
3. The plan output renders in the CR detail view with green (add) / red (destroy) coloring
4. After approval, Nexplane runs `terraform apply`
5. Rollback triggers `terraform destroy` with the same configuration

**Supported resources:** Any resource supported by the AWS provider (S3 buckets, security groups, IAM policies, etc.)

## Ansible (Local)

Runs Ansible playbooks via SSM transport. No direct SSH to target hosts needed.

**Change types:** `ansible_local_playbook`

**Transport:** `community.aws.aws_ssm` connection plugin with `session-manager-plugin`. The backend container runs the playbook; SSM forwards execution to the target EC2 instance.

**Flow:**
1. Submit a CR with the playbook content
2. Nexplane runs the playbook in `--check` mode as a preflight dry-run
3. After approval, runs the full playbook
4. `inventory_content` can be provided inline to override target selection

## Terraform (Remote)

Two-phase plan → apply workflow for external Terraform workspaces.

**Flow:**
1. Plan output is fetched and stored as a blast radius preview
2. Operator reviews the plan in the CR detail view
3. After approval, Nexplane triggers the apply
4. Rollback restores from the Terraform state prior to the apply

## Helm

Deploys or upgrades Kubernetes releases with automatic rollback on failure.

**Change type:** `helm_upgrade`

**Flow:**
1. Submit a CR with chart name, namespace, values override
2. Nexplane runs `helm upgrade --atomic`
3. If the deployment fails its health check within the configured timeout, Helm automatically rolls back
4. Manual rollback via `helm rollback` is available from the CR detail page

## Plan Diff Rendering

For all IaC change types, the plan output (Terraform diff, Ansible check output, Helm diff) renders in the change request detail view with syntax highlighting and green/red coloring. Reviewers see exactly what will change before approving.
