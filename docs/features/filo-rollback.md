# FILO Rollback Stack

Nexplane enforces First-In Last-Out (FILO) ordering when rolling back a sequence of change requests. If you applied change A and then change B, rollback unwinds B before A. This ordering guarantee is not advisory — the platform refuses out-of-order rollback by default.

## Why Order Matters

Many infrastructure changes create dependencies that break if unwound in the wrong order:

- **CA rotations** — rolling back a CA rotation before rolling back the changes that deployed certificates signed by the new CA leaves hosts with certificates they can no longer validate
- **Credential rotations** — rolling back a downstream service's credential update before rolling back the rotation that issued the new credential leaves the service unable to authenticate
- **Firewall changes** — rolling back an allow rule before rolling back the services that depend on it causes an outage during the rollback window

Enforcing FILO order ensures each rollback step lands in a state where its preconditions still hold.

## How It Works

Every change request carries an `application_sequence` field — an integer assigned by the platform at execution time within a project. When a rollback is requested, the platform checks whether any CR with a higher `application_sequence` in the same project is still in an applied state. If so, the platform blocks the rollback and surfaces a divergence warning listing the CRs that must be rolled back first.

The `application_sequence` is set automatically. You do not configure it manually.

## Divergence Warnings

If an operator requests an out-of-order rollback, the platform returns an error response with the following structure:

```json
{
  "error": "out_of_order_rollback",
  "message": "CR-42 must be rolled back before CR-39.",
  "blocking_crs": ["CR-42"]
}
```

The UI surfaces this as a warning banner on the rollback confirmation dialog, listing each blocking CR with a direct link.

To proceed, the operator must first rollback the blocking CRs, then return to roll back the original CR.

## Project Rollback

The `project_rollback` change type unwinds every applied CR in a project in strict FILO sequence. It is the recommended path for full project reversal — for example, after a failed migration or a go-back decision during a maintenance window.

`project_rollback` iterates the project's applied CRs in descending `application_sequence` order, executing each CR's rollback phase before moving to the next. Any individual rollback failure halts the project rollback and requires operator intervention before the sequence can resume.

See [Projects](../concepts/projects.md) for how CRs are grouped into projects and how project rollback is triggered.
