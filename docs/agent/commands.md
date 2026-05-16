# Agent Commands

The `nexplane-agent` binary provides a small CLI for installation, status, and diagnostics.

## install

Install the agent as a system service.

```
nexplane-agent install [flags]
```

| Flag | Default | Description |
|---|---|---|
| `--control-plane` | (required) | URL of the Nexplane control plane (e.g., `https://nexplane.example.com:8000`) |
| `--token` | (required) | One-time enrollment token from the Nexplane UI |
| `--poll-interval` | `5s` | How often to poll the control plane for new tasks |
| `--log-level` | `info` | Log verbosity: `debug`, `info`, `warning`, `error` |
| `--config-dir` | OS default | Directory to write config and certificates |

**Example:**

```bash
nexplane-agent install \
  --control-plane https://nexplane.example.com:8000 \
  --token eyJhbGci... \
  --poll-interval 10s \
  --log-level info
```

---

## uninstall

Remove the agent service and all agent files (config, certificates).

```
nexplane-agent uninstall
```

Requires root or Administrator. Does not prompt for confirmation -- run with care.

---

## status

Print the agent status: registration state, last heartbeat, pending tasks.

```
nexplane-agent status
```

Example output:

```
Agent status: registered
Control plane: https://nexplane.example.com:8000
Agent ID: agt_a1b2c3d4
Hostname: web-server-01
OS: linux/amd64
Agent version: 0.1.0
Last heartbeat: 3s ago
Pending tasks: 0
```

---

## version

Print the agent version.

```
nexplane-agent version
```

---

## run (foreground mode)

Run the agent in the foreground without installing as a service. Useful for testing or debugging.

```
nexplane-agent run --config /etc/nexplane-agent/config.yaml
```

Ctrl+C stops the agent. This does not affect the installed service if one exists.

---

## check

Verify connectivity to the control plane and validate the agent certificate without making any changes.

```
nexplane-agent check
```

Example output:

```
Checking control plane connectivity... OK
Checking mTLS certificate... OK (expires 2027-05-16)
Checking agent registration... OK (registered as web-server-01)
All checks passed.
```

If any check fails, the error message and suggested fix are printed.

---

## rotate-cert

Request a new client certificate from the control plane. Use if the current certificate is about to expire or has been revoked.

```
nexplane-agent rotate-cert
```

This generates a new key pair, sends a CSR to the control plane, and replaces the existing certificate. The old certificate is deleted. A new enrollment token is not needed -- the existing certificate is used to authenticate the rotation request.
