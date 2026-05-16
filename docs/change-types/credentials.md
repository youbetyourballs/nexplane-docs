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
