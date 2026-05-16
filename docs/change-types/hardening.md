# Hardening Change Types

Hardening changes apply security configuration baselines to hosts and systems. They are typically used to bring systems into compliance with a security standard such as the CIS Benchmarks.

## Apply CIS Hardening Profile

Applies a named CIS Benchmark profile to a Linux or Windows host. The profile is a curated set of hardening operations (service disabling, sysctl settings, file permissions, registry changes) that have been validated by the Nexplane team against the official CIS benchmark documents.

**Connectors:** SSH, WinRM, Agent

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| Profile | string | CIS profile name (e.g., `cis-rhel9-level1`, `cis-windows-server-2022-level1`) |

**Available profiles:**

| Profile ID | Benchmark | Level |
|---|---|---|
| `cis-rhel9-level1` | CIS Red Hat Enterprise Linux 9 | Level 1 |
| `cis-rhel9-level2` | CIS Red Hat Enterprise Linux 9 | Level 2 |
| `cis-ubuntu2204-level1` | CIS Ubuntu Linux 22.04 LTS | Level 1 |
| `cis-ubuntu2204-level2` | CIS Ubuntu Linux 22.04 LTS | Level 2 |
| `cis-debian12-level1` | CIS Debian Linux 12 | Level 1 |
| `cis-windows-server-2022-level1` | CIS Microsoft Windows Server 2022 | Level 1 |
| `cis-windows-server-2022-level2` | CIS Microsoft Windows Server 2022 | Level 2 |

**Execution:**
Each profile runs a sequence of typed operations. Before each operation, the current state is captured and stored in the change record. If an individual step fails, execution continues with the remaining steps (failures are logged per-step). A partial success is reported if some steps succeed and some fail.

**Rollback:** Restores the pre-change state for each step that was successfully executed. Steps that failed during execution are skipped during rollback.

**Risk base score:** 6 (medium -- some hardening changes can break applications with non-standard configurations)

---

## Disable Unused Service

Stops and disables a named system service.

**Connectors:** SSH, WinRM, Agent

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| Service Name | string | Service name (e.g., `telnet`, `rsh`, `rlogin`, `tftp`) |

**Execution (Linux):**
```
systemctl disable --now <service>
```

**Execution (Windows):**
```
Stop-Service -Name "<service>"; Set-Service -Name "<service>" -StartupType Disabled
```

**Rollback:** Re-enables and starts the service.

**Risk base score:** 4 (medium -- disabling the wrong service can break functionality)

---

## Set File Permission

Sets the mode and ownership on a specific file or directory path.

**Connectors:** SSH, Agent

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| Path | string | Absolute file path (e.g., `/etc/ssh/sshd_config`) |
| Mode | string | Octal permission mode (e.g., `0600`) |
| Owner | string | Owning user (e.g., `root`) |
| Group | string | Owning group (e.g., `root`) |

**Rollback:** Restores the previous mode, owner, and group recorded before the change.

**Risk base score:** 3 (low -- targeted change to a specific file)

---

## Set Sysctl Parameter

Writes a kernel parameter value using `sysctl -w` and persists it to `/etc/sysctl.d/99-nexplane.conf`.

**Connectors:** SSH, Agent

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| Key | string | Sysctl key (e.g., `net.ipv4.conf.all.send_redirects`) |
| Value | string | Value to set (e.g., `0`) |

**Rollback:** Restores the previous value and removes the persistence file entry.

**Risk base score:** 4 (medium -- kernel parameter changes take effect immediately)
