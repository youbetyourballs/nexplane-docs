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

## End-to-End Rotation with Consumer Fan-Out

**Change type:** `credential_rotation_fanout`

Rotating a secret at its source — an IAM key, Vault secret, or database password — doesn't automatically update every service that was using it. `credential_rotation_fanout` closes that gap by discovering and updating all consumers in the same request.

After rotating the credential at the source (optionally, via a configured rotate action), the executor scans across AWS and Kubernetes surfaces to find every component holding the old value: Lambda environment variables, ECS task definitions, SSM parameters, Kubernetes ConfigMaps, and Kubernetes Deployments. It then updates each consumer with the new credential in order, building a FILO rollback stack as it goes.

If a consumer update fails, the CR automatically pauses — the operator can review the error, fix the failing consumer (manually or by retrying), and resume. If verification detects that any consumer still holds the old credential after all updates complete, the CR pauses again until those stragglers are resolved. The rollback stack unwinds consumers in reverse order. Source credential restoration is not performed automatically — restore the original credential at the source manually if needed.

See [credential_rotation_fanout](../change-types/credentials.md#end-to-end-rotation-with-consumer-fan-out) for the full phase reference.

## Certificate Rotation

**Change type:** `step_ca_rotate_cert`

Rotates TLS certificates issued by a step-ca certificate authority, with optional automated deployment to EC2 instances.

The executor connects to a step-ca CA and issues a new certificate for the specified subject and subject alternative names (SAN) with a configurable validity period (default: 720 hours / 30 days). Once issued, the certificate and private key are held securely in a temporary directory. If `deploy_via_ssm` is enabled and an `instance_id` is provided, the executor uses AWS Systems Manager to write the new certificate and key to the target host, sets proper file permissions (0644 for certificate, 0600 for key), and optionally triggers a reload command (default: `nginx -s reload`).

**Important note:** Certificate rotation is not reversible once issued. The old certificate expires or is superseded by the new one, and the platform has no way to "un-issue" a certificate. If the newly issued certificate must be invalidated before expiry, revoke it using `step-ca revoke` out of band. The executor's rollback function records this limitation explicitly.

See [step_ca_rotate_cert](../change-types/credentials.md#certificate-rotation) for parameters.
