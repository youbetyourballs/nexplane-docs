# Composable Runbooks

Runbooks are user-defined, reusable multi-step workflows that chain existing change types with conditional logic. A runbook is a template — you define it once and execute it many times against different targets.

## Step Types

| Step Type | Description |
|-----------|-------------|
| `change` | Creates a change request of the specified type against a target |
| `condition` | Branches on the result of a previous step (success/failure/output value) |
| `human_checkpoint` | Pauses execution and waits for an operator to confirm before continuing |
| `parallel_group` | Runs a set of steps concurrently |

## Failure Handling

Each step has a configurable `on_failure` policy:

| Policy | Behavior |
|--------|----------|
| `abort` | Stop execution immediately; leave completed steps in place |
| `continue` | Log the failure and move to the next step |
| `rollback_all` | Trigger rollback of all completed steps in reverse order |

## Versioning

Each time a runbook is edited, a new version is created. Active executions keep a snapshot of the version they started with — editing a runbook does not affect in-progress runs. You can fork a runbook to create a new one based on an existing version.

## Seed Templates

Nexplane ships with three seed runbook templates:

**Engineer Onboarding**
Provisions accounts across all connected identity systems (AD, Okta, Entra ID, GitHub, Slack, Google Workspace) from a single form, in the correct order with human checkpoints between phases.

**Account Compromise IR**
Locks down a compromised account across all identity systems, preserves forensic evidence, forces MFA re-enrollment, and notifies the security team — as a single tracked, audited workflow.

**Patch Campaign**
Audits patch status across a fleet, schedules a maintenance window, applies patches in rolling batches, and verifies service health after each batch.

## API

| Endpoint | Description |
|----------|-------------|
| `GET /runbooks/` | List all runbooks |
| `POST /runbooks/` | Create a runbook |
| `GET /runbooks/{id}` | Get runbook with version history |
| `POST /runbooks/{id}/trigger` | Execute a runbook |
| `POST /runbooks/{id}/fork` | Fork a runbook to a new one |
| `GET /executions/{id}` | Get execution status |
| `POST /executions/{id}/resume` | Resume at a human checkpoint |
| `POST /executions/{id}/abort` | Abort an in-progress execution |
