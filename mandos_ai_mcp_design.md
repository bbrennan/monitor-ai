# Mandos-AI MCP Design

## Purpose

This document defines how Mandos-AI can eventually use the Model Context Protocol (MCP) to query persisted Mandos evidence and controlled Snowflake views from GitHub Copilot custom agents.

The immediate recommendation is **not** to start with MCP. Mandos-AI should first succeed as a setup, validation, report explanation, and alert triage assistant using documentation, examples, and user-provided artifacts.

MCP becomes valuable when Mandos has enough persisted monitoring evidence to support questions like:

```text
What changed over the last quarter?
Which issues keep recurring?
When did this model begin degrading?
What was the last healthy run?
Did the latest run look worse than the prior three?
```

At that point, Mandos-AI should query the evidence rather than rely on memory.

---

## Core principle

Mandos persists the evidence. Mandos-AI explains the evidence.

MCP is the controlled bridge between the conversational assistant and the evidence store.

```text
GitHub Copilot custom agent
        ↓
Mandos MCP server
        ↓
Mandos service layer
        ↓
Curated Mandos Snowflake views / tables
        ↓
Sourced facts returned to Mandos-AI
```

---

## Does this require Bedrock?

No. For a GitHub Copilot custom-agent workflow, Bedrock is not required.

MCP can be used with GitHub Copilot / VS Code workflows. Bedrock becomes relevant only if Mandos-AI later becomes a production dashboard chatbot or centrally hosted enterprise agent outside the developer environment.

Recommended split:

```text
GitHub Copilot + MCP
Best for developer productivity, Mandos setup, config generation, report explanation, and controlled querying from VS Code.

Bedrock / AgentCore, optional later
Best for a production Mandos dashboard assistant, centralized enterprise hosting, non-developer users, Okta integration, and broader runtime governance.
```

---

# MCP goals

The Mandos MCP server should enable Copilot to retrieve trusted Mandos evidence through controlled, auditable tools.

## Goals

- Query persisted Mandos baselines and runs.
- Retrieve metric trends.
- Compare monitoring runs.
- Summarize monitoring periods.
- Identify recurring issues.
- Support alert triage.
- Return source metadata with every factual result.
- Enforce read-only access initially.
- Query curated semantic views before raw tables.
- Avoid arbitrary SQL generation.

## Non-goals for initial MCP release

- No customer-level raw data access.
- No arbitrary SQL access.
- No writes to `MANDOS_*` tables.
- No official baseline creation.
- No official run persistence.
- No report generation as an official artifact without explicit confirmation.
- No replacement for Mandos Python APIs.
- No Model Validation approval automation.

---

# Architecture

## Recommended architecture

```text
.github/agents/mandos-analyst.agent.md
        ↓
Copilot MCP client
        ↓
Mandos MCP server
        ↓
Mandos tool/service layer
        ↓
Snowflake read-only connection
        ↓
MLHUB_OPS.MANDOS_* tables
MLHUB_OPS.VW_* semantic views
```

## Mandos MCP server responsibilities

The server should:

- Expose a small set of named tools.
- Validate tool parameters.
- Use parameterized query templates.
- Query only approved views/tables.
- Enforce row limits and date windows.
- Return structured JSON.
- Include source metadata.
- Include query limitations.
- Log tool calls for auditability.
- Never return raw PII/customer-level records by default.

## Mandos service layer responsibilities

The service layer should contain the actual business logic:

- Config validation
- Baseline lookup
- Run lookup
- Metric trend retrieval
- Run comparison
- Period summarization
- Recurring issue detection
- Alert triage

MCP should be a thin tool interface over this service layer.

---

# Evidence hierarchy

Mandos-AI should query sources in this order:

```text
1. Curated Mandos semantic views, such as VW_MODEL_RUN_HISTORY.
2. Normalized Mandos persistence tables, such as MANDOS_RUNS and MANDOS_METRICS.
3. Mandos configs and FeatureRules.
4. Mandos generated report artifacts.
5. Snowflake metadata.
6. Raw model/scored data only with explicit approval and controlled tools.
```

Initial MCP tools should use only levels 1-4.

---

# Recommended Snowflake objects

## Required minimum

```text
MANDOS_BASELINES
MANDOS_RUNS
```

## Strongly recommended normalized tables

```text
MANDOS_METRICS
MANDOS_ISSUES
```

## Recommended semantic views

```text
VW_MODEL_RUN_HISTORY
VW_MODEL_LATEST_STATUS
VW_MODEL_HEALTH_TREND
VW_MODEL_METRIC_TREND
VW_FEATURE_DQ_TREND
VW_FEATURE_DRIFT_TREND
VW_SEGMENT_HEALTH_TREND
VW_RECURRING_ISSUES
VW_RUN_COMPARISON_READY
VW_REPORT_INVENTORY
```

The MCP server should prefer the `VW_*` layer because it gives Data Engineering and Mandos maintainers a governed contract between Snowflake persistence and AI tools.

---

# Tool design principles

## Tool names

Use clear names with stable contracts:

```text
mandos.validate_config
mandos.get_baseline_context
mandos.list_model_runs
mandos.get_latest_run_summary
mandos.get_model_history
mandos.compare_runs
mandos.summarize_period
mandos.get_metric_trend
mandos.get_recurring_issues
mandos.triage_issues
mandos.generate_chart_data
```

## Tool output contract

Every tool should return:

```yaml
result:
  structured data returned by the tool

source_refs:
  - source_type: snowflake_view | snowflake_table | config | report | code
    source_name: MLHUB_OPS.VW_MODEL_RUN_HISTORY
    filters:
      MODEL_ID: credit_pd_v17
      START_DATE: 2026-01-01
      END_DATE: 2026-03-31

query_context:
  row_count_returned: 3
  time_window: 2026-01-01 to 2026-03-31
  grain: monthly
  limitations:
    - Label-based performance was not available for all runs.

facts_allowed: true
```

Mandos-AI can use returned `result` values as FACTS only if `facts_allowed: true` and source references are present.

## Query constraints

All tools should enforce:

- Required `model_id`
- Bounded date ranges
- Maximum row limits
- Approved metric names
- Approved metric families
- Approved views/tables
- Parameterized SQL only
- No arbitrary SQL string from the LLM

---

# Tool catalog

## 1. `mandos.validate_config`

### Purpose

Validate a Mandos config against the expected schema and common quality rules.

### Inputs

```yaml
config_path: string | null
config_yaml: string | null
strict: boolean = false
```

### Returns

```yaml
valid: boolean
errors:
  - field: BASELINE_TABLE
    message: Missing required field.
warnings:
  - field: FEATURE_RULES
    message: Critical features missing FeatureRules.
recommendations:
  - Add FeatureRules for top model drivers.
```

### Access tier

Read-only / local.

---

## 2. `mandos.get_baseline_context`

### Purpose

Retrieve baseline metadata for a model.

### Inputs

```yaml
model_id: string
model_version: string | null
baseline_id: string | null
```

### Returns

```yaml
model_id: credit_pd_v17
model_version: v17
baseline_id: baseline_2026_01
baseline_table: MLHUB_ORIGINATIONS.TRAINING_FEATURES_V17
baseline_created_at: 2026-01-15
baseline_window_start_ts: 2025-01-01
baseline_window_end_ts: 2025-12-31
row_count: 1200000
feature_count: 86
score_column: PD_SCORE
target_column: DEFAULT_FLAG
config_hash: abc123
feature_rules_hash: def456
```

### Access tier

Read-only.

---

## 3. `mandos.list_model_runs`

### Purpose

List persisted monitoring runs for a model.

### Inputs

```yaml
model_id: string
model_version: string | null
start_date: string | null
end_date: string | null
limit: integer = 50
```

### Returns

```yaml
runs:
  - run_id: run_2026_03
    run_timestamp: 2026-03-31
    baseline_id: baseline_2026_01
    overall_status: PASS
    health_score: 91
  - run_id: run_2026_04
    run_timestamp: 2026-04-30
    baseline_id: baseline_2026_01
    overall_status: WARNING
    health_score: 82
```

### Access tier

Read-only.

---

## 4. `mandos.get_latest_run_summary`

### Purpose

Retrieve the latest monitoring run summary.

### Inputs

```yaml
model_id: string
model_version: string | null
```

### Returns

```yaml
run_id: run_2026_04
run_timestamp: 2026-04-30
overall_status: WARNING
health_score: 82
dq_status: PASS
drift_status: WARNING
performance_status: UNAVAILABLE
proxy_status: PASS
critical_issue_count: 0
warning_issue_count: 4
report_uri: s3://.../report.pdf
```

### Access tier

Read-only.

---

## 5. `mandos.get_model_history`

### Purpose

Retrieve historical run-level model health data.

### Inputs

```yaml
model_id: string
model_version: string | null
start_date: string | null
end_date: string | null
grain: monthly | weekly | daily | run
```

### Returns

```yaml
history:
  - run_timestamp: 2026-01-31
    health_score: 94
    overall_status: PASS
    critical_issue_count: 0
    warning_issue_count: 1
  - run_timestamp: 2026-02-28
    health_score: 90
    overall_status: PASS
    critical_issue_count: 0
    warning_issue_count: 2
```

### Access tier

Read-only.

---

## 6. `mandos.compare_runs`

### Purpose

Compare two specific monitoring snapshots.

### Inputs

```yaml
model_id: string
run_id_a: string
run_id_b: string
include_segments: boolean = true
include_features: boolean = true
```

### Returns

```yaml
run_a:
  run_id: run_2026_03
  run_timestamp: 2026-03-31
run_b:
  run_id: run_2026_04
  run_timestamp: 2026-04-30
summary:
  health_score_delta: -9
  new_issues: 3
  resolved_issues: 1
  worsened_metrics: 5
  improved_metrics: 2
top_changes:
  - metric_name: PSI
    entity_type: SCORE
    feature_name: SCORE_BAND
    value_a: 0.11
    value_b: 0.31
    delta: 0.20
    status_a: PASS
    status_b: WARNING
```

### Access tier

Analysis.

---

## 7. `mandos.summarize_period`

### Purpose

Summarize model health over a period, such as last quarter or last year.

### Inputs

```yaml
model_id: string
model_version: string | null
start_date: string
end_date: string
grain: monthly | weekly | run
```

### Returns

```yaml
period:
  start_date: 2026-01-01
  end_date: 2026-03-31
  run_count: 3
summary:
  health_score_start: 94
  health_score_end: 86
  health_score_min: 86
  health_score_max: 94
  failing_run_count: 0
  warning_run_count: 1
trends:
  - metric_family: DRIFT
    metric_name: PSI
    entity: SCORE_BAND
    direction: worsening
    start_value: 0.08
    end_value: 0.22
    max_value: 0.22
recurring_issues:
  - issue_type: SCORE_DRIFT
    occurrence_count: 2
    latest_status: WARNING
```

### Access tier

Analysis.

---

## 8. `mandos.get_metric_trend`

### Purpose

Retrieve a metric time series.

### Inputs

```yaml
model_id: string
metric_family: DQ | DRIFT | PERFORMANCE | PROXY
metric_name: string
feature_name: string | null
segment_name: string | null
segment_value: string | null
start_date: string | null
end_date: string | null
```

### Returns

```yaml
metric_trend:
  - run_timestamp: 2026-01-31
    metric_value: 0.04
    threshold_value: 0.20
    status: PASS
  - run_timestamp: 2026-02-28
    metric_value: 0.11
    threshold_value: 0.20
    status: PASS
  - run_timestamp: 2026-03-31
    metric_value: 0.23
    threshold_value: 0.20
    status: WARNING
```

### Access tier

Read-only.

---

## 9. `mandos.get_recurring_issues`

### Purpose

Find repeated issues across monitoring history.

### Inputs

```yaml
model_id: string
lookback_runs: integer = 12
severity: string | null
metric_family: string | null
```

### Returns

```yaml
recurring_issues:
  - issue_type: FEATURE_DRIFT
    feature_name: INCOME_LOG
    occurrence_count: 6
    first_seen_run_id: run_2025_11
    last_seen_run_id: run_2026_04
    max_severity: WARNING
    latest_status: WARNING
```

### Access tier

Analysis.

---

## 10. `mandos.triage_issues`

### Purpose

Prioritize issues and reduce alert fatigue.

### Inputs

```yaml
model_id: string
run_id: string | null
lookback_runs: integer = 6
include_suppressed: boolean = true
```

### Returns

```yaml
action_required:
  - issue_id: issue_001
    reason: Persistent score PSI breach for 3 consecutive runs; high affected volume; near decision cutoff.
watchlist:
  - issue_id: issue_014
    reason: First-time warning breach; moderate magnitude; medium volume.
informational:
  - issue_id: issue_021
    reason: Metric moved but remains within historical range.
suppressed:
  - issue_id: issue_033
    reason: Low-volume segment below materiality threshold.
```

### Access tier

Analysis.

---

## 11. `mandos.generate_chart_data`

### Purpose

Return chart-ready data for custom reports without letting the LLM generate arbitrary SQL.

### Inputs

```yaml
model_id: string
chart_type: health_trend | psi_trend | issue_summary | segment_heatmap | score_distribution
run_id: string | null
start_date: string | null
end_date: string | null
filters: object | null
```

### Returns

```yaml
chart_type: health_trend
data:
  - x: 2026-01-31
    y: 94
    label: PASS
  - x: 2026-02-28
    y: 90
    label: PASS
metadata:
  x_label: Run Date
  y_label: Health Score
  title: Model Health Score Trend
```

### Access tier

Analysis.

---

# Safety and governance

## Read-only first

The first MCP release should be read-only. It should query persisted evidence and return facts. It should not alter Mandos state.

## No arbitrary SQL

The LLM should never send arbitrary SQL to Snowflake. Tools should use parameterized query templates.

Bad pattern:

```text
LLM writes SQL directly against MLHUB_OPS.
```

Good pattern:

```text
LLM calls mandos.get_metric_trend(model_id, metric_name, start_date, end_date).
MCP server validates params and executes a controlled query.
```

## Least privilege Snowflake access

The MCP server should use a Snowflake role with access limited to:

```text
MLHUB_OPS.MANDOS_* tables
MLHUB_OPS.VW_* views
```

It should not have broad access to domain schemas or raw scored data in the initial release.

## Audit logging

Every tool call should log:

```text
timestamp
user identity, if available
agent name
tool name
parameters, with sensitive fields redacted
source views/tables queried
row count returned
execution time
success/failure
```

## Result limits

All tools should enforce row limits. Historical queries should aggregate rather than return raw detailed rows.

## Data minimization

Return only what the assistant needs:

- Metric values
- Run metadata
- Issue summaries
- Segment-level aggregate values
- Source references
- Limitations

Do not return raw records unless explicitly designed and approved.

## Source references

Every fact-producing tool should return source references. Mandos-AI should cite those source references in the answer.

---

# Example Copilot custom agent MCP configuration

This is illustrative only. Final syntax should be verified against the active enterprise Copilot setup.

```yaml
---
name: mandos-analyst
description: Helps users configure, run, interpret, and report with Mandos model monitoring workflows.
tools:
  - read
  - search
  - edit
mcp-servers:
  mandos:
    command: python
    args:
      - -m
      - mandos_ai.mcp.server
    env:
      SNOWFLAKE_ACCOUNT: ${input:snowflake_account}
      SNOWFLAKE_USER: ${input:snowflake_user}
      SNOWFLAKE_ROLE: ${input:snowflake_role}
      SNOWFLAKE_WAREHOUSE: ${input:snowflake_warehouse}
      SNOWFLAKE_DATABASE: ${input:snowflake_database}
      SNOWFLAKE_SCHEMA: MLHUB_OPS
---

You are the Mandos Analyst agent...
```

Enterprise environments may prefer centrally managed remote MCP servers rather than local commands.

---

# Implementation phases for MCP

## MCP Phase 0: Design only

Create documentation and tool contracts.

Deliverables:

```text
docs/design/mandos_ai_mcp_design.md
docs/design/mandos_ai_tool_contracts.md
docs/design/mandos_semantic_views.md
```

No MCP server yet.

## MCP Phase 1: Local mock server

Build a local MCP server that reads sample JSON or SQLite data.

Purpose:

- Validate tool names.
- Validate response shapes.
- Test Copilot integration.
- Avoid Snowflake/security dependency.

Tools:

```text
validate_config
list_model_runs
get_latest_run_summary
compare_runs
summarize_period
triage_issues
```

## MCP Phase 2: Development Snowflake read-only server

Connect to development Snowflake with read-only access to sample Mandos tables/views.

Requirements:

- No raw customer data.
- Dev data only.
- Read-only role.
- Query templates only.
- Audit logging.

## MCP Phase 3: Enterprise read-only server

Host internally with controlled access.

Requirements:

- Centrally managed credentials or service account.
- Least-privilege Snowflake role.
- Approved schema/view list.
- User/tool audit logs.
- Row limits and timeouts.
- Security review.

## MCP Phase 4: Analysis tools

Add more advanced tools:

```text
summarize_period
detect_change_points
triage_issues
generate_chart_data
```

These remain read-only but perform more sophisticated logic.

## MCP Phase 5: Gated write tools, optional

Only after strong governance.

Possible tools:

```text
create_baseline_draft
persist_monitoring_run
generate_official_report
update_model_registry
```

These require explicit user confirmation and should likely be separated from the default `mandos-analyst` agent.

---

# Suggested package structure

```text
mandos_ai/
  mcp/
    server.py
    auth.py
    logging.py
    errors.py
    tools/
      validate_config.py
      baseline.py
      runs.py
      metrics.py
      issues.py
      trends.py
      triage.py
      chart_data.py
  services/
    config_service.py
    baseline_service.py
    run_service.py
    metric_service.py
    issue_service.py
    trend_service.py
    triage_service.py
  repositories/
    snowflake_repository.py
    mock_repository.py
  schemas/
    tool_inputs.py
    tool_outputs.py
```

Important design point:

The MCP layer should be thin. Business logic should live in services that can also be tested outside MCP.

---

# Testing strategy

## Unit tests

Test each service function:

- Config validation
- Run comparison
- Period summary
- Metric trend retrieval
- Recurring issue detection
- Triage classification

## Contract tests

Each MCP tool should have stable input/output schema tests.

## Golden prompt tests

Use representative questions:

```text
What changed between March and April?
What changed over the last quarter?
Which issues keep recurring?
What should I worry about in the latest run?
Can you prove Zuul has improved over the last year?
```

Expected behavior:

- Uses tools when data is required.
- Does not invent facts.
- States limitations when history is unavailable.
- Separates facts, interpretations, and recommendations.

## Security tests

Test that tools reject:

- Arbitrary SQL
- Unknown model IDs if access is restricted
- Excessive date ranges
- Unknown metric names
- Requests for raw rows
- Requests to unsupported schemas

---

# Initial priority list

When MCP is introduced, build in this order:

```text
1. mandos.validate_config
2. mandos.get_baseline_context
3. mandos.list_model_runs
4. mandos.get_latest_run_summary
5. mandos.get_metric_trend
6. mandos.compare_runs
7. mandos.summarize_period
8. mandos.get_recurring_issues
9. mandos.triage_issues
10. mandos.generate_chart_data
```

Do not start with write tools.

---

# Final recommendation

MCP should be a later-stage controlled evidence bridge, not the starting point.

The right path is:

```text
1. Build Mandos-AI as a Copilot custom agent for setup and validation.
2. Use Zuul 2.0 as the first serious pilot.
3. Build report explanation and alert triage.
4. Add custom reporting and visualization generation.
5. Once persisted history exists, introduce read-only MCP tools.
6. Use MCP to query Mandos evidence, not raw data.
7. Consider Bedrock only if Mandos-AI moves into a production dashboard or enterprise web application.
```

Mandos-AI should not become powerful because it remembers everything. It should become powerful because it knows how to retrieve trusted evidence, reason over it, and explain what matters.
