# IP Migration

Nexplane orchestrates IP address changes on live hosts with automatic rollback safety. The `change_ip` change type is dispatched to the Nexplane Agent, which captures a full network snapshot before making any change and restores it on rollback.

## Methods

Nexplane applies the lowest-risk method available at execution time. Set `method: auto` (the default) to let Nexplane choose.

| Method | When Used | Connectivity Guarantee |
|--------|-----------|------------------------|
| `tailscale` | Tailscale is active on the host | Agent stays reachable via Tailscale overlay; physical IP change is transparent |
| `secondary_swap` | Secondary IP can be assigned to the ENI/NIC | Old IP stays active until new IP is confirmed; two-phase commit |
| `commit_timer` | Tailscale not available; commit timer safety net required | Agent probes control plane after change; auto-rolls back if unreachable within the timer window |
| `manual` | Planned maintenance with human confirmation | Change applied; operator confirms before it is committed |

## Dead Man's Switch (commit_timer)

Before applying the change, the agent writes `pending_rollback.json` to disk. A background goroutine probes the control plane (`GET /health`) every few seconds.

- If the probe **succeeds** within the timer window: `pending_rollback.json` is deleted, the change is committed.
- If the probe **never succeeds** (agent loses network access, or agent crashes and restarts): `pending_rollback.json` is detected at startup and the original network configuration is restored automatically.

The timer window is configurable per CR (`commit_timer_seconds`, default 30s, range 10–300s).

## Multi-Host Campaigns

**Change type:** `ip_campaign`

Orchestrates IP changes across a fleet:

- `batch_size` — how many hosts to change per batch
- `abort_error_threshold` — if the error rate across a batch exceeds this value, the campaign halts and rolls back completed hosts

## DNS Coordination

**Change type:** `migrate_ip`

Handles DNS TTL-aware ordering:

- **Short TTL (≤120s):** single CR updates both the IP and DNS record together
- **Long TTL (>120s):** two-CR sequence — `prepare_dns` lowers the TTL first, then `migrate_ip` changes the IP once propagation is confirmed

## IP Migration Wizard (UI)

Available on server and endpoint asset detail pages via the **Change IP** button in the Network section. Guides the operator through four steps:

1. **Configure** — interface, new IP, gateway, DNS servers, method, commit timer
2. **Pre-flight** — live dry-run checks via a CR; results displayed before execution
3. **Execute** — stage progress and commit timer countdown
4. **Verify** — connectivity confirmation and rollback button

## Rollback

`change_ip_rollback` restores all pre-change state from the network snapshot: IP addresses, gateway, routes, DNS servers, and MTU. Rollback can be triggered:

- Manually from the CR detail page
- Automatically by the dead man's switch timer
- Programmatically via `client.rollback_cr()` in the smoke test suite
