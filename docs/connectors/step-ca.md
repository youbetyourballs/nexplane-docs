# step-ca (Smallstep)

The step-ca connector manages TLS certificate issuance and rotation via a Smallstep step-ca certificate authority.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| CA URL | Yes | e.g. `https://ca.example.com:9000` |
| Root CA Fingerprint | Yes | SHA-256 fingerprint from `step ca bootstrap` |
| Provisioner Name | No | JWK provisioner name (default: admin) |
| Provisioner Password | No | Provisioner password |
| step CLI Path | No | Path to the `step` CLI binary (default: `step`) |

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `rotate_certificate` | Reissue a TLS certificate for a domain; optionally deploy via SSM and trigger a service reload | N/A |
| `check_expiry` | Check remaining TLS certificate validity days for a host:port endpoint (read-only; warns when under threshold) | N/A |
