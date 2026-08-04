# Santa (macOS Allowlisting)

Nexplane includes a built-in Santa sync server. macOS hosts enroll by pointing their Santa configuration at `https://<nexplane-host>/santa/sync`. No separate sync server is required — Nexplane handles sync requests directly and maintains the rule database for enrolled hosts.

Santa enforces binary allowlisting and blocklisting on macOS based on SHA-256 hash or Apple Developer TeamID. Rules pushed from Nexplane are applied to enrolled hosts on the next sync cycle.

!!! warning "MDM enrollment required"
    Santa requires a SystemExtension MDM profile to load on macOS 11 and later. This profile must be pushed via an MDM provider before Nexplane can manage Santa rules on a host. Nexplane supports Jamf and MicroMDM for this purpose. See [macOS Agent](../agent/macos.md) for enrollment steps.

## Credential Fields

None — Santa enrollment uses the agent token configured in **Settings → Agent Configuration**. Hosts authenticate to the Santa sync endpoint using the same HMAC-signed agent token as the Nexplane Agent.

## Ingest

When a macOS host enrolls in the Santa sync server, Nexplane ingests:

- Host identity (hardware UUID, hostname, OS version, Santa version)
- Current rule set applied to the host
- Binary allow and block list entries (hash and TeamID rules)

Ingest runs on every sync cycle. The sync interval is configured on the host's Santa configuration (`SyncBaseURL` and `ClientAuthCertificateFile` entries in `/etc/santa/santad.conf`).

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `santa_rule_add` | Add a binary allow or block rule by SHA-256 hash or Apple Developer TeamID | Remove the rule |
| `santa_rule_remove` | Remove an existing allow or block rule | Restore the rule from the pre-change snapshot |
| `santa_sync_force` | Force a full rule sync on one or more enrolled hosts | N/A |

### santa_rule_add

Adds a new rule to the Santa rule database and propagates it to all enrolled hosts in scope on the next sync. Rules are applied by SHA-256 hash (for specific binaries) or by TeamID (for all binaries signed by a given Apple developer certificate).

**Parameters:** `rule_type` (`binary` or `teamid`, required), `identifier` (SHA-256 hash or TeamID string, required), `policy` (`ALLOWLIST` or `BLOCKLIST`, required), `custom_message` (string, optional — displayed to the user when a blocked binary is denied)

**Rollback:** Removes the rule from the database and forces a sync to enrolled hosts.

### santa_rule_remove

Removes an existing rule from the Santa rule database and propagates the removal to all enrolled hosts in scope on the next sync.

**Parameters:** `rule_type` (`binary` or `teamid`, required), `identifier` (required)

**Rollback:** Restores the rule from the pre-change snapshot and forces a sync.

### santa_sync_force

Forces an immediate full rule sync on one or more enrolled hosts. Use this to push rule changes without waiting for the next scheduled sync interval.

**Parameters:** `asset_ids` (array of asset IDs, optional — if omitted, syncs all enrolled hosts)

**Rollback:** Not applicable.
