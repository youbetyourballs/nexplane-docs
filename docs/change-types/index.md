# Change Types

A change type is a structured, typed operation that Nexplane knows how to execute and roll back. Every change request is associated with exactly one change type. Change types define what connector they require, what parameters they accept, what the execution steps are, and what the rollback operation is.

## Full Change Type Catalog

| Category | Change Types |
|----------|-------------|
| Infrastructure | `dns_update`, `security_group_update`, `microsegmentation_policy`, `snapshot_asset` |
| EC2 | `ec2_launch`, `ec2_start`, `ec2_stop`, `ec2_reboot`, `ec2_terminate`, `key_pair_create`, `key_pair_delete` |
| SSM | `ssm_command` |
| Network | `tailscale_join`, `tailscale_remove` |
| IP Migration | `change_ip`, `migrate_ip`, `ip_campaign` |
| Agent | `deploy_nexplane_agent`, `patch_packages`, `patch_campaign`, `isolate_host`, `rolling_restart`, `canary_config_push`, `distribute_file`, `fleet_health_check` |
| Identity | `offboard_user`, `onboard_user`, `key_rotation`, `rotate_db_credentials`, `rotate_ssh_keys`, `rotate_api_key`, `rotate_service_account` |
| IAM | `iam_user_create`, `iam_user_delete` |
| S3 Storage | `s3_bucket_create`, `s3_bucket_delete`, `s3_lifecycle_configure` |
| DNS (Route53) | `route53_zone_create`, `route53_record_upsert`, `route53_record_delete` |
| RDS | `rds_instance_create`, `rds_instance_delete`, `rds_snapshot_create` |
| Observability | `cloudwatch_alarm_create`, `cloudwatch_alarm_delete` |
| Incident Response | `lockdown_account`, `phishing_response`, `preserve_evidence` |
| IaC (local) | `terraform_local_apply`, `ansible_local_playbook` |
| IaC (remote) | `terraform_apply`, `ansible_playbook`, `helm_upgrade` |
| Database | `provision_db_user`, `deprovision_db_user`, `db_permission_change`, `configure_db_audit`, `promote_db_replica`, `db_connection_config` |
| Backup / Recovery | `create_backup`, `verify_backup`, `restore_files`, `dr_failover`, `scheduled_reboot` |
| Compliance | `enforce_cis_benchmark`, `collect_evidence` |
| SaaS (Google Workspace) | `remove_from_groups`, `reset_2fa`, `revoke_oauth_tokens`, `wipe_mobile_device`, `suspend_user`, `unsuspend_user` |
| SaaS (GitHub) | `remove_org_member`, `revoke_user_pats`, `enforce_branch_protection`, `archive_repo`, `disable_actions`, `enable_actions` |
| SaaS (Slack) | `deactivate_user`, `reactivate_user` |
| SaaS (Entra ID) | `remove_from_teams`, `assign_license`, `remove_license`, `revoke_sessions`, `disable_user` |
| SaaS (Kubernetes) | `restart_deployment`, `scale_deployment`, `apply_network_policy`, `update_rbac`, `rotate_secret`, `helm_upgrade`, `helm_rollback` |
| macOS | `macos_filevault_enable`, `macos_gatekeeper_enable`, `macos_santa_install`, `macos_santa_rule_add`, `macos_santa_mode_set`, `macos_softwareupdate_install`, `macos_profiles_install`, `macos_defaults_write`, `macos_sysinfo` … (23 macOS change types) |
| Security Policy | `apply_seccomp_profile`, `apply_apparmor_profile`, `apply_selinux_policy`, `apply_ebpf_policy` (synthesized from soak sessions) |
| Telemetry | `telemetry_agent_deploy`, `remote_command` |

## Risk Scoring

Each change type has a base risk score. The final score for a change request is calculated from:

- Base risk score of the change type
- Environment label of the connector (prod scores higher than staging)
- Blast radius of the target (how many systems depend on it)
- Whether rollback is available

| Score | Level | Approval Required |
|-------|-------|------------------|
| 1–3 | Low | None (auto-approved in non-prod) |
| 4–6 | Medium | 1 approver |
| 7–8 | High | 1 approver (approver or admin role) |
| 9–10 | Critical | 2 approvers (approver + admin) |

## Pages by Category

- [Compute](compute.md) — EC2, SSM, agent deploy, IP migration
- [Identity & IAM](identity.md) — offboarding, onboarding, IAM users, SaaS identity actions
- [Credentials](credentials.md) — key rotation, DB credentials, SSH keys, API keys
- [Hardening](hardening.md) — security groups, microsegmentation, host isolation
- [IaC](iac.md) — Terraform, Ansible, Helm
- [Database](database.md) — user provisioning, permissions, audit, RDS
- [Backup & Recovery](backup.md) — backups, restore, DR failover, scheduled reboot
- [Compliance](compliance.md) — CIS benchmark, evidence collection
- [Incident Response](incident-response.md) — account lockdown, phishing response, evidence preservation
- [Telemetry](telemetry.md) — agent deploy, remote command, CloudWatch, fleet ops
