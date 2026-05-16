# Network Exposure

## Backend API

The Nexplane backend API binds to `127.0.0.1:8000` in the default Docker Compose configuration. It is **not exposed to the public internet**.

The frontend (port 3000) is the only service that should be externally accessible. The frontend proxies API calls to the backend on localhost. If you deploy Nexplane on a server, put the frontend behind a reverse proxy (nginx, Caddy, etc.) with TLS — do not expose the backend API port publicly.

## Frontend

The React frontend serves on port 3000. In production, place it behind a TLS-terminating reverse proxy. Set `CORS_ORIGINS` in the backend environment to your actual frontend origin.

## Agent Communication

The Nexplane Agent communicates **outbound only**:

- The agent polls `GET /agent/jobs/next` (long-poll)
- The agent posts results to `POST /agent/result`
- No inbound network access is required on managed hosts
- No SSH port, no VPN, no firewall exceptions needed on the managed host

The control plane needs to be reachable from the agent (outbound HTTP/HTTPS from the agent to the control plane URL). The agent does not need to be reachable from the control plane.

## Agent Binary Distribution

Agent binary distribution is handled entirely by S3 (public read), not by the Nexplane backend. Managed machines download directly from S3 — the backend is never a download proxy. This means:

- The backend does not need internet access to serve agent binaries
- Binary distribution scales independently of the backend
- S3 cache headers ensure binaries are cached at the edge (1-year immutable cache for binaries, 60s cache for the `version` file)

## Summary

| Component | Bind Address | Externally Accessible |
|-----------|-------------|----------------------|
| Backend API | `127.0.0.1:8000` | No |
| Frontend | `0.0.0.0:3000` | Yes (behind reverse proxy + TLS in production) |
| PostgreSQL | Internal Docker network | No |
| Agent → Control Plane | Outbound from agent | N/A (outbound only) |
| Agent binaries | S3 public read | Yes (S3 CDN) |
