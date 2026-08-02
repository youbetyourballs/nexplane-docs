# Container Operations

ECS change types manage task definition revisions and service deployments on Amazon ECS.

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
