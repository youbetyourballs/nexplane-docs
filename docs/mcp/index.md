# MCP Server

Nexplane exposes a Model Context Protocol (MCP) server at `/mcp` that lets AI agents — Claude, Cursor, or any MCP-compatible client — read infrastructure state, create change requests, and monitor execution results. The server implements the MCP SSE (Server-Sent Events) transport.

Every MCP tool operates within the caller's organization. All proposed changes go through the normal CR lifecycle — the AI proposes, a human approves, and only then does the platform execute. The approval gate is enforced for every AI-assisted action without exception.

## Connecting

### Agent Token

All MCP tools require a `token` parameter — a Nexplane agent token. Tokens are created by an admin via the API:

```http
POST /auth/agent-tokens
Authorization: Bearer <user-jwt>
Content-Type: application/json

{
  "name": "claude-desktop",
  "scopes": ["read", "write"]
}
```

The response contains the token value. Store it — it is shown only once.

!!! note "Admin required"
    Only users with the `admin` role can create agent tokens. Token scope can be further restricted by `allowed_connector_types`, `allowed_asset_tags`, `allowed_cr_types`, and `allowed_roles`.

### Claude Desktop

Add Nexplane to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "nexplane": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sse", "http://localhost:8000/mcp"]
    }
  }
}
```

### Claude Code

```bash
claude mcp add nexplane --transport sse http://localhost:8000/mcp
```

## Discovery Pattern

Before creating a change request, agents should follow this discovery sequence:

1. `list_connectors` — confirm the needed connector is configured and active
2. `list_assets` / `search_assets` / `get_asset` — resolve the target asset and its context
3. `list_catalog_actions(connector_type)` / `list_change_types` — find the exact action or CR type
4. `create_change_request` — propose the change

Use planning context tools (`get_fleet_context`, `get_asset_history`, `get_cross_host_dependency_map`) to fill parameters from infrastructure state rather than asking the operator.

## Tool Domains

| Domain | Tools | Description |
|--------|-------|-------------|
| [Assets & Connectors](assets-connectors.md) | 14 | Enumerate and inspect infrastructure assets and connector status |
| [Change Requests](change-requests.md) | — | Create, approve, execute, and roll back change requests |
| [Findings & Identity](findings-identity.md) | — | Manage vulnerability findings and identity access |
| [Host Intelligence](host-intelligence.md) | — | Live host interrogation via the Nexplane Agent |
| [Projects, Planning & Migration](projects-planning.md) | — | Multi-CR project orchestration, migration profiling, reference scanning, runbooks |
