# Compliance Change Types

Compliance change types audit and enforce security configuration baselines and collect evidence for audits.

## CIS Benchmark Enforcement

**Change type:** `enforce_cis_benchmark`

Audits a host against CIS benchmark controls, then applies remediation for failing controls.

**Connector:** Nexplane Agent (`compliance` package, Linux only)

**Controls covered:**

- Filesystem permissions and mount options
- Kernel sysctl hardening parameters
- SSH daemon configuration
- PAM configuration
- auditd rules
- SELinux / AppArmor policy

**Flow:**
1. Audit phase: agent collects per-control pass/fail results
2. Nexplane displays a compliance score (controls passing / total controls)
3. Remediation is applied for failing controls
4. Post-remediation audit confirms the score improvement

**Rollback:** Per-step rollback restores the previous state for each remediated control.

## Evidence Collection

**Change type:** `collect_evidence`

Collects configuration files and command outputs relevant to a specific compliance control (SOC2, PCI DSS, ISO 27001) and packages them as a downloadable ZIP.

**Connector:** Nexplane Agent (`compliance` package)

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `control_id` | Compliance control identifier (e.g., `soc2-cc6.1`, `pci-dss-10.2`) |
| `target_hosts` | List of target asset IDs |

**Output:** Downloadable ZIP containing config files and command output, timestamped and linked to the authorizing change request.

**API:** `GET /compliance/evidence/{id}/download`

See [Compliance & Governance](../features/compliance.md) for drift detection and change freeze windows.
