# Host Intelligence

These tools dispatch live agent jobs to the Nexplane Agent running on the target host. Results reflect the current host state — not a cached snapshot. Results are cached 300 seconds per (organization, asset, tool) to avoid repeated queries during planning.

!!! note "Agent required"
    Host intelligence tools require the Nexplane Agent to be installed and connected on the target asset. Use `list_assets` to confirm the asset has `agent_connected: true` before calling these tools.

## Full Context

`get_host_full_context` aggregates all host intelligence tools into a single call. Use it when you need a complete picture of a host before planning changes.

**Parameters:** `asset_id`

Returns a bundle containing kernel info, running processes, cron jobs, local users, installed packages, running services, open ports, security posture, seccomp policy, AppArmor profiles, SELinux policy, sudoers rules, authorized keys, SSL certificates, and patch status.

## Individual Tools

| Tool | Description |
|------|-------------|
| `get_kernel_info` | Return kernel version, architecture, EOL status, and support state for a host. |
| `get_running_processes` | List all running processes on a host including pid, name, owner, command line, and open ports per process. |
| `get_cron_jobs` | List all scheduled cron jobs on a host (crontab, cron.d, systemd timers). Returns schedule expression, command, owning user, and source file. |
| `get_local_users` | List all local user accounts on a host. Returns username, uid, gid, group memberships, shell, last login, and locked status. |
| `get_installed_packages` | List all installed packages on a host (apt, yum/rpm, pip, snap, etc.). Returns package name, version, source manager, and install date. |
| `get_running_services` | List systemd/init services on a host with their state, enabled flag, and user they run as. |
| `get_open_ports` | List all listening TCP/UDP ports on a host with the process that owns each. Returns port number, protocol, process name, process owner, and bind address. |
| `get_security_posture` | Return a summary security posture: SELinux/AppArmor/seccomp modes, firewall rules count, and last audit timestamp. |
| `get_seccomp_policy` | Return eBPF/seccomp posture: active profiles, per-process profile map, unconfined processes. |
| `get_apparmor_profiles` | List AppArmor profiles with their enforcement mode and the processes they confine. |
| `get_selinux_policy` | Return SELinux enforcement mode, active policy name, count of recent denials, and a sample of the most recent denial messages. |
| `get_sudoers` | Return all sudoers rules with a risky flag for NOPASSWD or ALL grants. |
| `get_authorized_keys` | List all SSH authorized_keys entries across all user home directories. Returns key fingerprint, owning user, last-used timestamp, and stale flag. |
| `get_ssl_certs` | List TLS/SSL certificates found on a host. Returns subject, expiry, days remaining, issuer, and whether the chain is valid. |
| `get_patch_status` | Return patch status: count of pending patches, CVEs addressed, last patched timestamp, and list of critical pending patches. |

All tools require `token` and `asset_id` as parameters.
