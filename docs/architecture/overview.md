# Architecture Overview

Nexplane is a three-layer system: a control plane, a connector library, and a host agent. Each layer has a clear responsibility and communicates with the others through well-defined interfaces.

## Layers

```
+---------------------------+
|       React Frontend      |  Port 3000
+---------------------------+
            |
            | HTTPS / REST + WebSocket
            |
+---------------------------+
|     FastAPI Backend       |  Port 8000
|  (Control Plane API)      |
+---------------------------+
       |          |
       |          +-------------------+
       |                              |
+------+----------+    +--------------+------+
|   PostgreSQL    |    |     Connector Layer  |
|   (State Store) |    | (AWS/GCP/SSH/LDAP/..) |
+-----------------+    +---------------------+
                                  |
                       +----------+----------+
                       |   Go Agent (hosts)  |
                       |  Linux/Windows/macOS|
                       +---------------------+
```

## Control Plane

The control plane is the authoritative source of truth for all change requests, asset inventory, connector configurations, and audit logs. It is stateless between requests -- all state lives in PostgreSQL.

Key responsibilities:

- Authenticating users and agents
- Storing encrypted connector credentials
- Orchestrating change request lifecycle (draft -> approved -> executing -> complete)
- Scoring change risk
- Persisting audit logs for every state transition

See [Control Plane](control-plane.md) for implementation details.

## Connector Layer

Connectors are Python modules that wrap external system APIs. Each connector declares:

- What credentials it needs (stored encrypted in the backend)
- What actions it supports (mapped to change types)
- What the rollback for each action looks like

Connectors run inside the backend process. They are not separate services. The connector library currently includes over 70 connectors across cloud, identity, secrets, database, network, and host categories.

See [Connectors](../connectors/index.md) for the full list.

## Agent

The agent is a statically compiled Go binary. It runs as a system service on managed hosts and handles local operations that connectors cannot reach through a cloud API.

The agent uses a typed command allowlist -- it does not accept arbitrary shell commands. Every operation the agent can perform is defined at compile time and validated before execution.

See [Agent](agent.md) for implementation details.

## Change Request Lifecycle

Every change in Nexplane follows the same lifecycle:

```
Draft -> Submitted -> Approved -> Executing -> Complete
                                           -> Failed
                                           -> RolledBack
```

Each state transition is logged with a timestamp, the acting user or system, and the full payload. Transitions are append-only -- no state is ever deleted.

## Network Traffic

| Path | Protocol | Authentication |
|---|---|---|
| Browser -> Backend | HTTPS | JWT (cookie or header) |
| Backend -> Connectors | Cloud SDK / HTTPS | Encrypted connector credentials |
| Agent -> Backend | mTLS | Client certificate issued at enrollment |
| Backend -> Agent | via Agent polling | mTLS |

The agent initiates all connections to the backend -- the backend never connects outbound to agents. This means agents work behind NAT and in private networks without firewall changes.
