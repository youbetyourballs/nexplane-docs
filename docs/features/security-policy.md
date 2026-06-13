# Security Policy Auto-Generation

Writing least-privilege host security policies by hand is slow and error-prone. Nexplane generates them from **observed behavior** instead: agents watch how an asset actually behaves during a soak window, a per-backend plugin synthesizes a candidate policy, and you review the diff before it becomes an enforced, tracked change request.

This is driven by the `SecurityPolicyEngine` and its **soak session** lifecycle.

## Soak Session Lifecycle

```
Observe → Synthesize → Diff → Accept
```

1. **Observe** — start a session (`POST /security-policy/soak-sessions`). Agents collect runtime behavior from the target assets over the soak window.
2. **Synthesize** — when the window closes, a per-backend plugin turns the observations into a candidate policy.
3. **Diff** — review the proposed policy against the current baseline (`GET /security-policy/soak-sessions/{id}/diff`).
4. **Accept** — approve the diff (`POST /security-policy/soak-sessions/{id}/accept`). This emits a hardening change request and persists a new baseline.

The synthesized diff **must be reviewed and accepted** before any policy is enforced — observation alone never changes a host.

## Policy Backends

The engine ships pluggable policy backends, each synthesizing a different kind of host policy:

| Backend | Generates |
|---------|-----------|
| `seccomp` | seccomp syscall-filter profiles |
| `apparmor` | AppArmor profiles |
| `selinux` | SELinux policy modules |
| `ebpf_lsm` | eBPF LSM (Linux Security Module) policies |
| `ebpf_network` | eBPF network policies |

These map to the agent's `ossecurity` and `ebpf` command packages, which apply the resulting profiles on the host.

## Per-Project Baselines

Baselines are tracked per project so that re-observed behavior is always diffed against the **last accepted** policy rather than a blank slate:

| Endpoint | Purpose |
|----------|---------|
| `GET /security-policy/baselines/{project_id}` | Fetch the current accepted baseline for a project |
| `DELETE /security-policy/baselines/{project_id}` | Reset the baseline |

When a new soak session runs, its synthesized policy is diffed against this baseline, so you only review what changed.

## Soak Periods in Projects

Project phases support a configurable **soak period** (`soak_hours`, default **72h**). Staged rollouts auto-advance to the next phase only after the observation window closes clean — a phase that surfaces unexpected behavior holds until reviewed.

## In the UI

The **Security Policy Soak** panel surfaces:

- Active soak sessions and their remaining window
- The live diff between the synthesized policy and the current baseline
- The **Accept** action that emits the hardening change request

## Safety

Policy enforcement is gated end-to-end:

- A policy is enforced **only after** the soak window closes.
- The synthesized diff must be **reviewed and accepted** first.
- Acceptance produces a normal hardening change request, which flows through the standard [Safety Engine](../security/safety-engine.md) approval path.

## See also

- [Agent Command Packages](../agent/commands.md) — `ossecurity` and `ebpf` packages
- [Compliance & Governance](compliance.md)
- [Hardening change types](../change-types/hardening.md)
