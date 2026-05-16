# Credential Change Types

Credential changes rotate, revoke, or replace secrets, API keys, and passwords. These are among the most common changes in Nexplane, and every credential change type is designed with rollback in mind.

## Rotate IAM Access Key (AWS)

Creates a new IAM access key for a user, stores it encrypted in the change record, and deactivates the old key. The old key is not deleted immediately -- it remains in a deactivated state so it can be re-activated during rollback.

**Connector:** AWS

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| IAM User | string | Username of the IAM user |

**Execution:**
1. Calls `iam:CreateAccessKey` to create a new key
2. Stores the new key ID and secret (encrypted) in the change record
3. Calls `iam:UpdateAccessKey` with status `Inactive` on the old key

**Rollback:**
1. Re-activates the old key (`UpdateAccessKey` with status `Active`)
2. Deactivates the new key
3. Deletes the new key after confirming the old key is active

**Risk base score:** 5 (medium -- application downtime if consumers are not updated)

!!! warning "Key limit"
    AWS limits IAM users to 2 access keys. If the user already has 2 active keys, rotation will fail with `LimitExceeded`. Delete one key before rotating.

---

## Rotate Service Account Key (GCP)

Creates a new key for a GCP service account, stores it encrypted, and deletes the old key.

**Connector:** GCP

**Rollback:** GCP does not support re-creating a deleted key. Rollback disables the new key but cannot restore the old one. After rollback, you must create a new key manually.

**Risk base score:** 5

---

## Rotate Client Secret (Azure / Keycloak)

Generates a new client secret for an Azure service principal or Keycloak confidential client.

**Connectors:** Azure, Keycloak

**Rollback:** The old secret cannot be retrieved from Azure or Keycloak after a new one is created. Nexplane does not store the old plaintext secret. Rollback is not available for client secret rotation.

**Risk base score:** 6 (medium-high -- applications using the old secret will break immediately)

---

## Rotate KV Secret (Vault)

Writes a new version of a Vault KV v2 secret with updated values.

**Connector:** HashiCorp Vault

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| Secret Path | string | KV v2 path (e.g., `secret/myapp/db`) |
| Key | string | Key within the secret to update |
| New Value | string | New value (encrypted at rest in Nexplane) |

**Rollback:** Restores the previous version of the secret using Vault's versioning.

**Risk base score:** 5

---

## Rotate Database Password (PostgreSQL / MongoDB / Redis)

Updates the password for a database user account.

**Connectors:** PostgreSQL, MongoDB, Redis

**Rollback:** Nexplane does not store the old plaintext password. Rollback is not available. Set the password to a known value if you need to recover access.

**Risk base score:** 5-7 depending on the account's privilege level

---

## Rotate Local Password (SSH / WinRM)

Sets a new password for a local OS user account on a Linux or Windows host.

**Connectors:** SSH, WinRM

**Rollback:** Not available -- old password is not stored.

**Risk base score:** 4

---

## Revoke Exposed Credential

Immediately revokes a credential that has been identified as exposed (e.g., committed to a public repository). This is a high-urgency action that bypasses the normal approval queue in environments where the "emergency bypass" policy is enabled.

**Supported connectors:** AWS (IAM access key), GCP (service account key), Azure (client secret), LDAP (user password reset to random), Vault (token revocation)

**Rollback:** Not available -- revocation is intentionally irreversible.

**Risk base score:** 9 (high -- immediate service impact is expected and acceptable)
