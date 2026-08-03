# Container Operations

Container operations cover ECS service deployments and cross-cloud container image transfers.

## Container Image Transfer

**Change type:** `container_image_transfer`

Pulls a container image from one cloud registry and pushes it to another, with full rollback. Supports any combination of AWS ECR, Azure ACR, GCP Artifact Registry, and OCI OCIR as source or destination.

**Transfer paths:**

- **Agent docker** (default) — Nexplane agent on a Linux host performs `docker pull`, `docker tag`, and `docker push`. Used for all source/destination combinations except Azure-destination.
- **ACR import API** — Azure's server-side import API pulls the image directly into ACR without routing through the Nexplane agent. Used automatically when the destination is Azure ACR.

**Phases:**

1. Preflight — verify source image and tag exist; check destination for pre-existing tag; fail fast if `overwrite_existing` is false and the tag is already present
2. Snapshot — record whether the destination tag exists and, if so, its current digest (for rollback restoration)
3. Transfer — execute the appropriate transfer path; verify the transferred digest matches the source
4. Verify — confirm the destination tag now exists and the digest is correct
5. Report — summarise the transfer: method used, digest matched, bytes transferred

**Rollback:**

| Scenario | Rollback action |
|----------|----------------|
| Net-new tag (destination tag did not exist before transfer) | Delete the destination tag |
| Overwrote existing tag | Restore the previous digest by re-tagging from the original digest reference |
| GC edge case (original digest no longer in registry) | Partial rollback — tag deleted but original cannot be restored |

**Parameters:**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `source_connector_id` | Yes | Connector UUID for the source registry |
| `dest_connector_id` | Yes | Connector UUID for the destination registry |
| `source_image` | Yes | Source image path (e.g., `nexplane-smoke/alpine`) |
| `source_tag` | Yes | Source tag (e.g., `3.19`) |
| `dest_image` | Yes | Destination image path |
| `dest_tag` | Yes | Destination tag |
| `overwrite_existing` | No | Allow overwriting an existing destination tag (default: `false`) |
| `agent_asset_id` | No | Asset ID of a Linux host to use for agent-docker transfers; auto-selected if omitted |

**Connector type:** `container_registry` (use `list_catalog_actions("container_registry")` to discover)

---

## ECS Rolling Deploy

**Change type:** `ecs_rolling_deploy`

Registers a new ECS task definition revision (image tag change, environment variable update, or both), drives a rolling service update, and polls health gates before completing. If any health gate fails, the service is automatically rolled back to the previous task definition revision.

**Phases:**

1. Preflight — validate the ECS service and cluster exist; record the current task definition ARN for rollback
2. Register — create a new task definition revision with the specified image tag and/or environment variable changes; `old_value` is verified against the live value before the update is applied
3. Deploy — call `UpdateService` with the new task definition
4. Stability poll — wait for `runningCount == desiredCount` with a single active deployment
5. ALB/NLB health gate *(optional)* — wait for all targets in the specified target group to report healthy
6. HTTP probe gate *(optional)* — GET the specified health check URL and verify the expected HTTP status code

If phase 4, 5, or 6 fails, the executor automatically calls `UpdateService` back to the previous task definition ARN.

**Parameters:**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `service_arn` | Yes | ECS service ARN or name |
| `cluster` | Yes | Cluster name or ARN |
| `image_tag` | No* | Full image string (e.g., `nginx:1.27`) |
| `env_var_overrides` | No* | Array of `{key, old_value, new_value}` — `old_value` is verified against the live value before the update |
| `container_name` | No | Required when `image_tag` is set and the task definition has more than one container |
| `target_group_arn` | No | Enables ALB/NLB health gate |
| `health_check_url` | No | Enables HTTP probe gate |
| `health_check_expected_status` | No | Expected HTTP status code (default: 200) |
| `stability_timeout_seconds` | No | Stability poll timeout in seconds (default: 300) |
| `health_timeout_seconds` | No | Timeout for ALB and HTTP gates, applied independently to each (default: 60) |
| `region` | No | AWS region; defaults to connector credential region |

*At least one of `image_tag` or `env_var_overrides` is required.

**Rollback:** Call `UpdateService` back to the previous task definition ARN stored in `execution_result.old_task_def_arn`. The new task definition revision is left registered — use `ecs_task_def_deregister` to clean it up.

**Connector:** AWS

---

## ECS Task Definition Deregister

**Change type:** `ecs_task_def_deregister`

Deregisters an ECS task definition revision, marking it INACTIVE. Typically used to clean up a failed-deploy task definition after auto-rollback has restored the service.

!!! warning "Irreversible"
    ECS has no API to re-register a deregistered task definition revision. This operation cannot be rolled back.

**Parameters:** `task_def_arn` (required), `region` (optional)

**Rollback:** Not available.

**Connector:** AWS

---

## Update ECS Task Definition Environment Variables

**Change type:** `update_ecs_task_def_env`

Registers a new ECS task definition revision with an updated environment variable for a specified container. **Does not call `UpdateService`** — the running service continues on the old revision. Use this when only the new task definition ARN is needed (for example, as part of a `credential_rotation_fanout` that updates multiple consumers).

The executor verifies that the current value of the environment variable matches `old_value` before applying the change. If the variable is not found with the expected value, the operation is skipped without error.

Rollback requires updating any services that reference the new task definition ARN back to the original — this is surfaced as a manual action with `old_task_def_arn` in the rollback data.

To deploy the new revision to a live service, use `ecs_rolling_deploy` instead.

**Rollback:** Register a new revision restoring the original environment variables. Operators must manually update services that reference the new task definition ARN to point back to the old one.

**Connector:** AWS
