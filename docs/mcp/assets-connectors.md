# Assets & Connectors

## Assets

These tools enumerate and inspect infrastructure assets across all connected systems. Use them to answer "what do we have?" and to gather the context needed before proposing a change. `list_assets` and `search_assets` return summary fields; call `get_asset` or `get_asset_context` for full detail before creating a CR.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `list_assets` | Enumerate infrastructure assets across connected systems. Filter by type, environment, criticality, or connector. Returns summary fields — use `get_asset` for full detail. | `asset_type`, `environment`, `criticality`, `connector_id`, `limit` |
| `get_asset` | Get full context for an asset including owner, connector source, and recent changes. Use before creating a CR to understand what you're touching. | `asset_id` |
| `get_asset_context` | Get a full planning context bundle for an asset: recent CRs, open findings, timeline, and connector. Use before creating a CR to ensure the plan is appropriate for this asset. | `asset_id` |
| `list_asset_findings` | List open security findings for an asset. Use to identify what needs remediation before or after a change. Filter by status or severity. | `asset_id`, `status`, `severity`, `limit` |
| `get_asset_timeline` | Get ordered change and event history for an asset. Use to answer: what changed recently and who approved it? Returns the last N timeline events including CR executions, finding ingests, and scan events. | `asset_id`, `limit` |
| `search_assets` | Search assets by name substring or tag value. Use to find assets when you don't know the exact ID. Returns the same fields as `list_assets`. | `query`, `limit` |
| `get_asset_neighbors` | Get the immediate relationship neighborhood of an asset: upstream (what this asset depends on) and downstream (what depends on this asset). Use before planning changes to understand blast radius. | `asset_id` |
| `get_asset_upstream` | BFS traversal of all assets this asset transitively depends on. `max_depth` controls how many hops to follow. Returns each node with id, name, asset_type, relationship_type, direction, and depth. | `asset_id`, `max_depth` |

!!! note "Asset IDs"
    Always resolve an asset ID via `search_assets` or `list_assets` before creating a CR. Passing an unknown or incorrect asset ID to a CR will cause a planning error.

## Connectors

These tools inspect connector health, retrieve connector detail, and discover the catalog actions available for each connector type. Connector credentials are never returned by any tool.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `list_connectors` | List all connectors configured for the org with type and status. Optionally filter by connector type or active-only. Credentials are never returned. | `connector_type`, `enabled_only` |
| `get_connector` | Get connector detail including type, status, and scoped permissions. Credentials are never returned. | `connector_id` |
| `test_connector` | Test connectivity for a connector. Returns success/failure and the connector's current status. Does not modify the connector record. | `connector_id` |
| `get_connector_status` | Get the current operational status of a connector (active, error, inactive, pending). | `connector_id` |
| `list_connector_change_types` | List catalog actions available for a connector identified by its UUID. Use `list_catalog_actions` instead when you already know the connector type string. | `connector_id` |
| `list_catalog_actions` | List all catalog actions available for a connector type (e.g. `aws`, `gcp`, `okta`, `kubernetes`). Returns `action_id`, display name, description, and parameter schema for each action. Use this to discover valid `action_id` values before creating a `catalog_action` CR. | `connector_type` |

!!! note "Discovering actions"
    Use `list_catalog_actions(connector_type)` as the primary discovery tool when you know the connector type. Use `list_connector_change_types(connector_id)` when you only have the connector UUID. The two tools cover the same catalog — choose based on what you have available.
