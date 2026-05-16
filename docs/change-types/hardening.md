# Hardening Change Types

Hardening changes improve security posture by applying configuration changes to hosts and network infrastructure.

## Network Hardening

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `security_group_update` | Add or remove rules from an AWS security group | Restore original ruleset |
| `microsegmentation_policy` | Apply a microsegmentation network policy | Staged simulation mode only — policy is simulated before applying |
| `dns_update` | Update DNS records | Restore previous DNS state |

**Note:** `microsegmentation_policy` runs in staged simulation mode by default. The safety engine blocks execution on production assets unless explicit simulation confirmation is provided.

## Host Isolation

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `isolate_host` | Flush iptables/nftables (Linux) or Windows Firewall rules; allow only management CIDR and Nexplane control plane | `restore_network_access` — applies pre-isolation state from snapshot |
| `restore_network_access` | Restore network access to an isolated host | N/A |

**Connector:** Nexplane Agent

The pre-isolation network state (all active rules) is captured before any flush. `restore_network_access` reapplies that snapshot exactly.

## OS Security Hardening (Agent)

Dispatched via the `ossecurity` agent command package:

| Command | Description |
|---------|-------------|
| `configure_selinux` | Set SELinux mode (enforcing/permissive/disabled) |
| `configure_seccomp` | Apply a seccomp filter profile |
| `apply_sysctl_hardening` | Apply kernel hardening parameters from a profile |
| `configure_host_firewall` | Configure iptables/nftables rules |
| `blacklist_kernel_modules` | Blacklist insecure kernel modules |
| `harden_mount_options` | Apply security mount options (noexec, nosuid, nodev) |
| `deploy_auditd_rules` | Deploy auditd rule set |
| `setup_file_integrity_monitoring` | Configure file integrity monitoring |
| `audit_os_security_posture` | Audit current OS security configuration |
| `configure_ebpf_security_policy` | Deploy an eBPF security policy |

**Connector:** Nexplane Agent (Linux only)

## Azure NSG

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `azure_nsg_update` | Add or update an NSG rule | Restore original rule |
| `azure_nsg_restore` | Restore an NSG to a previous state | N/A |

**Connector:** Azure

## GCP Firewall

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `gcp_firewall_create` | Create a GCP firewall rule | Delete the rule |
| `gcp_firewall_delete` | Delete a GCP firewall rule | Recreate the rule |

**Connector:** GCP

## Patch Packages

**Change type:** `patch_packages`

Apply OS package updates via the Nexplane Agent. Supports `apt`, `yum`, and `dnf`. Can target security-only updates or specific CVEs.

**Change type:** `patch_campaign`

Orchestrate a patch campaign across a fleet with batch size control and abort threshold.
