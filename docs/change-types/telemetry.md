# Telemetry & Fleet Change Types

Telemetry and fleet change types deploy monitoring agents, manage services across fleets, and create observability resources.

## CloudWatch (AWS)

| Change Type | Description | Rollback |
|-------------|-------------|---------|
| `cloudwatch_alarm_create` | Create a CloudWatch alarm with configured metric, threshold, and actions | Delete the alarm |
| `cloudwatch_alarm_delete` | Delete a CloudWatch alarm | Recreate the alarm |

**Connector:** AWS

## Telemetry Agent Deploy

**Change type:** `telemetry_agent_deploy`

Deploy a telemetry or monitoring agent (CloudWatch Agent, Datadog Agent, etc.) to a managed host via the Nexplane Agent.

**Connector:** Nexplane Agent

## Remote Command

**Change type:** `remote_command`

Execute a pre-approved command template on a managed host. Only commands defined in the allow-list for the connector can be executed — freeform shell is blocked unconditionally.

**Connector:** Nexplane Agent or SSH

## Fleet Operations

| Change Type | Description |
|-------------|-------------|
| `rolling_restart` | Restart a service across a fleet N hosts at a time, with health checks between batches and an abort threshold |
| `canary_config_push` | Push a config to one host, verify, then roll to the rest |
| `distribute_file` | Push a file to all hosts in a fleet |
| `fleet_health_check` | Run a health check (disk, load, service status, pending-reboot) across a fleet before major operations |

**Connector:** Nexplane Agent (`fleet` package)

See [Fleet Operations](../features/fleet-operations.md) for full details on batch control and abort thresholds.
