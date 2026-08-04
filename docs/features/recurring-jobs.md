# Recurring Jobs

Recurring Jobs are APScheduler-backed change request triggers that fire on a cron schedule. Each tick creates a new change request in the normal lifecycle — plan, approve, execute — so recurring operations remain auditable and subject to the same approval gates as manually created CRs.

## How It Works

When a scheduled job fires, the platform creates a CR with the configured `change_type` and `parameters`, runs the plan phase, and places the CR in `awaiting_approval`. An operator approves the CR before execution proceeds. No recurring job executes autonomously.

If a job fires while a previous tick's CR is still open, the platform skips the new tick and logs a warning rather than stacking multiple open CRs for the same job.

## Job Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Human-readable job name |
| `cron_expression` | string | Standard five-field cron expression (e.g. `0 2 * * *`) |
| `change_type` | string | The change type ID to execute (e.g. `ssh_ca_rotation`) |
| `parameters` | object | Parameters passed to the change type executor |
| `enabled` | boolean | Whether the job is active; set to `false` to pause without deleting |

## Common Use Cases

**Scheduled SSH CA rotation** — rotate the fleet SSH certificate authority monthly:

```json
{
  "name": "Monthly SSH CA Rotation",
  "cron_expression": "0 3 1 * *",
  "change_type": "ssh_ca_rotation",
  "parameters": { "key_type": "ed25519" },
  "enabled": true
}
```

**Nightly IAM baseline scan** — audit AWS IAM roles every night and generate remediation proposals:

```json
{
  "name": "Nightly IAM Baseline",
  "cron_expression": "0 1 * * *",
  "change_type": "aws_iam_role_baseline",
  "parameters": { "stale_threshold_days": 90, "generate_remediations": true },
  "enabled": true
}
```

**Weekly privileged account audit** — query Domain Admins and flag stale or MFA-less accounts every Monday:

```json
{
  "name": "Weekly Privileged Account Audit",
  "cron_expression": "0 6 * * 1",
  "change_type": "privileged_account_audit",
  "parameters": { "stale_threshold_days": 90, "generate_remediations": true },
  "enabled": true
}
```

## Managing Recurring Jobs

Recurring jobs are managed via the **Scheduled Operations** page in the Nexplane UI or via the API:

- `GET /recurring-jobs/` — list all jobs
- `POST /recurring-jobs/` — create a job
- `PUT /recurring-jobs/{id}/` — update a job or toggle `enabled`
- `DELETE /recurring-jobs/{id}/` — delete a job
