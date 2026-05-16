# Change Requests

A change request (CR) is the atomic unit of work in Nexplane. Every infrastructure action — rotating a key, isolating a host, updating a firewall rule — is modeled as a structured change request with a defined target, parameters, safety review, approval gate, execution, and typed rollback.

## Lifecycle

```
Draft → Planned → Awaiting Approval → Approved → Executing → Verifying → Completed
                                                                        ↘ Rolled Back
                                                                        ↘ Failed
```

| State | Description |
|-------|-------------|
| Draft | Created, not yet submitted |
| Planned | Safety review passed, submitted for approval |
| Awaiting Approval | Waiting for required approvers |
| Approved | All approvals received |
| Executing | Connector or agent is running the change |
| Verifying | Post-execution check running |
| Completed | Change applied successfully |
| Rolled Back | Change reversed via rollback |
| Failed | Execution error; rollback may be available |

## Safety Review

Before a CR can be submitted for approval, the Safety Engine evaluates it:

- **Risk score** — calculated from change type inherent risk, asset environment (prod vs staging), blast radius (how many systems depend on this asset), and rollback availability
- **Blocking conditions** — some scenarios prevent submission entirely (e.g., a critical-asset change with no rollback strategy defined, or a freeform shell command)

See [Safety Engine](../security/safety-engine.md) for the full ruleset.

## Approval Tiers

| Risk Level | Score | Approval Required |
|------------|-------|------------------|
| Low | 1–3 | None (auto-approved in non-prod) |
| Medium | 4–6 | 1 approver |
| High | 7–8 | 1 approver (approver or admin role) |
| Critical | 9–10 | 2 approvers (approver + admin) |

Approval policies are configurable per organization.

## Rollback

Every change type defines its own inverse operation. Rollback is a first-class action: it goes through the same approval flow as the original change and produces its own audit log entry.

Rollback availability is shown on every CR. Some change types explicitly mark `rollback_supported: false` (e.g., `promote_db_replica`) and display a blast radius warning at approval time.

## Audit Trail

Every state transition is recorded immutably with a timestamp, the acting user, and the full request/response payload. Audit events are queryable via `GET /audit-events/`.

## Change Freeze

If a change freeze is active, the approve and execute endpoints return `423 Locked`. Emergency bypass requires the `ir_responder` role, a mandatory justification header, and produces an audit log entry. See [Compliance & Governance](../features/compliance.md).
