# Agent Self-Update

The Nexplane Agent checks for updates on every startup and replaces itself automatically when a newer version is available.

## How It Works

1. **Fetch version** — agent downloads the `version` file from S3:
   ```
   https://nexplane-agent-downloads.s3.us-east-1.amazonaws.com/version
   ```
   The file contains the current version string (e.g., `0.1.0`).

2. **Compare** — if the S3 version is newer than the running binary's compiled-in version, the agent proceeds with the update.

3. **Download** — the agent downloads the platform-appropriate binary:
   ```
   nexplane-agent-linux-amd64-{VERSION}
   nexplane-agent-linux-arm64-{VERSION}
   nexplane-agent-windows-amd64-{VERSION}.exe
   ```

4. **Verify** — the agent downloads the `.sha256` sidecar and verifies the SHA256 checksum of the downloaded binary. If verification fails, the download is discarded and the agent continues running the current version.

5. **Atomic replace (Linux)** — the verified binary is written to a temp file and atomically renamed over the current binary with `os.Rename`. The agent then re-executes itself with `syscall.Exec`, replacing the current process with the new binary with no downtime.

6. **Windows** — atomic re-exec is not supported on Windows. The agent logs that a new version is available and requires a manual restart or Windows Service Manager restart to apply the update.

## S3 Cache Headers

| Object | Cache-Control |
|--------|--------------|
| `version` file | `max-age=60` (re-fetched frequently) |
| Binary files | `immutable, max-age=31536000` (binaries never change for a given version) |

## Self-Hosted Distribution

Set `NEXPLANE_AGENT_DOWNLOAD_URL` in the backend environment to point to your own distribution server. The agent uses this URL (injected at registration time) for version checks and downloads. Your server must serve:

- `version` — plain text version string
- `nexplane-agent-{os}-{arch}-{version}[.exe]` — binary
- `nexplane-agent-{os}-{arch}-{version}[.exe].sha256` — SHA256 checksum

## Build and Upload

```bash
# Build the Docker image (compiles agent for all platforms)
docker compose build backend

# Upload binaries to S3
./scripts/upload-agent-to-s3.sh
```
