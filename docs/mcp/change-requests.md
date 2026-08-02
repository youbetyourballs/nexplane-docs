# Change Requests

AI agents interact with infrastructure exclusively through change requests — they propose changes, humans approve them, and the platform executes them. These tools cover the full CR lifecycle: discovering available change types, creating draft CRs, reviewing the generated safety plan, routing to approvers, executing approved changes, monitoring progress, and rolling back when needed.

## Creating and Executing a CR — Workflow

The typical agent workflow for a safe infrastructure change:

1. `list_change_types` — discover available CR types for your infrastructure
2. `get_change_type` — retrieve the full parameter schema and requirements for a specific change type
3. `create_change_request` — propose the change (creates a draft CR with no execution)
4. `get_change_request_plan` — review the AI-generated execution plan and rollback path
5. `submit_for_approval` — move the CR to awaiting_approval so operators can review and approve
6. `approve_change_request` — approve the CR (requires operator or agent with approver role)
7. `execute_change_request` — run the approved change
8. `get_execution_progress` — poll for completion and progress updates
9. `rollback_change_request` — undo an executed change if needed

!!! note
    `create_change_request` always produces a CR in `draft` state. The platform never executes automatically — approval is always required before any action touches infrastructure.

## Tool Reference

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `list_change_types` | Discover what infrastructure changes are available. Returns all change types configured for your organization including native types (ec2_stop, key_rotation, etc.) and connector-backed actions. Use this first to find the right change_type before creating a CR. | (none) |
| `get_change_type` | Get the full parameter schema for a change type including all parameters, their types, preconditions, and expected effects. Use to understand what's required before creating a CR. | `change_type` |
| `list_change_requests` | Query change history for the org. Use to answer: what changed recently, who approved it, and what's in flight? Filter by status (draft/approved/executed/rolled_back/failed), change_type, or asset_id. Returns summary fields — use get_change_request for full detail. | `status`, `change_type`, `asset_id`, `limit` |
| `get_change_request` | Get full CR detail including plan steps, parameters, approval history, and execution log. Automatically includes the asset context bundle so you can validate the plan is appropriate. | `cr_id` |
| `get_change_request_plan` | Get the AI-generated execution plan for a CR — steps, estimated impact, rollback path, and preconditions. Review before approving. Call this after create_change_request and before approve_change_request. Automatically includes the asset context bundle. | `cr_id` |
| `create_change_request` | Create a draft Change Request for an infrastructure change against a target asset. The CR is created in draft state — it must be reviewed, approved, and executed separately. This tool NEVER executes a change directly. Discovery: use list_change_types to see available change_type values. For catalog_action CRs (connector-backed actions such as AWS/GCP/Okta/Kubernetes ops), pass parameters with connector_type, action_id, params, and optional rollback_strategy. Returns the draft CR and asset context bundle. | `change_type`, `asset_id`, `title`, `parameters` |
| `approve_change_request` | Approve a Change Request. Respects role-based permissions — tokens without approval permission are rejected. A user cannot approve a CR they created (platform-enforced). Use after reviewing the plan with get_change_request_plan. | `cr_id`, `comment` (optional) |
| `reject_change_request` | Reject a Change Request with a stated reason. Only users with approver or admin role can reject. Rejected CRs cannot be executed. | `cr_id`, `reason` |
| `submit_for_approval` | Move a CR from Draft to Awaiting Approval so approvers are notified. Use after create_change_request and reviewing get_change_request_plan. Generates an execution plan if still in draft. Returns updated CR status. | `cr_id` |
| `execute_change_request` | Execute an approved Change Request. Triggers the executor against the target connector. The CR must be in approved state. Execution is asynchronous — poll get_execution_progress for status. | `cr_id` |
| `get_execution_progress` | Poll execution progress for a CR currently in Executing state. Returns completed_steps, total_steps, current_step, percent_complete, and any error messages. Call repeatedly until status is 'completed' or 'failed'. | `cr_id` |
| `rollback_change_request` | Roll back an executed Change Request using the stored rollback snapshot. Only completed or failed CRs can be rolled back. Creates a rollback execution run. Returns rollback feasibility and steps. | `cr_id` |
| `get_cr_manifest` | Return the full CR type vocabulary enriched with planning metadata. Each entry includes: change_type, display_name, domain, action_class, touches, preconditions, effects, rollback_type, and parameters. Use filters to load only the relevant slice for a planning goal (by domain, action_class, touches, or rollback_type). Returns {"count": N, "entries": [...]}. | `domain` (optional), `action_class` (optional), `touches` (optional), `rollback_type` (optional) |
| `explain_change_request` | Get a compact human-readable summary of a CR suitable for LLM reasoning: what it does, what it touches, the risk level, who approved it, and whether rollback is available. Use instead of get_change_request when you need a quick, structured overview. | `cr_id` |
