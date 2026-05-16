# Identity Lifecycle

Nexplane manages the full lifecycle of user identities across all connected identity systems — from onboarding through offboarding — as audited, reversible change requests.

## User Offboarding (Kill Switch)

A single `offboard_user` change request atomically disables a departing employee across all connected identity systems in five ordered phases:

| Phase | Actions |
|-------|---------|
| 1. Disable accounts | AD, Okta, Entra ID, Google Workspace simultaneously |
| 2. Revoke access | GitHub org membership, Slack deactivation |
| 3. Revoke sessions | Active sessions across all identity providers |
| 4. Rotate credentials | Force-rotate any API keys or service account passwords owned by the user |
| 5. Endpoint isolation | Optionally isolate CrowdStrike-managed endpoints associated with the user |

Each phase completes before the next begins. Per-system results are recorded in the change request detail view.

## User Onboarding

A single `onboard_user` change request provisions an account across all connected identity systems from a single form:

- Creates the user in AD and/or Okta and/or Entra ID
- Creates GitHub org membership
- Creates Google Workspace account
- Adds the user to configured default groups

Provisioning order follows configured dependencies (e.g., AD first, then Okta sync, then downstream systems).

## Access Reviews

Periodic campaigns collect current group memberships across all identity connectors and present them for manager review:

1. **Collect** — query all identity connectors for current memberships
2. **Review** — present memberships to reviewers; each access item is marked `retain` or `revoke`
3. **Remediate** — auto-generate `remove_from_groups` or `disable_user` change requests for all revoked items
4. **Approve and execute** — standard CR approval flow applies

Access reviews are launched from **Access Reviews** in the sidebar or via `POST /access-reviews/`. Review campaigns have a configurable deadline after which unreviewed items are escalated.
