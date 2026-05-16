# Nexplane Agent

The Nexplane Agent is a cross-platform Go binary that runs on managed machines and executes commands dispatched by the control plane. It reverses the connection direction — the agent calls out to the control plane, so no inbound SSH, VPN, or firewall rules are required on managed hosts.

## Platforms

| Platform | Architectures |
|----------|--------------|
| Linux | amd64, arm64 |
| Windows | amd64 |

## Distribution

Binaries are published to a public S3 bucket after each build:

```
https://nexplane-agent-downloads.s3.us-east-1.amazonaws.com/
  nexplane-agent-linux-amd64-{VERSION}
  nexplane-agent-linux-arm64-{VERSION}
  nexplane-agent-windows-amd64-{VERSION}.exe
  + .sha256 sidecar for each binary
  version  (plain text: current version string)
```

## How It Works

1. The agent starts and registers itself at `POST /agent/register`
2. The managed machine appears as an Asset in Nexplane inventory automatically
3. The agent enters a long-poll loop on `GET /agent/jobs/next`
4. When a job arrives, the agent verifies the HMAC-SHA256 signature
5. The agent executes the command and posts the result to `POST /agent/result`
6. The agent immediately polls again

## Security

- **HMAC-SHA256 job signing** — every job payload is signed with the agent HMAC secret. Jobs with invalid signatures are rejected before execution.
- **Typed commands only** — all permitted operations are registered at compile time. No arbitrary shell execution.
- **Outbound only** — the agent never opens a listening port. All communication is outbound HTTP.

## Command Packages

The agent supports 15 command packages covering patching, hardening, IaC, fleet operations, forensics, compliance, database administration, and more. See [Command Packages](commands.md).

## Self-Update

On startup the agent checks S3 for a newer version, downloads and verifies it, and atomically replaces itself. See [Self-Update](self-update.md).
