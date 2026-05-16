# Runbook: Upgrading Nexplane

Use this runbook when upgrading Nexplane to a new version.

---

## Before You Start

1. **Read the release notes** for the version you are upgrading to. Check for breaking changes, required migration steps, or new environment variables.

2. **Back up the database.** Upgrades run database migrations automatically. If a migration fails, you will need the backup to recover.

   ```bash
   # Using pg_dump (adjust connection details)
   pg_dump -h localhost -U nexplane nexplane > nexplane-backup-$(date +%Y%m%d).sql
   ```

3. **Note the current version.**

   ```bash
   curl http://localhost:8000/health | python3 -m json.tool
   ```

4. **Notify users.** The upgrade requires a brief restart. Active change executions will be interrupted and should be re-run after the upgrade.

---

## Upgrade Steps

### 1. Pull the new images

```bash
docker compose pull
```

### 2. Stop the services

```bash
docker compose stop backend worker
```

Do not stop the database (`db`) or Redis -- they do not need to be restarted during a backend upgrade.

### 3. Start the new backend

```bash
docker compose up -d backend worker
```

On startup, the backend automatically runs any pending Alembic migrations. Watch the logs:

```bash
docker compose logs -f backend
```

You should see:

```
INFO  alembic.runtime.migration - Running upgrade <old_rev> -> <new_rev>, <description>
INFO  uvicorn.main - Application startup complete.
```

If migrations fail, the backend will exit. See [Migration Failure](#migration-failure) below.

### 4. Upgrade the frontend

```bash
docker compose pull frontend
docker compose up -d frontend
```

### 5. Verify the upgrade

```bash
curl http://localhost:8000/health
```

Confirm the version field reflects the new version.

Open the UI and verify you can log in and load the asset inventory.

### 6. Upgrade agents (if required)

Check the release notes for the new version. If the release notes indicate an agent upgrade is required:

1. Download the new agent binary for each platform
2. Stop the agent service on each host
3. Replace the binary
4. Start the agent service

The agent version is shown in **Settings > Agents** for each registered agent. Nexplane logs a warning (but does not block operation) when an agent is running an older version than the control plane.

---

## Migration Failure

If the database migration fails during startup, the backend will print an error and exit:

```
ERROR alembic.runtime.migration - Can't locate revision identified by 'abc123def456'
```

This usually means the database is on a newer version than the code (downgrade attempted) or the migration history is corrupted.

**Recovery steps:**

1. Restore the database backup taken before the upgrade:

   ```bash
   psql -h localhost -U nexplane -d nexplane < nexplane-backup-YYYYMMDD.sql
   ```

2. Roll back the Docker images to the previous version:

   ```bash
   docker compose down
   # Edit docker-compose.yml to pin to the previous image tag
   docker compose up -d
   ```

3. File an issue at github.com/youbetyourballs/nexplane with the full migration error output.

---

## Rollback Plan

If the upgrade causes unexpected problems:

1. Stop the new containers: `docker compose stop backend worker frontend`
2. Restore the database from backup
3. Edit `docker-compose.yml` to pin `image:` tags to the previous version
4. Start: `docker compose up -d`

Nexplane images are tagged by version. If you were running `0.2.0` before and upgraded to `0.3.0`, pin back to `youbetyourballs/nexplane-backend:0.2.0`.
