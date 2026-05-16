# Safety Engine

The Safety Engine evaluates every change request before it can be submitted for approval. It applies a set of rules to assign a risk score and block dangerous configurations unconditionally.

## Risk Score Calculation

The final risk score for a change request is calculated from:

1. **Base risk score** — defined per change type (e.g., `ec2_terminate` = 9, `snapshot_asset` = 2)
2. **Environment multiplier** — production assets score higher than staging
3. **Blast radius** — how many systems depend on the target asset
4. **Rollback availability** — changes without rollback score higher

## Blocking Rules

The following scenarios block submission regardless of risk score:

| Scenario | Behavior |
|----------|----------|
| Critical-asset change with no rollback strategy | Blocked — cannot generate plan |
| Remote command without an approved template | Blocked at safety review |
| Freeform shell command | Blocked unconditionally |
| Agent job payload with invalid HMAC | Rejected before execution |
| IaC apply without prior plan review | Blocked — plan must exist before apply |
| DB replica promotion | Marked `rollback_supported: false`; blast radius warning required at approval |

## Risk Level Actions

| Score | Level | Default Behavior |
|-------|-------|-----------------|
| 1–3 | Low | Auto-approved in non-production environments |
| 4–6 | Medium | Requires 1 approver |
| 7–8 | High | Requires 1 approver (approver or admin role) |
| 9–10 | Critical | Requires 2 approvers (approver + admin) |

## Special Cases

**Microsegmentation policies:** Always run in staged simulation mode. The safety engine blocks execution on production assets unless explicit simulation confirmation is provided in the CR parameters.

**AI provider not configured:** The AI planning assistant returns a `402` response and the UI shows inline configuration guidance. No safety bypass — the assistant simply isn't available.

**Active change freeze:** The approve and execute endpoints return `423 Locked`. Emergency bypass requires the `ir_responder` role plus a mandatory `X-Bypass-Justification` header. All bypass attempts are recorded in the audit trail whether successful or not.

**Step credential output:** Credentials produced during multi-step execution (new passwords, rotated keys) travel in process memory only. They are never written to logs, the database, or temporary files. The only stored artifact is the encrypted rollback snapshot of the pre-change state.
