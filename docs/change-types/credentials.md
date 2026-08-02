# Credential Change Types

Credential changes rotate, revoke, or replace secrets, API keys, and passwords. Credentials generated during execution travel in memory between steps only — never written to logs or the database.

## Key Rotation

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `key_rotation` | Generic key rotation orchestrator — routes to the correct rotation handler based on asset type | Restores previous key state |
| `key_pair_create` | Create an EC2 key pair | Delete the key pair |
| `key_pair_delete` | Delete an EC2 key pair | Not available |

## DB Credential Rotation

**Change type:** `rotate_db_credentials`

1. Generate a new password
2. Update the database user's password (PostgreSQL, MySQL, or MSSQL)
3. Update application config files on the target host(s) via the Nexplane Agent
4. Restart the dependent service
5. Verify the database connection with the new credentials

**Rollback:** Restore original password and config, restart service.

## SSH Key Fleet Rotation

**Change type:** `rotate_ssh_keys`

1. Remove the old key by fingerprint from `authorized_keys` across the target fleet
2. Add the new public key to all hosts
3. Back up the original `authorized_keys` for rollback

**Rollback:** Restore `authorized_keys` from backup on all hosts.

## API Key Rotation

**Change type:** `rotate_api_key`

Rotates AWS IAM access keys, Okta API tokens, or GitHub Personal Access Tokens. Propagates the new key to Kubernetes secrets, agent env files, or SSM Parameter Store before deactivating the old key.

**Rollback:** Re-activate old key, remove new key from propagation targets.

## Service Account Rotation

**Change type:** `rotate_service_account`

Updates a service account password in AD and/or Okta, then pushes the new password to dependent services via asset metadata.

## S3 Bucket Operations

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `s3_bucket_create` | Create an S3 bucket | Delete the bucket |
| `s3_bucket_delete` | Delete an S3 bucket | Not available |
| `s3_lifecycle_configure` | Configure lifecycle rules on an S3 bucket | Restore previous lifecycle configuration |

**Connector:** AWS

## Route53 DNS

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `route53_zone_create` | Create a hosted zone | Delete the zone |
| `route53_record_upsert` | Create or update a DNS record | Delete or restore previous record |
| `route53_record_delete` | Delete a DNS record | Recreate the deleted record |

**Connector:** AWS

---

## End-to-End Rotation with Consumer Fan-Out

**Change type:** `credential_rotation_fanout`

Rotates a credential at its source and automatically updates all infrastructure components that reference it — closing the gap between "the key was rotated" and "everything using the key has the new value."

1. Rotate (optional) — call the specified rotate action at the source (IAM access key, Vault secret, database password). If the rotate action returns a `new_value`, that value is used for all subsequent consumer updates. This phase is skipped if no `rotate` spec is provided.
2. Scan — search for consumers across the configured scopes. Supported surfaces: AWS Lambda environment variables, ECS task definition environment variables, SSM parameters, Kubernetes ConfigMaps, and Kubernetes Deployment environment variables.
3. Update — push the new credential value to each consumer in order, building a FILO rollback stack as updates are applied. If any consumer update returns an error, the CR transitions to `paused` immediately — remaining consumers are not updated. The partial state is preserved for operator review.
4. Verify — re-scan all configured scopes to confirm the old credential value is no longer present. If any consumer still holds the old value, the CR transitions to `paused`.

**Rollback:** Unwind consumers in reverse (FILO) order using the `rollback_data` captured during each update. Consumers that were not successfully updated are skipped. After all consumers are unwound, the source credential is restored via the rotate action's rollback path. If rollback fails for any consumer, the execution result records the failure so operators can restore the remaining consumers manually.

**Connector:** AWS (source credential and AWS-surface consumers) + Kubernetes (k8s-surface consumers)

---

## Certificate Rotation

**Change type:** `step_ca_rotate_cert`

Reissues a TLS certificate from a step-ca certificate authority and optionally deploys the new certificate to a target host.

1. Issue — connect to the step-ca CA and issue a new certificate for the specified subject and SAN with the configured validity period (default: 720 hours / 30 days). The certificate and private key are written to a secure temporary directory.
2. Deploy (optional) — if `deploy_via_ssm` is enabled and an `instance_id` is provided, write the new certificate and key to the target EC2 instance via SSM Run Command. The key is written with mode `0600`; the certificate with mode `0644`. After writing, the configured `reload_command` is executed (default: `nginx -s reload`).

**Rollback:** Not available. Once a certificate is issued by the CA, the old certificate is expired or superseded. If the newly issued certificate must be invalidated, revoke it via `step-ca revoke` out of band. The executor's rollback function records this explicitly and returns `rolled_back: false`.

**Connector:** step-ca (certificate issuance) + AWS SSM (optional host deployment)
