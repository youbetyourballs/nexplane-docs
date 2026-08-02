# Rolling Deploys with Auto-Rollback

Nexplane orchestrates ECS rolling deploys as tracked change requests with multi-gate health checks and automatic rollback if any gate fails.

## The Problem

Registering a new ECS task definition revision doesn't deploy it — `UpdateService` must be called separately. Tracking the old revision ARN for rollback, watching deployment health, and running `UpdateService` again if health fails is manual work that is easy to skip under pressure.

`ecs_rolling_deploy` closes this gap: it registers the new revision, triggers the service update, polls three health gates, and rolls back automatically if any fails.

## Health Gates

The executor polls gates in sequence. All three can be enabled simultaneously; earlier gates must pass before later ones are checked.

**1. ECS Stability**

Polls `DescribeServices` every 10 seconds. Passes when `runningCount == desiredCount` and there is exactly one active deployment. If the stability timeout is exceeded (default: 300s), the service is rolled back automatically.

**2. ALB/NLB Target Health** *(optional)*

When `target_group_arn` is provided, polls `DescribeTargetHealth` every 10 seconds. Passes when all registered targets report `healthy`. Enable this when the ECS service sits behind a load balancer.

**3. HTTP Probe** *(optional)*

When `health_check_url` is provided, GETs the URL and checks for the expected HTTP status code (default: 200). Useful for verifying application-level health beyond ECS service stability.

## Auto-Rollback

If any health gate times out or returns an unexpected result, the executor immediately calls `UpdateService` back to the previous task definition ARN. The execution result records:

- `rolled_back: true`
- `rollback_reason` — which gate failed and why
- `old_task_def_arn` and `new_task_def_arn`

The new task definition revision is left registered after auto-rollback — it is not automatically deregistered. Use [`ecs_task_def_deregister`](../change-types/container-operations.md#ecs-task-definition-deregister) to clean it up once you've confirmed it's no longer needed.

## Platform Rollback

If a deploy completes successfully but a regression is later discovered, operators can trigger a platform rollback from the CR detail view. This calls `UpdateService` back to the task definition ARN recorded at the start of the deploy.

## See Also

- [Container Operations](../change-types/container-operations.md) — full CR type reference with all parameters
