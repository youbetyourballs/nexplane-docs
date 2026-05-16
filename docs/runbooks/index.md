# Runbooks

This section contains step-by-step troubleshooting guides for common Nexplane operational issues.

## Available Runbooks

| Runbook | When to Use |
|---|---|
| [Agent Registration Failure](agent-registration-failure.md) | Agent does not appear in the UI after installation, or shows as "offline" |
| [Connector Credential Errors](connector-credential-errors.md) | Test connection fails, discovery returns errors, or change execution fails with auth errors |
| [Rollback Failure](rollback-failure.md) | A rollback operation fails or is not available for a completed change |
| [Upgrading](upgrade.md) | Upgrading Nexplane to a new version |

## General Troubleshooting Tips

**Check the backend logs first.**

Most errors are explained in the backend container logs:

```bash
docker compose logs -f backend
```

**Check the change request detail page.**

Failed change requests and failed rollbacks include a detailed error message and stack trace on the change detail page. This is usually the fastest path to the root cause.

**Use the health endpoint.**

```bash
curl http://localhost:8000/health
```

The health endpoint reports database connectivity and the current version. If the database is unreachable, most operations will fail.

**Check the audit log.**

Every state transition is logged in the audit log. Go to **Settings > Audit Log** to see a full history of what happened and when.
