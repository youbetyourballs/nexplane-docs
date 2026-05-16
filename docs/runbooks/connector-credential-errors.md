# Runbook: Connector Credential Errors

Use this runbook when:
- The **Test Connection** button returns an error
- Asset discovery fails with an authentication error
- A change request fails during execution with a credential-related error

---

## Step 1: Identify the Error

Click **Test Connection** on the connector row in **Settings > Connectors**. The error message will indicate the cause.

Common error patterns:

| Error Message | Likely Cause |
|---|---|
| `AuthFailure: AWS was not able to validate the provided access credentials` | Wrong or expired AWS access key |
| `InvalidClientTokenId: The security token included in the request is invalid` | Access key ID is incorrect or deleted |
| `LDAP error: Invalid credentials (49)` | Wrong bind password |
| `LDAP error: No such object (32)` | Base DN does not exist in the directory |
| `Connection refused` | Wrong host, port, or firewall blocking the connection |
| `x509: certificate has expired or is not yet valid` | Expired TLS certificate on the target system |
| `google.auth.exceptions.TransportError` | GCP service account key is invalid or has been deleted |
| `ClientAuthenticationError` | Azure client secret is wrong or expired |

---

## AWS Credential Errors

### "AuthFailure" or "InvalidClientTokenId"

The access key is invalid. Verify:

1. The access key ID and secret key are copied correctly (no trailing spaces)
2. The key is in **Active** status in the AWS console (IAM > Users > Security Credentials)
3. The key belongs to the correct AWS account

To update credentials: click **Edit** on the connector in the Nexplane UI and re-enter the key ID and secret.

### "AccessDenied"

The key is valid but lacks permissions for the operation. Check the minimum permissions listed in [AWS Connector](../connectors/aws.md#minimum-permissions-required) and compare against the IAM policy attached to the user.

---

## LDAP Credential Errors

### "Invalid credentials (49)"

The bind password is incorrect. Verify by testing the bind manually:

```bash
ldapwhoami -H ldaps://dc01.corp.example.com:636 \
  -D "CN=nexplane,CN=Users,DC=corp,DC=example,DC=com" \
  -W
```

If this fails, the password is wrong or the account is locked.

### "No such object (32)"

The Base DN or Bind DN does not exist. Verify the DN using an LDAP browser (Apache Directory Studio, ldapsearch):

```bash
ldapsearch -H ldaps://dc01.corp.example.com:636 \
  -D "CN=nexplane,CN=Users,DC=corp,DC=example,DC=com" \
  -W -b "DC=corp,DC=example,DC=com" -s base "(objectClass=*)"
```

### "Connect error" or "Connection refused"

- Verify the server URL is correct (include `ldaps://` for LDAP over TLS)
- Verify port 636 (LDAPS) or 389 (LDAP) is open from the Nexplane host to the LDAP server
- If using a self-signed LDAP certificate, provide the CA certificate in the connector configuration

---

## GCP Credential Errors

### "google.auth.exceptions.TransportError" or "invalid_grant"

The service account key JSON is invalid, expired, or the service account has been deleted.

Verify the service account exists:

1. Go to GCP console > IAM & Admin > Service Accounts
2. Find the service account email from the key JSON (`client_email` field)
3. Check that it exists and is not disabled
4. Check that the key ID (`private_key_id`) appears in the service account's Keys tab

If the key has been deleted or the service account is gone, create a new key and update the connector.

---

## Azure Credential Errors

### "ClientAuthenticationError: AADSTS7000215"

The client secret is incorrect or has expired. Azure client secrets have configurable expiry dates.

1. Go to Azure AD > App Registrations > [your app] > Certificates & Secrets
2. Check the expiry date of the current secret
3. If expired, create a new secret and update the connector

---

## HashiCorp Vault Errors

### "permission denied"

The Vault token lacks the required policy for the operation. Check the policies attached to the token:

```bash
vault token lookup <token>
```

Compare the listed policies against the minimum permissions in [Vault Connector](../connectors/vault.md#minimum-permissions-required).

### "token expired"

The Vault token TTL has elapsed. Generate a new token with appropriate policies and update the connector.

---

## PostgreSQL / MongoDB / Redis Errors

### "password authentication failed"

The database password has changed since the connector was configured. Update the connector credentials with the current password.

### "could not connect to server: Connection refused"

- Verify the host and port
- Check that the database is running
- Check that the database's `pg_hba.conf` (Postgres) or `bindIp` (MongoDB) allows connections from the Nexplane host
