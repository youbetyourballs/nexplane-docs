# Secret & Credential Rotation

Nexplane orchestrates credential rotation as tracked, approval-gated change requests. Credentials travel in memory between execution steps only — they are never written to logs or the database.

## DB Credential Rotation

**Change type:** `rotate_db_credentials`

1. Generate a new password
2. Update the database user's password
3. Update the application config file(s) on the target host(s)
4. Restart the dependent service
5. Verify the database connection with the new credentials
6. Rollback: restore the original password and config, restart service

Supports PostgreSQL, MySQL, and MSSQL. The Nexplane Agent executes steps 3–5 on the host.

## SSH Key Fleet Rotation

**Change type:** `rotate_ssh_keys`

1. Identify the old key by fingerprint in `authorized_keys` across the target fleet
2. Remove the old key from all hosts
3. Add the new public key to all hosts
4. Back up the original `authorized_keys` for rollback

The old key material is captured in the rollback snapshot and restored if the CR is rolled back.

## API Key Rotation

**Change type:** `rotate_api_key`

Supports rotating:

- **AWS IAM access keys** — create new key, update dependent resources (Kubernetes secrets, agent env files, SSM Parameter Store), disable old key
- **Okta API tokens** — generate new token, propagate to configured destinations, revoke old token
- **GitHub Personal Access Tokens** — generate new PAT, propagate, revoke old token

## Service Account Rotation

**Change type:** `rotate_service_account`

1. Update the service account password in AD and/or Okta
2. Push the new password to dependent services via asset metadata
3. Verify service connectivity

## Security Model

Credentials generated or discovered during rotation steps are held in process memory only. They are passed between executor steps in-process and never written to the database, logs, or temporary files. The only storage is the encrypted rollback snapshot (which stores the pre-rotation state, not the new credential).
