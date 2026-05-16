# Projects

Projects group related change requests into a sequenced, dependency-linked security initiative. A project models the full execution plan for a goal — not just a list of CRs, but the order they must run and what each one depends on.

## What Projects Do

- **Dependency tracking** — declare that CR B cannot execute until CR A completes; Nexplane enforces this automatically
- **Visual DAG** — a dependency graph (React Flow + Dagre auto-layout) shows the project structure and execution state at a glance
- **Sequenced execution** — CRs in a project are executed in dependency order; independent CRs can run in parallel

## AI Planning Assistant

Available on draft projects. Opens a conversational panel where an operator describes their security goal in plain language. The AI proposes a structured change plan referencing your actual asset inventory.

Example: "Deploy EDR to all EC2 instances not running CrowdStrike" → AI proposes a `deploy_nexplane_agent` CR for each matching asset, sequenced correctly.

**Supported AI providers:**

- Anthropic Claude (default: `claude-sonnet-4-6`)
- OpenAI (configurable in Settings)

Configure the provider and API key in **Settings → AI Providers**. If AI is not configured, the panel shows inline guidance with a `402` response.

## Project States

| State | Description |
|-------|-------------|
| Draft | Being planned; AI assistant available |
| Active | Execution in progress |
| Completed | All CRs completed |
| Blocked | A dependency CR failed |

## Seeded Example Projects

The demo environment includes 10 pre-seeded example projects covering real security workflows:

1. Isolate and Investigate Compromised Endpoint
2. Deploy EDR to Unprotected Hosts
3. Remediate Critical CVE Across Fleet
4. Offboard Departed Employee
5. Remediate Public Azure Storage
6. Tighten Firewall After Vulnerability Scan
7. MFA Enforcement for Non-Compliant Accounts
8. Microsegmentation for Payments Subnet
9. Harden Payments-API Workload Isolation
10. Identify and Respond to Firewall Chokepoints
