# Runbook: Agent Registration Failure

Use this runbook when:
- The agent was installed but does not appear in **Settings > Agents** after 60 seconds
- An agent shows status "offline" after previously being registered
- The agent log shows connection or certificate errors

---

## Symptom 1: Agent not appearing after installation

### Check: Is the agent service running?

**Linux:**
```bash
systemctl status nexplane-agent
```

If the service is not running:
```bash
journalctl -u nexplane-agent -n 50
```

**Windows:**
```powershell
Get-Service NexplaneAgent
Get-EventLog -LogName Application -Source NexplaneAgent -Newest 20
```

**macOS:**
```bash
sudo launchctl list | grep nexplane
log show --predicate 'subsystem == "ai.nexplane.agent"' --last 30m
```

### Check: Can the agent reach the control plane?

```bash
curl -v https://nexplane.example.com:8000/health
```

If this fails:
- Verify the `--control-plane` URL is correct (check `/etc/nexplane-agent/config.yaml`)
- Check that the host can reach the control plane port (firewall rules, security groups)
- Verify DNS resolves correctly: `nslookup nexplane.example.com`

### Error: "enrollment token already used"

```
FATAL: enrollment failed: token has already been used
```

Enrollment tokens are single-use. Generate a new token in **Settings > Agents > New Enrollment Token** and re-run the install command:

```bash
nexplane-agent install \
  --control-plane https://nexplane.example.com:8000 \
  --token NEW_TOKEN
```

### Error: "x509: certificate signed by unknown authority"

```
FATAL: enrollment failed: Post "https://nexplane.example.com:8000/agent/enroll": 
  x509: certificate signed by unknown authority
```

The control plane is using a self-signed or private CA certificate. Pass the CA certificate to the agent:

```bash
nexplane-agent install \
  --control-plane https://nexplane.example.com:8000 \
  --token YOUR_TOKEN \
  --ca-cert /path/to/ca.crt
```

Or for testing only, skip TLS verification (not recommended for production):

```bash
nexplane-agent install \
  --control-plane https://nexplane.example.com:8000 \
  --token YOUR_TOKEN \
  --insecure-skip-verify
```

### Error: "clock skew too large"

```
FATAL: enrollment failed: certificate validation error: certificate not yet valid
```

The agent host's clock is more than 5 minutes out of sync with the control plane. Synchronize NTP:

```bash
# Linux
timedatectl set-ntp true
timedatectl status

# Windows
w32tm /resync /force
```

---

## Symptom 2: Agent shows "offline" after being registered

### Check: Is the agent service still running?

Run the status commands from Symptom 1.

### Check: Has the client certificate expired?

```bash
nexplane-agent check
```

If the certificate is expired:

```bash
nexplane-agent rotate-cert
```

This requests a new certificate using the existing (expired) one as proof of identity. If the certificate is too expired for the control plane to accept, generate a new enrollment token and reinstall:

```bash
nexplane-agent uninstall
nexplane-agent install --control-plane ... --token NEW_TOKEN
```

### Check: Was the agent deleted from the UI?

If an agent is deleted from **Settings > Agents**, its registration is revoked. The agent will log:

```
ERROR: heartbeat rejected: agent not found or revoked
```

Re-enroll with a new token.

---

## Symptom 3: Agent registered but tasks are not executing

### Check: Is the change request approved?

Tasks are only dispatched to agents after the change request is approved. Verify the change request status in the UI.

### Check: Is the correct agent selected?

On the change request detail page, verify that the target agent matches the registered agent's hostname.

### Check: Agent logs for task errors

```bash
# Linux
journalctl -u nexplane-agent -n 100 | grep -i "task\|error\|fail"
```

Common task errors and fixes are in the [change type documentation](../change-types/index.md).
