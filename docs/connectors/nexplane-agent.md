# Nexplane Agent Connector

The Nexplane Agent Connector is the interface for dispatching commands to registered Nexplane Agents running on managed hosts. It is automatically configured when you generate an agent secret in Settings.

## Credential Fields

None — this connector uses the agent HMAC secret configured in **Settings → Agent Configuration**.

## How It Works

When a change request targets an agent-registered asset and uses an agent-supported change type, Nexplane dispatches the job via `POST /agent/jobs/` signed with the agent HMAC secret. The agent picks up the job on its next poll, verifies the signature, executes the command, and posts the result back.

## Supported Command Packages

All 15 agent command packages are available through this connector:

| Package | Commands | Platform |
|---------|----------|----------|
| `changip` | `change_ip`, `change_ip_rollback` | Linux + Windows |
| `linuxpatch` | `apply_linux_patches`, `audit_linux_patch_status` | Linux |
| `winpatch` | `apply_windows_patches`, `audit_windows_patch_status` | Windows |
| `isolation` | `isolate_host`, `restore_network_access` | Linux + Windows |
| `forensics` | `collect_forensics` | Linux + Windows |
| `compliance` | `audit_cis_compliance`, `collect_evidence` | Linux |
| `credrotation` | `rotate_db_credentials`, `rotate_ssh_keys`, `update_agent_env_file` | Linux + Windows |
| `iac` | `terraform_plan`, `terraform_apply`, `terraform_rollback`, `ansible_check`, `ansible_run`, `helm_diff`, `helm_upgrade`, `helm_rollback` | Linux |
| `fleet` | `restart_service`, `push_config_file`, `distribute_file`, `health_check` | Linux + Windows |
| `backup` | `create_backup`, `restore_files` | Linux + Windows |
| `reboot` | `graceful_reboot`, `verify_post_reboot` | Linux + Windows |
| `dbadmin` | `provision_db_user`, `deprovision_db_user`, `grant_permissions`, `revoke_permissions`, `configure_db_audit`, `db_connection_config` | PostgreSQL, MySQL, MSSQL |
| `ossecurity` | SELinux, AppArmor, seccomp, sysctl, iptables, kernel modules, mount hardening, auditd, FIM, eBPF | Linux |
| `linuxauth` | PAM, SSH, CA certs, NTP, user/group audit, privesc audit | Linux |
| `winharden` | LAPS, Credential Guard, PowerShell CLM, AppLocker, SMB, BitLocker, Windows Firewall, TLS, RDP, audit policy, registry | Windows |
| `crossplatform` | TLS certificates, DNS resolver, software inventory, syslog forwarding | Linux + Windows |
| `linuxupgrade` | In-place or containerize-and-migrate OS upgrades | Linux |

See [Agent Command Packages](../agent/commands.md) for the full command reference.
