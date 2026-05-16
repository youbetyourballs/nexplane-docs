# Tailscale Connector

The Tailscale connector enables secure mesh networking as a tracked Nexplane change. It installs Tailscale on EC2 instances via SSM and manages node membership in your tailnet.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Auth Key | Yes | A **reusable, pre-authorized** Tailscale auth key (`tskey-auth-...`) |
| Tailnet | No | Tailnet name — for display |

!!! warning "Use a reusable key"
    The auth key must be **reusable** and **pre-authorized**. Single-use keys are consumed on the first node join and break subsequent runs. Pre-authorized means no manual approval step in the Tailscale admin console is needed.

## Capabilities

### `tailscale_join`

Installs Tailscale on an EC2 instance via SSM and joins the tailnet with the configured auth key. Returns the instance's Tailscale IP after joining.

**Flow:**
1. Download and install the Tailscale package via SSM
2. Run `tailscale up --authkey <key> --hostname <instance-id>`
3. Verify the node appears in the tailnet
4. Record the Tailscale IP in the change result and update the asset

### `tailscale_remove`

Gracefully removes a node from the tailnet:
1. Run `tailscale logout` on the instance via SSM
2. Remove the node from the tailnet via the Tailscale API

**Rollback:** `tailscale_join` — re-join the node to the tailnet.

## How the Backend Uses Tailscale

During smoke tests, the Nexplane backend container itself joins the tailnet (using kernel TUN mode with `/dev/net/tun`). This allows EC2 instances that have joined via `tailscale_join` to reach the control plane over the Tailscale overlay network — which is required for the `change_ip` dead man's switch probe to succeed.
