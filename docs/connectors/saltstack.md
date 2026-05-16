# SaltStack

The SaltStack connector manages Salt minions via the Salt API.

## Credential Fields

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name |
| Salt API URL | Yes | e.g. `https://salt-master:8080` |
| Username | Yes | Salt API username |
| Password | Yes | Salt API password |
| EAuth Method | No | Authentication backend (default: pam) |

## Ingest

Discovers Salt minions (ID, OS, IP, kernel) and recent Salt jobs.

## Capabilities

| Action | Description | Rollback |
|--------|-------------|---------|
| `run_state` | Apply a Salt state to target minions (async) | N/A |
| `run_function` | Run an allowlisted Salt function (`sys.doc`, `test.ping`, `pkg.list_pkgs`, `service.status`) | N/A |
| `sync_minion` | Sync all Salt modules and states to target minions | N/A |
| `accept_key` | Accept a pending minion key | Reject the key |
| `reject_key` | Reject or delete a minion key | N/A |
