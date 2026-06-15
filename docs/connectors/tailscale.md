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

## Granting a Collaborator VPN Access to the Dev Instance

To let an "interested party" reach the dev instance over Tailscale, issue them their **own** per-person auth key from the credentials already stored on the Tailscale connector — don't reshare one key. The repeatable tool is `backend/scripts/grant_vpn_access.py`.

!!! info "Prerequisite: issuing credentials"
    Minting individual keys requires **OAuth client credentials** (`oauth_client_id` + `oauth_client_secret`) or a Tailscale **API access token** (`api_key`) on the connector. A single stored reusable `auth_key` is for *machine* enrollment (`tailscale_join`) and cannot — and should not — be used to grant people access. The tool refuses that case. Also define the device tag (default `tag:collaborator`) in your tailnet ACL and make the OAuth client an owner of it.

Run it on the dev instance (or anywhere `DATABASE_URL`, `SECRET_KEY`, and `SECRET_BACKEND` match the dev environment), from the `backend/` directory:

```bash
# Grant Lucas Nelson access (single-use, non-ephemeral, tag:collaborator, 90-day expiry)
python -m scripts.grant_vpn_access "Lucas Nelson" --email lucas@example.com

# Tighter expiry / custom tag
python -m scripts.grant_vpn_access "Lucas Nelson" --expiry-days 30 --tag tag:collaborator

# Identify the dev-instance node to share with them
python -m scripts.grant_vpn_access --list-tailnet
```

The tool prints a one-time auth key, the exact `tailscale up --authkey …` command for the collaborator to run, and the dev-instance node(s) they can then reach. Each key's description records **who** it was issued to and **when**, so access stays auditable.

**Secure defaults** (override with flags):

| Property | Default | Why |
|----------|---------|-----|
| Reusable | `false` | One key enrolls one device — no sharing |
| Ephemeral | `false` | The collaborator's device persists across reconnects |
| Pre-authorized | `true` | No manual approval step in the admin console |
| Tag | `tag:collaborator` | Access is governed by ACLs on the tag |
| Expiry | 90 days | Access is time-boxed, not permanent |

**Revoking access:** delete the device at [tailscale.com/admin/machines](https://login.tailscale.com/admin/machines), or expire the key at [tailscale.com/admin/settings/keys](https://login.tailscale.com/admin/settings/keys).
