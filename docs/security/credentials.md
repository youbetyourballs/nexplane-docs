# Credential Storage

This page describes how Nexplane stores and protects connector credentials.

## Overview

Connector credentials (AWS access keys, LDAP bind passwords, SSH private keys, etc.) are sensitive. Nexplane stores them encrypted at rest and decrypts them only in memory during connector operations. Plaintext credentials never touch disk and are never returned by the API.

## Encryption Scheme

Nexplane uses AES-256-GCM (Galois/Counter Mode) for credential encryption.

**Key derivation:**

The encryption key is derived from the `SECRET_KEY` environment variable using PBKDF2-HMAC-SHA256:

```
encryption_key = PBKDF2(
    password = SECRET_KEY,
    salt = "nexplane-credential-key-v1",
    iterations = 100000,
    dklen = 32
)
```

**Per-credential encryption:**

Each credential value is encrypted independently with a unique 96-bit (12-byte) IV:

```
iv = random_bytes(12)
ciphertext, tag = AES-256-GCM.encrypt(
    key = encryption_key,
    nonce = iv,
    plaintext = credential_value,
    aad = connector_id  # binds ciphertext to a specific connector
)
blob = base64(iv + ciphertext + tag)
```

The `aad` (additional authenticated data) parameter binds each ciphertext to its connector ID. Swapping a ciphertext between connectors will cause decryption to fail, preventing a class of attacks where an attacker moves ciphertexts between records.

## Storage

The encrypted blob is stored in the `credentials` column of the `connectors` table as a text field. PostgreSQL itself performs no encryption of this column -- the encryption is entirely at the application layer.

For defense in depth, enable PostgreSQL at-rest encryption (e.g., AWS RDS encryption, encrypted EBS volume) in addition to Nexplane's application-layer encryption.

## Key Management

**Nexplane does not manage the encryption key.** The key is provided via the `SECRET_KEY` environment variable. You are responsible for:

- Generating a strong key: `python3 -c "import secrets; print(secrets.token_hex(32))"`
- Storing it in a secrets manager (AWS Secrets Manager, HashiCorp Vault, etc.) rather than in plaintext in a `.env` file
- Rotating the key periodically (see Key Rotation below)
- Not committing the key to source control

## Key Rotation

To rotate the encryption key:

1. Set the new key in `NEW_SECRET_KEY` environment variable
2. Run the key rotation migration:

   ```bash
   docker compose exec backend python -m nexplane.tools.rotate_encryption_key \
     --old-key $SECRET_KEY \
     --new-key $NEW_SECRET_KEY
   ```

   This decrypts all credential blobs with the old key and re-encrypts them with the new key in a single transaction. If it fails midway, the original credentials are unchanged.

3. Update `SECRET_KEY` to the new value and restart the backend.

## What Is and Is Not Encrypted

**Encrypted:**
- All connector credential values (passwords, API keys, private keys, tokens)

**Not encrypted (but stored in the database):**
- Connector metadata (name, type, host, username, region)
- Asset metadata (IDs, names, tags)
- Change request records and audit logs
- User email addresses and bcrypt-hashed passwords

The metadata is considered lower-sensitivity than the credentials themselves. If your threat model requires encrypting metadata as well, enable full-database encryption at the PostgreSQL or infrastructure layer.

## Credential Access Logging

Every time a connector credential is decrypted (for a Test Connection, discovery run, or change execution), the event is recorded in the audit log with:

- Timestamp
- Actor (user or system task)
- Connector ID
- Operation type

This allows you to audit all access to connector credentials over time.
