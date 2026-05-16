# Credential Storage

Nexplane uses `SecretsService` — a versioned, swappable encryption layer — to protect all connector credentials and sensitive configuration at rest.

## Encryption

**Algorithm:** Fernet (symmetric encryption, AES-128-CBC + HMAC-SHA256). The `cryptography` Python library implementation.

**Key derivation:** The encryption key is derived from the `SECRET_KEY` environment variable. Set this to a strong random value in production:

```bash
python3 -c "import secrets; print(secrets.token_hex(32))"
```

## Versioning

`SecretsService` maintains multiple key versions with a 7-day TTL on old versions:

1. When a new `SECRET_KEY` is configured, a new key version is generated
2. Existing secrets are re-encrypted with the new key version on first access
3. Old key versions are retained for 7 days so existing tokens and sessions remain valid
4. After 7 days, old versions are purged

This allows zero-downtime key rotation without invalidating all active sessions simultaneously.

## What Is Encrypted

- Connector credentials (API keys, passwords, private keys, service account JSONs)
- Agent HMAC secret
- AI provider API keys
- Webhook secrets

## What Is Never Stored Plaintext

- Credential values are encrypted immediately on receipt and never written to disk, logs, or returned in API responses
- Credentials generated during execution steps (new passwords, rotated keys) travel in process memory between steps only and are never written to the database

## Abstraction for Vault / HSM

`SecretsService` is designed with a swappable backend interface. The Fernet implementation can be replaced with:

- **HashiCorp Vault** — store and retrieve secrets from Vault KV v2; key rotation handled by Vault
- **AWS Secrets Manager** — native AWS secret management
- **HSM** — hardware security module for regulated environments

The connector code has no knowledge of which backend is in use — it only calls `secrets_service.encrypt()` and `secrets_service.decrypt()`.
