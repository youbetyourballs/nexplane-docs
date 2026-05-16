# Agent Command Packages

The Nexplane Agent supports 15 command packages. All commands are typed — registered at compile time. No arbitrary shell execution is possible.

## Command Package Reference

| Package | Platform | Commands |
|---------|----------|---------|
| `changip` | Linux + Windows | `change_ip`, `change_ip_rollback` |
| `linuxpatch` | Linux | `apply_linux_patches`, `audit_linux_patch_status` |
| `winpatch` | Windows | `apply_windows_patches`, `audit_windows_patch_status` |
| `isolation` | Linux + Windows | `isolate_host`, `restore_network_access` |
| `forensics` | Linux + Windows | `collect_forensics` |
| `compliance` | Linux | `audit_cis_compliance`, `collect_evidence` |
| `credrotation` | Linux + Windows | `rotate_db_credentials`, `rotate_ssh_keys`, `update_agent_env_file` |
| `iac` | Linux | `terraform_plan`, `terraform_apply`, `terraform_rollback`, `ansible_check`, `ansible_run`, `helm_diff`, `helm_upgrade`, `helm_rollback` |
| `fleet` | Linux + Windows | `restart_service`, `push_config_file`, `distribute_file`, `health_check` |
| `backup` | Linux + Windows | `create_backup`, `restore_files` |
| `reboot` | Linux + Windows | `graceful_reboot`, `verify_post_reboot` |
| `dbadmin` | PostgreSQL, MySQL, MSSQL | `provision_db_user`, `deprovision_db_user`, `grant_permissions`, `revoke_permissions`, `configure_db_audit`, `db_connection_config` |
| `ossecurity` | Linux | `configure_selinux`, `configure_seccomp`, `apply_sysctl_hardening`, `configure_host_firewall`, `blacklist_kernel_modules`, `harden_mount_options`, `deploy_auditd_rules`, `setup_file_integrity_monitoring`, `audit_os_security_posture`, `audit_ebpf_posture`, `configure_ebpf_security_policy`, `deploy_ebpf_policy` |
| `linuxauth` | Linux | `harden_ssh`, `configure_pam`, `manage_ca_certificates`, `configure_ntp`, `audit_users_and_groups`, `audit_privesc_vulnerabilities` |
| `winharden` | Windows | LAPS, Credential Guard, PowerShell CLM, AppLocker, SMB signing, BitLocker, Windows Firewall, TLS protocols, RDP hardening, audit policy, registry hardening |
| `crossplatform` | Linux + Windows | `harden_tls_protocols`, `configure_dns_resolver`, `audit_software_inventory`, `configure_syslog` |
| `linuxupgrade` | Linux | `estimate_image_size`, in-place OS upgrade, containerize-and-migrate |

## Package Details

### changip

IP address change operations with rollback safety. See [IP Migration](../features/ip-migration.md).

- `change_ip` — change a host's IP using the lowest-risk method available (tailscale/secondary_swap/commit_timer/manual)
- `change_ip_rollback` — restore all pre-change network state (IP, gateway, routes, DNS, MTU) from snapshot

### linuxpatch

Apply OS package updates on Linux:

- `apply_linux_patches` — apt/yum/dnf; security-only or CVE-targeted; dry-run support; before/after diff
- `audit_linux_patch_status` — report pending updates without applying

### winpatch

Apply Windows updates via the Windows Update Agent COM API:

- `apply_windows_patches` — target specific KBs or all pending; schedule reboot if required
- `audit_windows_patch_status` — report pending Windows updates without applying

### isolation

Network isolation for incident response:

- `isolate_host` — flush all network rules; allow only management CIDR and Nexplane control plane
- `restore_network_access` — restore pre-isolation network state from snapshot

### forensics

- `collect_forensics` — collect auth logs, journal, auditd, netstat, ARP, process state → tar.gz → S3 pre-signed upload

### compliance

- `audit_cis_compliance` — per-control pass/fail across filesystem, sysctl, SSH, PAM, auditd, SELinux/AppArmor
- `collect_evidence` — collect config files and command outputs for a specific compliance control → downloadable ZIP

### credrotation

- `rotate_db_credentials` — generate new password, update DB user, update config files, restart service, verify connection
- `rotate_ssh_keys` — remove old key by fingerprint from `authorized_keys`, add new public key
- `update_agent_env_file` — update a key-value pair in an agent environment file

### iac

IaC CLI wrappers (Linux agent, runs CLI tools in-process):

- `terraform_plan`, `terraform_apply`, `terraform_rollback`
- `ansible_check`, `ansible_run`
- `helm_diff`, `helm_upgrade`, `helm_rollback`

### fleet

Fleet coordination:

- `restart_service` — restart a named systemd/Windows service
- `push_config_file` — write a file with backup of the original
- `distribute_file` — push a file to the host (no backup)
- `health_check` — disk usage, load average, service status, pending-reboot flag

### backup

restic-based backup and restore:

- `create_backup` — backup specified paths to S3 using restic
- `restore_files` — restore files from a restic snapshot with path filter and SHA256 verification

### reboot

- `graceful_reboot` — drain connections, notify services, reboot
- `verify_post_reboot` — check named services are healthy after reboot

### dbadmin

Database user and permission management (PostgreSQL, MySQL, MSSQL):

- `provision_db_user` — create a database user with scoped grants
- `deprovision_db_user` — drop a database user
- `grant_permissions` — grant specific permissions
- `revoke_permissions` — revoke specific permissions
- `configure_db_audit` — configure audit logging (pg_audit, general_log, SQL Audit)
- `db_connection_config` — update connection limits and timeout settings

### ossecurity

Linux OS security hardening:

- SELinux mode configuration
- seccomp filter deployment
- sysctl kernel hardening parameters
- iptables/nftables host firewall configuration
- Kernel module blacklisting
- Mount option hardening (noexec/nosuid/nodev)
- auditd rule deployment
- File integrity monitoring setup
- OS security posture audit
- eBPF security policy deployment

### linuxauth

Linux authentication and access hardening:

- SSH daemon configuration hardening
- PAM configuration
- CA certificate management
- NTP configuration
- User and group audit
- Privilege escalation vulnerability audit

### winharden

Windows security hardening suite:

- LAPS configuration
- Credential Guard
- PowerShell Constrained Language Mode
- AppLocker policy
- SMB signing enforcement
- BitLocker drive encryption
- Windows Firewall configuration
- TLS protocol hardening
- RDP security settings
- Audit policy configuration
- Registry hardening

### crossplatform

Cross-platform utilities:

- `harden_tls_protocols` — disable insecure TLS versions and cipher suites
- `configure_dns_resolver` — set DNS resolver configuration
- `audit_software_inventory` — enumerate installed packages/software
- `configure_syslog` — configure syslog forwarding

### linuxupgrade

Linux OS upgrade operations:

- `estimate_image_size` — non-destructive estimation of OS image size for migration planning
- In-place OS upgrade
- Containerize-and-migrate (package running OS into a container image for migration)
