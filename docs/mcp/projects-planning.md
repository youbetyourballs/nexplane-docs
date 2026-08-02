# Projects, Planning & Migration

## Projects

These tools provide the full project lifecycle for coordinating multi-step infrastructure changes. A project is a named, goal-driven sequence of Change Requests executed in dependency order with FILO rollback support. The typical flow is: create a project, chat with the AI planner to draft a CR sequence, materialize the plan into real CRs, define success criteria, execute phases, and verify outcomes. Use `list_projects` and `get_project_status` to monitor progress; use `rollback_project` to unwind completed work in reverse order.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `list_projects` | List all projects for the organisation. Optionally filter by status (draft/in_progress/completed/cancelled/rolling_back). Returns summary fields. Use `get_project` for full detail including CR list. | `status`, `limit` |
| `get_project` | Get full project detail including all CRs (with status), pending approvals, dependency edges, and rollback history. Use before executing or rolling back. | `project_id` |
| `get_project_status` | Get actionable project status: what's pending approval, what's blocked by dependencies, and which CRs are ready to execute right now (approved with all deps completed). | `project_id` |
| `get_project_timeline` | Return the execution timeline for a project: all CRs ordered by last-updated time with execution outcomes. Use to audit what happened and in what order. | `project_id` |
| `estimate_project_risk` | Estimate execution risk: blast radius (unique assets touched), rollback coverage, estimated duration, and highest-risk CR. Use before approving or executing a project. | `project_id` |
| `create_project` | Create a new project in draft status. A project is a named, goal-driven sequence of Change Requests. Use `chat_with_project` to build the plan interactively, then `add_cr_to_project` to assemble the CR sequence. | `name`, `goal`, `description`, `template` |
| `chat_with_project` | Send a message to the AI project planner. The AI has full context of the project goal and all registered assets. It will ask clarifying questions and eventually produce a `proposed_crs` plan you can materialise with `materialize_project_plan`. Returns `{reply, proposed_crs}`. | `project_id`, `message` |
| `add_cr_to_project` | Add an existing Change Request to a project. `sequence_order` defaults to max(existing)+1. `depends_on` is a list of ProjectChangeRequest IDs that must complete before this CR is eligible for execution. | `project_id`, `cr_id`, `sequence_order`, `depends_on` |
| `remove_cr_from_project` | Remove a Change Request from a project. Automatically re-numbers remaining members to preserve a contiguous sequence_order starting at 1. | `project_id`, `cr_id` |
| `reorder_project_crs` | Reorder CRs in a project by providing the desired ordered list of CR IDs. `sequence_order` is set to 1-based index matching the provided list. All CR IDs must already be members of the project. | `project_id`, `ordered_cr_ids` |
| `execute_project_phase` | Execute one phase of the project. If `cr_ids` is provided, execute only those specific CRs (they must be approved). If omitted, auto-selects all approved CRs whose dependencies are all completed. Returns `{started_executions, skipped}`. | `project_id`, `cr_ids` |
| `rollback_project` | Initiate a full FILO rollback of a project. Rolls back all completed CRs in reverse sequence order. `to_cr_id` limits rollback to a specific range — from the most recent CR down to and including the specified CR. | `project_id`, `notes`, `to_cr_id` |
| `materialize_project_plan` | Convert an AI-proposed plan into real CRs attached to the project. Each item in `proposed_crs` must have: `change_type`, `target_assets` (list of asset names), `desired_outcome` (dict), `seq` (int), `depends_on` (list of seq ints), and optionally `rationale`. Returns `{created_crs, errors}`. | `project_id`, `proposed_crs` |
| `define_success_criteria` | Define success criteria for a project. Each criterion has a `type` (cr_completed / host_state_check / service_check / port_check / manual), a `description`, and an `assertion` dict with type-specific fields. | `project_id`, `criteria` |
| `check_success_criteria` | Evaluate all success criteria for a project against live platform state. Returns `{overall: pass\|fail\|partial\|not_checked, criteria: [...]}`. Use after `execute_project_phase` to verify outcomes. | `project_id` |

!!! note "Project workflow"
    The recommended sequence is: `create_project` → `chat_with_project` (iterate until `proposed_crs` is ready) → `materialize_project_plan` → `define_success_criteria` → `execute_project_phase` → `check_success_criteria`. Use `get_project_status` at any point to see what is ready to execute next.

---

## Planning Context

These read-only tools help AI agents gather the context needed before proposing a change. Use them to understand asset history, fleet composition, dependency graphs, and organizational precedents. None of these tools create or modify data — they are safe to call at any point during planning.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `get_asset_history` | Return the change history for a specific asset. Queries all CRs whose `target_asset_ids` contains this asset, filtered to the last N days and optionally to specific change types. Returns CR summaries with outcome, approver, duration, and rollback status. Use before creating a CR to understand what has already been done to this asset. | `asset_id`, `since_days`, `cr_types` |
| `get_fleet_context` | Query the asset fleet with rich filters for planning purposes. Filters include: `os` (ILIKE match on metadata), `kernel_version_lt` (semver less-than), `environment`, `tag_key`/`tag_value` pair, `connector_type`, and `has_open_findings`. Returns asset_id, name, type, environment, connector_type, tags, open_findings_count, and last_cr_at for each match. | `os`, `kernel_version_lt`, `environment`, `tag_key`, `tag_value`, `connector_type`, `has_open_findings`, `limit` |
| `find_similar_assets` | Find assets similar to the given asset using Jaccard similarity on tags. Matches assets of the same type and environment. Returns sorted list with `shared_tags` and `different_tags` for each candidate. Use to find peer assets when planning fleet-wide changes. | `asset_id`, `limit` |
| `get_migration_precedents` | Return statistics on past executions of a specific change type in this org: success_rate, avg_duration_minutes, rollback_frequency, and the top 5 failure error strings. Use before creating a CR to understand historical risk and common failure modes. | `change_type`, `asset_type`, `limit` |
| `get_cross_host_dependency_map` | Build a dependency graph for a set of assets. Classifies edges as internal (both endpoints in the set) or external (one endpoint outside). Also identifies CRs that touch 2+ assets in the set. Use before planning multi-host changes to understand blast radius and coordination needs. | `asset_ids` |
| `get_kernel_eol_status` | Report kernel end-of-life status for Linux/EC2 assets. Reads `kernel_version` from asset metadata and matches to a hardcoded EOL table. Returns supported status, EOL date, and `days_until_eol` (negative = already EOL). If `asset_ids` is omitted, scans all org server/endpoint assets. | `asset_ids` |
| `get_environment_diff` | Compare software, users, services, and ports across 2+ assets. Reads from the intelligence cache. Returns: common items present on all assets, per-asset differences, and `missing_data` for assets where cache is unavailable. Use to detect configuration drift before applying fleet-wide changes. Requires at least 2 asset_ids. | `asset_ids` |
| `get_project_precedents` | Find projects with similar goals using PostgreSQL pg_trgm fuzzy matching. Returns projects with similarity > 0.3, ordered by descending score. For each project: completion stats, rollback count, duration in days, and the ordered CR sequence summary. Use to find precedents before planning a new project. | `goal`, `limit` |

---

## Migration Profiling

These tools support structured application migration workflows. The discovery and baseline tools create draft CRs — they do not execute immediately. Each CR must be submitted for approval and executed separately through the standard CR lifecycle. The read tools (`list_application_profiles`, `get_application_profile`, `list_database_instances`) are read-only and can be called at any planning stage.

!!! note "Ordered workflow"
    Migration profiling follows a strict sequence:

    1. `discover_application_profile` — map what is running and what it depends on
    2. `capture_behavioral_baseline` — record current behavior as the verification benchmark
    3. *(perform the migration)*
    4. `verify_against_baseline` — confirm the migrated system matches the pre-migration benchmark

    Each step creates a draft CR. Execute them in order, approving each before proceeding.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `list_application_profiles` | List discovered application profiles. Each profile describes a running workload's endpoints, dependencies, services, config files, and library versions. Use to find what applications have been discovered before planning a migration. | `environment`, `limit` |
| `get_application_profile` | Get the full application profile for a discovered workload — endpoints, dependencies, config files, library versions. Use before planning a migration to understand what the application depends on and what verification targets to use. | `asset_id` |
| `discover_application_profile` | Create a draft CR that maps what is running on a host and what it depends on — endpoints, services, library versions, config files, and dependencies — without requiring operator knowledge of the application. Produces an `application_profile` asset that is the starting point for `capture_behavioral_baseline`. | `asset_id`, `title` |
| `capture_behavioral_baseline` | Create a draft CR that records how the application behaves right now — HTTP endpoint latency/status, service states, dependency row counts — as the success benchmark for post-migration verification. Default observation window is 20 minutes. Run against the `application_profile` asset produced by `discover_application_profile`. | `profile_asset_id`, `observation_window_seconds`, `title` |
| `verify_against_baseline` | Create a draft CR that confirms the migrated system matches its pre-migration behavioral benchmark across four layers: Infrastructure (host reachability), Service (expected ports open), Application (HTTP endpoints within latency threshold), and Data (database dependencies reachable). Failure in Application or Data layers triggers FILO rollback. | `profile_asset_id`, `title` |
| `list_database_instances` | List discovered database instances. Each instance has engine, version, DSN template, and baseline row counts. Use when planning a database migration to identify source and target instances. | `environment`, `limit` |

---

## Reference Scanning

These tools manage the `scan_for_references` CR type, which locates all places across your connected systems where a resource being migrated is referenced. After a scan completes, ambiguous hits are surfaced as exceptions requiring operator review. Use the exception resolution tools to triage each hit: resolve it with an explicit update CR, re-run AI triage with additional context, or dismiss it as a false positive.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `scan_for_references` | Create a `scan_for_references` CR in draft state. Searches for references to resources being migrated across the specified connectors. Returns the CR id and initial status. Poll `get_scan_results` until the scan completes. | `search_terms`, `connector_ids`, `migration_context`, `asset_id` |
| `get_scan_results` | Get the current status and result summary for a `scan_for_references` CR. Returns status, total hit count, confident update count, and exception counts. Poll until `status=completed` before reviewing results. | `scan_cr_id` |
| `list_reference_exceptions` | List all scan exceptions for a completed scan. Each exception is an ambiguous hit that requires operator review. Use `resolve_reference_exception`, `reattempt_reference_triage`, or `dismiss_reference_exception` to handle each one. | `scan_cr_id` |
| `resolve_reference_exception` | Resolve a scan exception by creating an `update_reference` CR with explicit params. Use this when you can determine the correct update (old_value, new_value, location). Creates a draft update_reference CR linked to the exception. | `exception_id`, `update_cr_params` |
| `reattempt_reference_triage` | Re-run AI triage on a scan exception with additional operator context. If the AI becomes confident, the exception is marked resolved automatically. If still ambiguous, the reason is updated with refined analysis. | `exception_id`, `operator_context` |
| `dismiss_reference_exception` | Dismiss a scan exception with a documented reason. Use when the hit is a false positive or the reference intentionally stays unchanged. Reason is required and appears in the audit trail. Unresolved exceptions at execution time become `reference_not_updated` findings. | `exception_id`, `reason` |

!!! note "Exception triage order"
    Prefer `resolve_reference_exception` when the correct update is clear. Use `reattempt_reference_triage` when you have additional context that might help the AI classify the hit. Use `dismiss_reference_exception` only for confirmed false positives — dismissed exceptions are permanently recorded in the audit trail.

---

## Runbooks

Runbooks are reusable, versioned sequences of steps that execute against specified target assets. A runbook can include CR-creation steps, human approval gates, conditions, and notifications. Execution is asynchronous — after calling `execute_runbook`, poll `get_runbook_execution_status` for progress. The `auto_execute` flag on a runbook controls whether CR steps within it are submitted for human approval or executed immediately.

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `list_runbooks` | List runbooks for the org with `auto_execute` setting and description. Optionally filter by tag. Returns summary fields — use `get_runbook` for full detail. | `tag`, `limit` |
| `get_runbook` | Get runbook detail including all steps, types, and `auto_execute` setting. Step types include: `cr` (creates a CR), `approval` (human gate), `condition`, and `notification`. | `runbook_id` |
| `execute_runbook` | Execute a runbook against specified target assets. Creates a RunbookExecution record and queues execution. Returns execution ID and asset context bundle for the first target asset. Execution is asynchronous — poll `get_runbook_execution_status` for progress. | `runbook_id`, `target_asset_ids`, `context` |
| `get_runbook_execution_status` | Get the current status of a runbook execution including per-step results. Status values: `running` / `waiting_human` / `completed` / `failed` / `rolled_back`. | `execution_id` |
