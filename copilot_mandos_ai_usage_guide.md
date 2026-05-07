# GitHub Copilot Usage Guide for Mandos-AI

**Audience:** Risk Data Science / Machine Learning developers, Mandos maintainers, and AI-assisted development users  
**Primary use case:** Using GitHub Copilot and custom agents to develop, operate, and explain Mandos workflows safely, especially when MCP is not available  
**Generated:** 2026-05-07  
**Status:** Design and implementation guidance; adapt exact commands to the current Mandos codebase

---

## 1. Executive Summary

GitHub Copilot agents are most useful when they are treated as **structured engineering assistants**, not as uncontrolled autonomous developers. For Mandos-AI, the right pattern is:

```text
Copilot reasoning + repository tools + validated configuration + thin operational wrappers
```

Mandos already provides the core monitoring engine. Copilot should not duplicate Mandos logic. Instead, Copilot should help users:

1. Discover and understand the repository.
2. Write and validate YAML configurations.
3. Create tests, docs, examples, and onboarding material.
4. Execute approved local commands when allowed.
5. Summarize evidence produced by Mandos.
6. Avoid unsupported claims unless they are backed by Mandos outputs or approved metrics.

When MCP is available, Mandos workflows can eventually be exposed as schema-controlled MCP tools. When MCP is not available, the best substitute is a **thin CLI / Makefile wrapper** that calls Mandos workflows through approved, config-driven commands.

The most important design recommendation is:

> **Mandos arguments should mostly live in validated YAML, while Copilot should call small, config-first commands.**

That keeps the agent from improvising low-level parameters during evidence-generating monitoring runs.

---

## 2. Core Mental Model

### 2.1 Copilot is not one thing

“GitHub Copilot” can mean several surfaces:

| Surface | Typical usage | Tool access style |
|---|---|---|
| Copilot Chat in VS Code | Interactive coding, local repo assistance | Built-in tools, MCP tools, extension tools |
| Copilot Agent Mode in VS Code | Multi-step coding and refactoring | Workspace tools and user-approved operations |
| Copilot CLI | Terminal-based agent workflow | Shell/file/search/web/delegation tools with approvals |
| Copilot cloud agent on GitHub | Issue/PR/task execution in GitHub-hosted environment | Custom agents, built-in cloud tools, configured MCP tools |
| Custom agents | Specialized behavior and tool policy | `.agent.md` files with YAML frontmatter and instructions |

The exact tool availability depends on the environment, organization settings, repository settings, installed extensions, and whether MCP is enabled.

### 2.2 Reasoning versus tools

A useful distinction:

```text
Reasoning = deciding what should be done.
Tools = inspecting, editing, executing, querying, fetching, or validating something.
```

Copilot can reason about Mandos architecture, write code, propose a config, explain a monitoring concept, or draft SQL. But without tools, it cannot verify the live schema, query Snowflake, run Mandos, inspect generated artifacts, or validate a report.

For Mandos-AI:

| Task | Reasoning-only acceptable? | Tool required? |
|---|---:|---:|
| Explain Mandos architecture | Yes | No |
| Draft YAML config | Yes | No, but validation is better |
| Review a code module | No | Needs repo read/search |
| Modify code | No | Needs edit tool |
| Run tests | No | Needs execute/shell |
| Create Snowpark session | No | Needs approved command or Python execution |
| Build baseline | No | Needs Mandos runtime + Snowflake credentials |
| Profile/compare/monitor actual data | No | Needs approved Mandos execution path |
| Summarize actual model issues | No | Needs Mandos output or persisted evidence |
| Claim “model is healthy” | No | Needs certified metrics/evidence |

Mandos-AI should follow the rule:

> **No metric → no claim.**

---

## 3. Proper GitHub Copilot Usage

### 3.1 What Copilot is good at

Copilot is strongest when used for bounded, reviewable, evidence-backed engineering tasks:

- Explaining unfamiliar code.
- Finding where logic lives in a repository.
- Drafting implementation plans.
- Writing unit tests and fixtures.
- Refactoring small, well-scoped modules.
- Creating documentation and examples.
- Generating boilerplate code.
- Reviewing code quality, structure, typing, logging, and test coverage.
- Converting a design decision into a concrete checklist.
- Creating config templates from known schema rules.

### 3.2 What Copilot should not be trusted to do blindly

Avoid letting Copilot independently make high-risk or evidence-sensitive decisions:

- Running arbitrary SQL against production data.
- Creating or modifying Snowflake objects without explicit approval.
- Inferring model health without metrics.
- Creating baselines without a validated config and run record.
- Modifying security/authentication logic without review.
- Rewriting large parts of Mandos without a plan and acceptance criteria.
- Making architecture changes that violate KISS, YAGNI, or existing Mandos design principles.
- Adding dependencies without justification.
- Generating long, clever abstractions where simple functions/classes would work.

### 3.3 The right prompt structure

Use prompts that have five parts:

```text
1. Role
2. Context
3. Task
4. Constraints
5. Acceptance criteria
```

Example:

```text
You are a staff-level Python reviewer for the Mandos model monitoring library.

Context:
Mandos is a config-driven Python library for data quality, drift, and model performance monitoring using Snowflake/Snowpark. We value KISS, YAGNI, SOLID, typed code, pytest coverage, Ruff/Black/Mypy, and clear logging.

Task:
Review the current profile/compare/monitor workflow structure. Identify dead code, duplicated abstractions, unclear boundaries, missing tests, unsafe Snowflake execution patterns, and config validation gaps.

Constraints:
Do not implement changes. Do not rewrite files. Do not invent Mandos APIs that do not exist. Prefer small recommendations over broad refactors.

Acceptance criteria:
Produce a Markdown findings document with severity, evidence, affected files, suggested fix, and test implications.
```

### 3.4 Use Copilot in modes

Different tasks need different agent behaviors.

| Mode | Goal | Tools | Output |
|---|---|---|---|
| Discovery | Understand repo, find files, map workflows | read/search | Findings doc |
| Planning | Convert problem into design and steps | read/search | Implementation plan |
| Implementation | Make bounded code changes | read/search/edit/execute | PR-ready diff |
| Testing | Add/fix tests and run suite | read/search/edit/execute | Test changes and results |
| Review | Inspect for quality and risks | read/search | Review report |
| Operation | Run approved commands | execute | Structured outputs |
| Evidence summarization | Explain outputs | read/search/execute | Evidence-backed summary |

Do not mix all modes in one prompt. “Review, redesign, implement, test, deploy, and summarize” is too broad.

### 3.5 Keep Copilot inside a contract

Copilot works better with explicit contracts:

```text
Allowed:
- Inspect source files.
- Propose small changes.
- Edit tests and docs.
- Run `make test`, `make lint`, `make typecheck`.
- Use `mandos-ai` commands for approved workflows.

Not allowed:
- Run arbitrary Snowflake SQL.
- Create/drop/modify Snowflake objects directly.
- Add new dependencies without justification.
- Rename public APIs without a migration plan.
- Claim model health unless Mandos output supports it.
```

### 3.6 Recommended repository-level instructions

Create or maintain repository instructions that Copilot will repeatedly see.

Recommended files:

```text
.github/copilot-instructions.md
.github/agents/mandos-ai.agent.md
.github/agents/mandos-reviewer.agent.md
.github/agents/mandos-testing.agent.md
AGENTS.md
CONTRIBUTING.md
docs/architecture/
docs/operations/
docs/onboarding/
examples/configs/
examples/notebooks/
```

The exact support for these files can vary by Copilot surface, but the principle is consistent: make the repo self-explanatory to the agent.

### 3.7 Custom agents

A custom Copilot agent is a Markdown profile with YAML frontmatter and instructions. Repository-level custom agent profiles are created as `.agent.md` files under `.github/agents` for repository agents.

Example:

```yaml
---
name: mandos-ai-agent
description: Helps Risk DS developers configure, run, and interpret Mandos workflows safely
tools: ["read", "search", "edit", "execute"]
---

You are a Mandos-AI assistant.

Mandos is a config-driven Python model monitoring library. It supports Snowpark sessions,
baseline creation, data profiling, drift comparison, production monitoring, and reporting.

Use approved repository commands only. Prefer YAML configuration over ad hoc arguments.
Do not invent public APIs. Do not run arbitrary Snowflake SQL. Do not claim model health
unless the claim is supported by Mandos output or persisted evidence.
```

Use separate agents for separate jobs:

| Agent | Purpose | Suggested tools |
|---|---|---|
| `mandos-planner` | Architecture and implementation planning | read/search |
| `mandos-reviewer` | Codebase review and simplification | read/search |
| `mandos-testing` | Test design and implementation | read/search/edit/execute |
| `mandos-docs` | Documentation and examples | read/search/edit |
| `mandos-ai-agent` | Config-driven Mandos workflow assistance | read/search/edit/execute |

---

## 4. How Copilot Tools Work

### 4.1 Built-in tools

Without MCP, Copilot can still use built-in tools depending on the surface. Common custom-agent aliases include:

| Alias | Purpose |
|---|---|
| `read` | Read file contents |
| `search` | Search files or text in files |
| `edit` | Edit files |
| `execute` | Run shell commands through the relevant shell |
| `agent` | Invoke another custom agent |
| `web` | Web search/fetch where supported |
| `todo` | Structured task list where supported |

Do not assume every alias works in every environment. VS Code, Copilot CLI, and Copilot cloud agent differ.

### 4.2 MCP tools

MCP tools are named functions exposed by an MCP server. A tool has a name, description, and schema. The model can discover available tools and call them with structured arguments.

Example MCP-style tool names:

```yaml
tools:
  - "github/list_issues"
  - "playwright/browser_snapshot"
  - "company-snowflake/describe_table"
  - "mandos/get_latest_issues"
```

These names only work if the MCP server exists and has been configured.

### 4.3 Extension tools

In VS Code, tools can also come from extensions. This is separate from MCP. Extension tools may provide access to platform-specific actions, cloud services, test frameworks, or repository intelligence.

For Mandos-AI, do not assume extension tools exist unless your team has installed and approved them.

### 4.4 Tool policy matters

A powerful agent with too many tools becomes harder to control. Prefer:

```yaml
tools: ["read", "search", "edit"]
```

for planning and review, and only add:

```yaml
tools: ["execute"]
```

when running tests, linters, or approved wrapper commands is necessary.

Avoid broad tool exposure such as:

```yaml
tools:
  - "company-snowflake/*"
  - "mandos/*"
  - "execute"
```

unless the execution environment is sandboxed, audited, and explicitly approved.

---

## 5. Working Without MCP Access

### 5.1 What changes without MCP

Without MCP, this does not create usable tools:

```yaml
tools:
  - "company-snowflake/list_approved_connections"
  - "company-snowflake/describe_table"
  - "company-snowflake/execute_readonly_query"
```

Those are MCP-style names. If MCP is not configured, Copilot will not have those callable functions.

What still works:

```yaml
tools:
  - "read"
  - "search"
  - "edit"
  - "execute"
```

That means Copilot can inspect the repo, edit files, and possibly run shell commands. If Mandos is installed in the environment and `execute` is available, Copilot can run approved Python/CLI commands.

### 5.2 Do we need to build our own tools?

For normal code work: **No.** Built-in repo tools are enough.

For governed enterprise operations: **Yes, but keep them thin.**

Without MCP, your “tools” should be:

```text
Makefile targets
Python CLI commands
Shell scripts with guardrails
Internal SDK functions
Pre-approved notebooks
```

The key is that Copilot should not be forced to improvise low-level Python, Snowflake, and Mandos method calls every time. Give it a stable command surface.

### 5.3 Recommended non-MCP architecture

```text
Copilot Agent
  uses read/search/edit/execute
        ↓
Approved Makefile / CLI commands
        ↓
Thin Mandos-AI wrapper
        ↓
Mandos public workflows
        ↓
Snowpark / Snowflake / report artifacts / persisted evidence
```

This is the best bridge between “no MCP today” and “MCP later.”

### 5.4 Thin wrapper does not mean duplicated logic

A thin wrapper should not reimplement Mandos. It should:

1. Load YAML.
2. Validate YAML.
3. Resolve environment variables and secrets safely.
4. Create or validate a Snowpark session.
5. Call the appropriate Mandos public workflow.
6. Return compact JSON.
7. Write logs and artifacts.
8. Exit with meaningful status codes.

It should not:

- Recalculate Mandos metrics.
- Duplicate profile/compare/monitor logic.
- Construct ad hoc SQL outside Mandos unless explicitly part of Mandos internals.
- Let the agent pass arbitrary SQL.
- Let the agent override critical runtime settings without config validation.

### 5.5 Example Makefile interface

```makefile
CONFIG ?= configs/example_model.yaml
OUTPUT ?= artifacts/latest

mandos-check-session:
	mandos-ai check-session --config $(CONFIG) --output json

mandos-build-baseline:
	mandos-ai build-baseline --config $(CONFIG) --output json

mandos-profile:
	mandos-ai profile --config $(CONFIG) --output json --artifact-dir $(OUTPUT)

mandos-compare:
	mandos-ai compare --config $(CONFIG) --output json --artifact-dir $(OUTPUT)

mandos-monitor:
	mandos-ai monitor --config $(CONFIG) --output json --artifact-dir $(OUTPUT)

mandos-report:
	mandos-ai report --config $(CONFIG) --format pdf --artifact-dir $(OUTPUT)

lint:
	ruff check .

format:
	black .

typecheck:
	mypy src tests

test:
	pytest -q
```

### 5.6 Example CLI interface

```bash
mandos-ai check-session --config configs/zuul_2_0.yaml --output json
mandos-ai build-baseline --config configs/zuul_2_0.yaml --output json
mandos-ai profile --config configs/zuul_2_0.yaml --output json
mandos-ai compare --config configs/zuul_2_0.yaml --output json
mandos-ai monitor --config configs/zuul_2_0.yaml --output json
mandos-ai report --config configs/zuul_2_0.yaml --format pdf
```

### 5.7 Example agent instructions without MCP

```yaml
---
name: mandos-ai-agent
description: Configures, runs, and summarizes Mandos workflows without MCP
tools: ["read", "search", "edit", "execute"]
---

You are a Mandos-AI assistant for Risk Data Science model monitoring workflows.

Mandos is installed in this environment, but MCP is not available. Therefore you do not
have callable MCP tools such as mandos/profile or company-snowflake/describe_table.

Use only approved commands:
- make mandos-check-session CONFIG=<path>
- make mandos-build-baseline CONFIG=<path>
- make mandos-profile CONFIG=<path>
- make mandos-compare CONFIG=<path>
- make mandos-monitor CONFIG=<path>
- make mandos-report CONFIG=<path>
- make test
- make lint
- make typecheck

Rules:
- Prefer validated YAML configuration over ad hoc command arguments.
- Do not run arbitrary Snowflake SQL.
- Do not create/drop/modify Snowflake objects directly.
- Do not claim model health unless the command output provides evidence.
- Summarize JSON output faithfully and include uncertainty when evidence is missing.
- If a command is missing, propose the command interface and implementation rather than bypassing the wrapper.
```

### 5.8 When not to use `execute`

Avoid giving `execute` to agents that only need to read, plan, or review.

Use this for review-only agents:

```yaml
tools: ["read", "search"]
```

Use this for docs agents:

```yaml
tools: ["read", "search", "edit"]
```

Use this only for agents that must run tests, linters, or approved Mandos commands:

```yaml
tools: ["read", "search", "edit", "execute"]
```

### 5.9 Permission and sandbox rules

For local and CLI-based Copilot usage:

- Use a virtual environment.
- Use a branch or worktree for agent changes.
- Prefer read-only Snowflake roles for agent-run workflows.
- Use row limits for previews.
- Use query tags.
- Do not expose secrets in prompts, repo files, or logs.
- Avoid `--allow-all-tools` unless running inside a constrained sandbox.
- Deny destructive commands such as `rm -rf`, `git push`, direct deployment, and unapproved DDL.

---

## 6. Mandos-AI Operating Model

### 6.1 Mandos critical workflows

The user-provided Mandos workflow list includes:

```python
create_session()      # create a Snowpark session
build_baseline()     # create ground-truth baseline for model monitoring
.profile()           # data quality scan
.compare()           # drift analysis scan
.monitor()           # DQ + drift + model performance
```

The prompt said “6 critical processes” but listed five explicit workflows. This guide treats **formal report/export generation** as the recommended sixth operational workflow when needed:

```python
.report() or export_report() or export_pdf()  # formal evidence artifact, if supported by current Mandos API
```

If the current Mandos codebase has a different sixth method, replace the report/export examples with the actual method.

### 6.2 The most important Mandos-AI principle

Most arguments should be **config-driven**.

Copilot should not frequently call Mandos like this:

```python
monitor.profile(
    table_name="...",
    baseline_table="...",
    feature_columns=[...],
    score_column="...",
    target_column="...",
    thresholds={...},
    segments=[...],
    where_clause="...",
)
```

Instead, Copilot should help create this:

```bash
mandos-ai profile --config configs/zuul_2_0.yaml --output json
```

The YAML becomes the source of operational intent.

### 6.3 Why config-first matters

Config-first Mandos-AI gives you:

| Requirement | Benefit |
|---|---|
| Repeatability | Same model config produces same workflow behavior |
| Governance | Config can be reviewed, versioned, approved |
| Auditability | Config hash can be stored with run outputs |
| Safety | Agent cannot casually override critical arguments |
| Maintainability | Python API remains clean; workflow settings live in YAML |
| Better onboarding | Model owners can fill guided templates |
| Future MCP path | One config schema can back CLI and MCP tools |

### 6.4 Recommended Mandos workflow boundaries

| Workflow | Purpose | Agent should provide | Wrapper should do |
|---|---|---|---|
| `create_session()` | Validate Snowpark connectivity | Config path / connection name | Resolve env, create session, validate role/db/schema |
| `build_baseline()` | Create baseline evidence | Config path | Validate baseline config, build baseline, persist metadata |
| `.profile()` | Data quality scan | Config path + optional snapshot label | Run DQ checks, return issue summary/artifacts |
| `.compare()` | Drift analysis | Config path + baseline/current identifiers | Run drift checks, return top shifts/issues |
| `.monitor()` | Production monitoring | Config path | Run DQ + drift + performance, persist evidence |
| Report/export | Human-readable evidence | Config path or run ID | Generate PDF/HTML/JSON artifacts |

### 6.5 Recommended YAML structure

This is a conceptual template. Align exact field names to current Mandos models.

```yaml
mandos_config_version: "1.0"

model:
  model_id: "zuul_2_0"
  model_version: "2026.05"
  business_domain: "originations"
  owner_team: "risk_data_science"
  model_owner: "<name-or-group>"
  risk_tier: "high"
  description: "Originations decisioning model"

snowflake:
  connection_name: "mlhub_prod_readonly"
  account_env: "SNOWFLAKE_ACCOUNT"
  user_env: "SNOWFLAKE_USER"
  role: "MLHUB_READONLY_ROLE"
  warehouse: "MLHUB_XS_WH"
  database: "MLHUB"
  schema: "MLHUB_ORIGINATIONS"
  authenticator: "externalbrowser"
  query_tag:
    application: "mandos-ai"
    model_id: "zuul_2_0"
    environment: "prod"
  allow_write: false

source_data:
  entity_key: "APPLICATION_ID"
  timestamp_column: "SNAPSHOT_DATE"
  score_column: "MODEL_SCORE"
  target_column: "DEFAULT_FLAG"
  prediction_column: "MODEL_SCORE"
  feature_columns:
    - "BUREAU_SCORE"
    - "DTI_RATIO"
    - "OPEN_CREDIT_LINES"
    - "APPLICATION_CHANNEL"
  segment_columns:
    - "PRODUCT_TYPE"
    - "CHANNEL"
  baseline:
    table: "MLHUB_ORIGINATIONS.MODEL_TRAINING_BASELINE"
    where_clause: "MODEL_VERSION = '2026.05'"
  current:
    table: "MLHUB_ORIGINATIONS.MODEL_SCORING_SNAPSHOT"
    where_clause: "SNAPSHOT_DATE = CURRENT_DATE()"

baseline:
  enabled: true
  strategy: "from_training_snapshot"
  overwrite_existing: false
  baseline_id: "zuul_2_0__2026_05__training"
  golden_bins:
    enabled: true
    score_bins: "decile"
    custom_edges: null
  persist:
    enabled: true
    table: "MLHUB_OPS.MANDOS_BASELINES"

profile:
  enabled: true
  checks:
    schema:
      enabled: true
      severity: "error"
    missing_values:
      enabled: true
      max_missing_rate: 0.05
      severity: "warning"
    unexpected_values:
      enabled: true
      severity: "error"
    whitespace_padding:
      enabled: true
      severity: "warning"
    constant_features:
      enabled: true
      max_top_value_rate: 0.98
      severity: "warning"
    cap_floor_clusters:
      enabled: true
      max_floor_rate: 0.20
      max_cap_rate: 0.20
      severity: "warning"
    sentinel_values:
      enabled: true
      values: ["9999", "99999", "UNKNOWN"]
      severity: "warning"

compare:
  enabled: true
  drift:
    psi:
      enabled: true
      warning_threshold: 0.10
      failure_threshold: 0.25
    csi:
      enabled: true
      warning_threshold: 0.10
      failure_threshold: 0.25
    categorical_shift:
      enabled: true
      max_new_category_rate: 0.02
    segment_drift:
      enabled: true
      segments:
        - "PRODUCT_TYPE"
        - "CHANNEL"

monitor:
  enabled: true
  include_profile: true
  include_compare: true
  include_performance: true
  performance:
    label_availability: "delayed"
    metrics:
      - "ks"
      - "auc"
      - "event_rate_by_score_bin"
    early_warning_proxies:
      enabled: true
      metrics:
        - "approval_rate"
        - "booking_rate"
        - "near_cutoff_mass"

reporting:
  enabled: true
  formats: ["json", "pdf"]
  artifact_dir: "artifacts/mandos/zuul_2_0"
  include_sections:
    - "executive_summary"
    - "top_issues"
    - "data_quality"
    - "drift"
    - "performance"
    - "appendix"

run_policy:
  environment: "prod"
  dry_run: false
  max_runtime_minutes: 60
  require_config_hash: true
  require_readonly_role: true
  fail_on_unvalidated_config: true
  log_level: "INFO"
```

### 6.6 Validate config before execution

Use Pydantic or a similar validation library to validate YAML before Mandos runs.

Minimum validation expectations:

- Required model identity fields are present.
- Snowflake connection fields are valid or resolvable from environment variables.
- Table names are fully qualified or resolved consistently.
- Feature columns are non-empty where required.
- Score/target/entity keys are present.
- Thresholds are valid numeric ranges.
- `allow_write` defaults to `false`.
- Production runs require a config hash.
- Reporting paths are controlled and workspace-safe.

Example shape:

```python
from pathlib import Path
from typing import Literal

import yaml
from pydantic import BaseModel, Field, field_validator


class SnowflakeConfig(BaseModel):
    connection_name: str
    role: str
    warehouse: str
    database: str
    schema: str
    allow_write: bool = False


class ModelConfig(BaseModel):
    model_id: str
    model_version: str
    business_domain: str
    owner_team: str


class MandosConfig(BaseModel):
    mandos_config_version: str = "1.0"
    model: ModelConfig
    snowflake: SnowflakeConfig

    @field_validator("mandos_config_version")
    @classmethod
    def validate_config_version(cls, value: str) -> str:
        if value != "1.0":
            raise ValueError("Unsupported Mandos config version")
        return value


def load_config(path: str | Path) -> MandosConfig:
    with open(path, "r", encoding="utf-8") as f:
        raw = yaml.safe_load(f)
    return MandosConfig.model_validate(raw)
```

### 6.7 Recommended JSON output contract

Mandos-AI CLI commands should return compact JSON for Copilot to summarize.

Example:

```json
{
  "status": "completed",
  "workflow": "monitor",
  "model_id": "zuul_2_0",
  "model_version": "2026.05",
  "run_id": "2026-05-07T12:01:33Z",
  "config_hash": "sha256:abc123...",
  "sections": {
    "data_quality": {
      "status": "warning",
      "issue_count": 4
    },
    "drift": {
      "status": "fail",
      "issue_count": 2
    },
    "model_performance": {
      "status": "pass",
      "issue_count": 0,
      "label_availability": "delayed"
    }
  },
  "top_issues": [
    {
      "severity": "high",
      "category": "drift",
      "feature": "MODEL_SCORE",
      "metric": "PSI",
      "value": 0.31,
      "threshold": 0.25,
      "message": "Score PSI exceeded failure threshold."
    }
  ],
  "artifact_paths": {
    "json": "artifacts/mandos/zuul_2_0/latest/summary.json",
    "pdf": "artifacts/mandos/zuul_2_0/latest/report.pdf",
    "logs": "artifacts/mandos/zuul_2_0/latest/run.log"
  }
}
```

The agent should be instructed to summarize only from these fields unless it has opened a referenced artifact.

### 6.8 Suggested wrapper implementation shape

```python
"""Thin CLI wrapper for Mandos-AI workflows.

This module should not reimplement Mandos metrics. It validates config, calls
public Mandos workflows, and returns structured outputs for agent consumption.
"""

from __future__ import annotations

import json
from pathlib import Path
from typing import Any

import typer

app = typer.Typer(help="Approved CLI for Mandos-AI workflows")


@app.command("check-session")
def check_session(config: Path, output: str = "json") -> None:
    cfg = load_config(config)
    session = create_session_from_config(cfg)
    result = {
        "status": "completed",
        "workflow": "check-session",
        "model_id": cfg.model.model_id,
        "connection_name": cfg.snowflake.connection_name,
        "role": cfg.snowflake.role,
    }
    emit(result, output)


@app.command("profile")
def profile(config: Path, output: str = "json") -> None:
    cfg = load_config(config)
    session = create_session_from_config(cfg)
    monitor = create_mandos_monitor(cfg, session)
    result = monitor.profile()
    emit(normalize_result(result, cfg, workflow="profile"), output)


def emit(result: dict[str, Any], output: str) -> None:
    if output == "json":
        print(json.dumps(result, indent=2, default=str))
    else:
        raise ValueError(f"Unsupported output format: {output}")


if __name__ == "__main__":
    app()
```

Use `typer`, `click`, or `argparse`. Keep the wrapper boring.

---

## 7. Mandos-AI Prompt Patterns

### 7.1 Discovery prompt

```text
You are a Mandos-AI discovery agent. Inspect the repository and identify where Mandos handles:
- Snowpark session creation
- baseline creation
- profile/data quality scanning
- compare/drift scanning
- monitor orchestration
- reporting/exporting
- YAML/Pydantic configuration
- CLI or Makefile entry points

Do not edit files. Produce a Markdown map of relevant files, public APIs, config models, and missing integration points for a non-MCP Copilot workflow.
```

### 7.2 Config creation prompt

```text
Create a Mandos YAML config for model onboarding using the existing config schema in this repository.

Use these model details:
[PASTE MODEL DETAILS]

Rules:
- Do not invent unsupported fields.
- Prefer existing examples as templates.
- Use placeholders where Snowflake credentials or exact table names are unknown.
- Include comments explaining each major section.
- After drafting, validate the config against the existing Pydantic models if an approved validation command exists.
```

### 7.3 Thin wrapper prompt

```text
We need a thin non-MCP wrapper so Copilot can run Mandos safely through approved commands.

Task:
Inspect the current Mandos public API and propose a CLI with commands:
- check-session
- build-baseline
- profile
- compare
- monitor
- report/export if supported

Constraints:
- Do not reimplement Mandos logic.
- All workflows must be config-first.
- Return compact JSON for agent consumption.
- Use existing config validation if available.
- Do not add Snowflake write behavior unless the config explicitly allows it and Mandos already supports it.

Output:
Create an implementation plan and file-by-file diff proposal. Do not implement until asked.
```

### 7.4 Safe execution prompt

```text
Run the approved Mandos profile workflow for this config:
configs/zuul_2_0.yaml

Use only approved commands from the Makefile or mandos-ai CLI.
Do not run arbitrary Snowflake SQL.
Return a concise summary of the JSON output, including status, issue counts, top issues, artifact paths, and uncertainty.
```

### 7.5 Evidence summarization prompt

```text
Summarize the Mandos run output in artifacts/mandos/zuul_2_0/latest/summary.json.

Rules:
- Only make claims supported by the JSON or referenced artifacts you open.
- Separate data quality, drift, and model performance findings.
- Highlight top severity issues first.
- Include whether labels were available or delayed.
- Do not recommend remediation that is not supported by the evidence; label hypotheses as hypotheses.
```

---

## 8. Using Popular Python Libraries with Copilot and Mandos

### 8.1 Snowpark

Mandos should use Snowpark for large-table operations where compute should remain in Snowflake.

Best practices:

- Create sessions from validated config.
- Use read-only roles for agent-run workflows.
- Set query tags for auditability.
- Push aggregation to Snowflake.
- Avoid collecting large datasets to local memory.
- Convert to pandas only for small summary outputs.
- Close sessions when no longer needed.
- Prefer explicit table/column references over stringly typed ad hoc SQL.

Good pattern:

```python
from snowflake.snowpark import Session


def create_session(connection_parameters: dict[str, str]) -> Session:
    session = Session.builder.configs(connection_parameters).create()
    session.query_tag = "application=mandos-ai;workflow=profile"
    return session
```

Avoid:

```python
# Avoid collecting full production tables locally.
df = session.table("VERY_LARGE_TABLE").to_pandas()
```

Prefer:

```python
# Push aggregation down, then collect only summary output.
summary = (
    session.table("MODEL_SCORING_SNAPSHOT")
    .group_by("PRODUCT_TYPE")
    .count()
    .to_pandas()
)
```

### 8.2 pandas

Use pandas for small, local, summary-level analysis and downstream metric calculations after Snowflake/Snowpark has produced aggregated primitives.

Best practices:

- Use pandas for summary outputs, not raw full production tables.
- Prefer vectorized operations over row loops.
- Keep transformation functions pure and testable.
- Validate required columns at function boundaries.
- Use `.copy()` when intentionally modifying slices.
- Use stable column naming conventions.
- Return DataFrames with predictable schemas.

Good pattern:

```python
import pandas as pd


def summarize_issues(issues: pd.DataFrame) -> pd.DataFrame:
    required = {"severity", "category", "feature", "metric"}
    missing = required - set(issues.columns)
    if missing:
        raise ValueError(f"Missing required columns: {sorted(missing)}")

    return (
        issues.groupby(["severity", "category"], dropna=False)
        .size()
        .reset_index(name="issue_count")
        .sort_values(["severity", "issue_count"], ascending=[True, False])
    )
```

Avoid:

```python
# Slow and hard to test.
for i, row in df.iterrows():
    df.loc[i, "flag"] = custom_logic(row)
```

### 8.3 Matplotlib

Use Matplotlib for deterministic static charts that can be embedded into PDF reports.

Best practices:

- Prefer explicit `fig, ax = plt.subplots()` over global state.
- Set figure size intentionally.
- Save charts to artifact paths.
- Close figures after saving to prevent memory leaks.
- Keep chart creation separate from data computation.
- Return file paths or figure objects, not mixed report logic.
- Use stable labels and titles.

Good pattern:

```python
from pathlib import Path

import matplotlib.pyplot as plt
import pandas as pd


def plot_issue_counts(issue_counts: pd.DataFrame, output_path: Path) -> Path:
    fig, ax = plt.subplots(figsize=(8, 4.5))
    ax.bar(issue_counts["category"], issue_counts["issue_count"])
    ax.set_title("Mandos Issue Counts by Category")
    ax.set_xlabel("Category")
    ax.set_ylabel("Issue Count")
    ax.tick_params(axis="x", rotation=30)
    fig.tight_layout()
    fig.savefig(output_path, dpi=150, bbox_inches="tight")
    plt.close(fig)
    return output_path
```

Avoid:

```python
# Avoid mixing plotting, data querying, and report writing in one function.
def make_report(session, table, pdf_path):
    data = session.table(table).to_pandas()
    plt.plot(data["x"], data["y"])
    # ... then write PDF directly here
```

### 8.4 ReportLab

Use ReportLab for formal PDF artifacts when you need programmatic control over report layout.

Best practices:

- Keep report generation separate from metric computation.
- Use a structured report model as input.
- Embed pre-rendered chart images from Matplotlib.
- Keep styles centralized.
- Use templates and reusable sections.
- Include run metadata, config hash, model ID, model version, and artifact timestamp.
- Do not let Copilot write one-off hardcoded PDF layouts for each model.

Recommended structure:

```text
src/mandos/reporting/
  __init__.py
  models.py          # ReportSummary, ReportSection, ReportIssue
  styles.py          # shared ReportLab styles
  charts.py          # chart generation wrappers
  pdf_report.py      # ReportLab document assembly
  templates.py       # section builders
```

Example shape:

```python
from pathlib import Path

from reportlab.lib.pagesizes import LETTER
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer
from reportlab.lib.styles import getSampleStyleSheet


def build_pdf_report(title: str, summary: str, output_path: Path) -> Path:
    styles = getSampleStyleSheet()
    doc = SimpleDocTemplate(str(output_path), pagesize=LETTER)
    story = [
        Paragraph(title, styles["Title"]),
        Spacer(1, 12),
        Paragraph(summary, styles["BodyText"]),
    ]
    doc.build(story)
    return output_path
```

### 8.5 Pydantic and YAML

Use Pydantic to validate configs before Mandos runs. Use YAML for human-editable model onboarding configs.

Best practices:

- Keep secrets out of YAML.
- Reference secrets through environment variable names.
- Validate config types and value ranges.
- Use field validators for thresholds, table names, and required columns.
- Store config version.
- Store config hash with run outputs.
- Prefer explicit defaults over implicit magic.

Good pattern:

```python
from pydantic import BaseModel, Field


class PsiConfig(BaseModel):
    enabled: bool = True
    warning_threshold: float = Field(default=0.10, ge=0.0)
    failure_threshold: float = Field(default=0.25, ge=0.0)
```

### 8.6 Jinja2

Use Jinja2 for SQL templates when SQL generation is unavoidable.

Best practices:

- Keep templates small and named by purpose.
- Validate template inputs before rendering.
- Avoid string concatenation for SQL.
- Quote identifiers consistently.
- Log template name and parameter hash.
- Test rendered SQL shape with fixtures.
- Avoid embedding business rules in invisible template branches.

Example:

```sql
-- templates/profile_missing_values.sql.j2
select
    '{{ column_name }}' as column_name,
    count(*) as row_count,
    sum(case when {{ column_name }} is null then 1 else 0 end) as missing_count
from {{ table_name }}
where {{ where_clause }}
```

### 8.7 pytest

Use pytest to enforce that Copilot-generated changes are correct.

Best practices:

- Require tests for every behavioral change.
- Add regression tests for every bug fix.
- Use small fixtures.
- Mock Snowflake where unit testing local logic.
- Use integration tests sparingly for Snowflake workflows.
- Tag slow/integration tests.
- Test config validation failures.
- Test JSON output contracts.

Example:

```python
import pytest
from pydantic import ValidationError


def test_psi_config_rejects_negative_threshold():
    with pytest.raises(ValidationError):
        PsiConfig(warning_threshold=-0.1)
```

### 8.8 Ruff, Black, and Mypy

Copilot-generated code should be held to the same quality bar as human code.

Recommended commands:

```bash
ruff check .
black --check .
mypy src tests
pytest -q
```

Recommended Copilot instruction:

```text
All generated Python must be Black-formatted, Ruff-clean, Mypy-compatible where practical, and use NumPy-style docstrings for public functions/classes.
```

### 8.9 Loguru or standard logging

Use centralized logging. Avoid print statements except CLI JSON output.

Best practices:

- Log workflow start/end.
- Log model ID, model version, run ID, config hash.
- Log table names and predicates carefully.
- Do not log secrets.
- Use query tags for Snowflake correlation.
- Keep logs structured enough for debugging.

---

## 9. Mandos-AI Guardrails

### 9.1 Data and Snowflake guardrails

Mandos-AI should enforce:

- Read-only default role.
- Explicit opt-in for writes.
- No arbitrary SQL execution by Copilot.
- Approved connections only.
- Environment-variable based credentials.
- Query tagging.
- Runtime limits.
- Row limits for previews.
- Config validation before execution.
- Artifact and log paths constrained to workspace-approved locations.

### 9.2 Model monitoring guardrails

Mandos-AI should not claim:

- “Model is healthy” without monitoring outputs.
- “Drift is root-caused by X” without evidence.
- “Performance degraded” without labels, proxies, or estimated-performance methodology.
- “No issues found” if only one section ran.
- “Production is safe” from dev/test runs.

Use language like:

```text
Based on the Mandos monitor JSON, the DQ section passed, drift failed due to score PSI, and model performance was not evaluated because labels are delayed.
```

Avoid:

```text
The model is bad because score PSI failed.
```

Better:

```text
The score distribution changed enough to exceed the configured PSI threshold. This is a monitoring signal requiring investigation, not by itself proof that realized credit performance has degraded.
```

### 9.3 Copilot edit guardrails

Require Copilot to:

- Keep changes small.
- Explain why each file changed.
- Update tests.
- Run relevant checks when `execute` is available.
- Avoid broad refactors unless explicitly requested.
- Preserve public APIs unless instructed otherwise.
- Avoid adding dependencies without approval.
- Prefer simple functions/classes over clever frameworks.

### 9.4 Evidence and audit fields

Every Mandos workflow should aim to preserve:

- `model_id`
- `model_version`
- `baseline_id`
- `run_id`
- `config_hash`
- `code_version` or git SHA
- `snapshot predicate`
- `source table names`
- `feature columns hash`
- `query tag`
- `run timestamp`
- `workflow name`
- `artifact paths`

---

## 10. Recommended Repository Layout

```text
mandos/
  .github/
    copilot-instructions.md
    agents/
      mandos-ai.agent.md
      mandos-reviewer.agent.md
      mandos-testing.agent.md
      mandos-docs.agent.md
  docs/
    architecture/
      mandos-ai-operating-model.md
      non-mcp-copilot-workflow.md
    onboarding/
      model-documentation-template.md
      config-field-guide.md
    operations/
      running-mandos-ai.md
      troubleshooting.md
  examples/
    configs/
      example_originations_model.yaml
      example_residual_value_model.yaml
    notebooks/
      profile_example.ipynb
      compare_example.ipynb
      monitor_example.ipynb
  src/
    mandos/
      ...
    mandos_ai/
      cli.py
      config.py
      result_contracts.py
      commands.py
  tests/
    unit/
    integration/
  Makefile
  pyproject.toml
  README.md
  AGENTS.md
```

### 10.1 `AGENTS.md`

`AGENTS.md` should summarize durable repo rules:

```markdown
# Mandos Agent Instructions

Mandos is a config-driven Python library for model monitoring using Snowflake/Snowpark.

Principles:
- KISS, YAGNI, SOLID.
- Clarity over cleverness.
- No metric → no claim.
- Do not duplicate Mandos logic in wrappers.
- Prefer config-driven workflows.
- Push large-table computation to Snowflake/Snowpark.
- Use pandas only for summary-level outputs.
- Keep reporting separate from metric computation.

Approved commands:
- make test
- make lint
- make typecheck
- make mandos-profile CONFIG=<path>
- make mandos-compare CONFIG=<path>
- make mandos-monitor CONFIG=<path>
```

### 10.2 `.github/copilot-instructions.md`

This should contain high-level, always-on instructions:

```markdown
# Copilot Instructions for Mandos

When assisting with this repository:

- Treat Mandos as an evidence-grade model monitoring library.
- Do not invent APIs; inspect the code first.
- Prefer YAML/Pydantic config-driven behavior.
- Use Snowpark for large-table operations.
- Avoid loading full Snowflake tables into pandas.
- Keep wrappers thin and boring.
- Use Black, Ruff, Mypy, pytest, and NumPy docstrings.
- Separate computation, orchestration, visualization, and reporting.
- Do not make unsupported claims about model health.
```

---

## 11. Example Mandos Custom Agents

### 11.1 Mandos planner

```yaml
---
name: mandos-planner
description: Creates technical plans for Mandos changes without implementation
tools: ["read", "search"]
---

You are a staff-level Mandos planning agent.

Your job is to inspect the repository and produce clear implementation plans.
Do not edit files. Do not run commands. Do not invent APIs.

Every plan must include:
- Problem statement
- Existing relevant files/APIs
- Proposed design
- File-by-file changes
- Risks
- Test plan
- Acceptance criteria
```

### 11.2 Mandos reviewer

```yaml
---
name: mandos-reviewer
description: Reviews Mandos code quality, structure, maintainability, and safety
tools: ["read", "search"]
---

You are a staff-level Python reviewer for Mandos.

Review for:
- KISS / YAGNI
- SOLID boundaries
- Pythonic structure
- duplicated code
- dead code
- unsafe Snowflake patterns
- poor config validation
- missing tests
- unclear logging

Do not implement. Produce a findings report with severity, evidence, and suggested fixes.
```

### 11.3 Mandos testing agent

```yaml
---
name: mandos-testing
description: Improves Mandos test coverage and quality
tools: ["read", "search", "edit", "execute"]
---

You are a Mandos testing specialist.

Allowed commands:
- make test
- make lint
- make typecheck

Focus on meaningful tests, not coverage theater.
Add regression tests for bugs. Mock Snowflake for unit tests. Use integration tests only where necessary.
```

### 11.4 Mandos AI operator without MCP

```yaml
---
name: mandos-ai-operator
description: Runs approved config-driven Mandos workflows without MCP
tools: ["read", "search", "execute"]
---

You are a Mandos-AI operator.

MCP is not available. You may not call mandos/profile or company-snowflake tools.
Use only approved CLI or Makefile commands.

Approved commands:
- make mandos-check-session CONFIG=<path>
- make mandos-build-baseline CONFIG=<path>
- make mandos-profile CONFIG=<path>
- make mandos-compare CONFIG=<path>
- make mandos-monitor CONFIG=<path>
- make mandos-report CONFIG=<path>

Summarize only returned JSON and referenced artifacts. Do not run arbitrary SQL.
```

---

## 12. Future MCP Migration Path

The non-MCP wrapper should be designed so it can later become an MCP server without changing Mandos itself.

### 12.1 Today: CLI wrapper

```text
mandos-ai profile --config configs/model.yaml --output json
```

### 12.2 Later: MCP tool

```yaml
tools:
  - "mandos/profile"
  - "mandos/compare"
  - "mandos/monitor"
```

MCP tool schema:

```json
{
  "name": "profile",
  "description": "Run Mandos data quality profile using a validated config file.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "config_path": {
        "type": "string",
        "description": "Path to a Mandos YAML config in the repository."
      },
      "output": {
        "type": "string",
        "enum": ["json"],
        "default": "json"
      }
    },
    "required": ["config_path"]
  }
}
```

### 12.3 Keep business logic below both interfaces

```text
CLI command ┐
            ├── MandosAIService.profile(config_path) ── Mandos public API
MCP tool  ──┘
```

Do not build separate logic for CLI and MCP.

---

## 13. Practical Workflows

### 13.1 Onboard a new model

```text
1. User provides model documentation and table details.
2. Copilot drafts YAML config from template.
3. Copilot validates config with approved command.
4. Human reviews config.
5. Copilot runs check-session.
6. Copilot runs build-baseline if approved.
7. Copilot runs profile/compare/monitor as appropriate.
8. Copilot summarizes JSON evidence.
9. Human reviews results and artifacts.
10. Config and evidence are committed or persisted according to policy.
```

### 13.2 Pre-launch development validation

Use:

```bash
make mandos-profile CONFIG=configs/model_dev.yaml
make mandos-compare CONFIG=configs/model_dev.yaml
make mandos-report CONFIG=configs/model_dev.yaml
```

Output should be framed as development validation, not production monitoring.

### 13.3 Production monitoring

Use:

```bash
make mandos-monitor CONFIG=configs/model_prod.yaml
make mandos-report CONFIG=configs/model_prod.yaml
```

Output should include:

- DQ status.
- Drift status.
- Model performance status or label-delay status.
- Top issues.
- Artifact paths.
- Run metadata.
- Config hash.

### 13.4 Troubleshooting a failed run

Copilot should inspect:

1. JSON status.
2. Run logs.
3. Config validation errors.
4. Snowflake connection errors.
5. Missing table/column errors.
6. Mandos stack trace.
7. Recent code changes.
8. Tests around failing workflow.

Prompt:

```text
Investigate why the approved Mandos monitor command failed for configs/zuul_2_0.yaml.
Read the JSON output and logs. Do not rerun with modified arguments unless the fix is clear.
Identify root cause, affected config/code, and minimal fix.
```

---

## 14. Anti-Patterns

Avoid these patterns.

### 14.1 Agent improvises Python entrypoints

Bad:

```bash
python - <<'PY'
from mandos import *
session = create_session(...)
# many improvised arguments
PY
```

Better:

```bash
make mandos-monitor CONFIG=configs/model.yaml
```

### 14.2 Agent runs direct SQL

Bad:

```bash
python run_sql.py "drop table ..."
```

Better:

```bash
mandos-ai monitor --config configs/model.yaml --output json
```

### 14.3 Reporting code calculates metrics

Bad:

```python
def build_pdf(session, table):
    # queries data, computes metrics, plots charts, writes PDF
```

Better:

```text
metrics layer → summary model → chart layer → report layer
```

### 14.4 Config schema is loose

Bad:

```python
config = yaml.safe_load(path)
threshold = config.get("threshold", "whatever")
```

Better:

```python
config = MandosConfig.model_validate(yaml.safe_load(path))
```

### 14.5 Copilot makes unsupported claims

Bad:

```text
The model is performing well.
```

Better:

```text
The monitor output shows DQ passed and drift had no configured failures. Model performance was not evaluated because labels are delayed.
```

---

## 15. Recommended Standards for Mandos-AI Outputs

Every Copilot-generated Mandos summary should include:

```text
- Workflow run
- Model ID/version
- Data snapshot/baseline identifiers
- Overall status
- Data quality status
- Drift status
- Performance status or label availability
- Top issues by severity
- Artifact paths
- What was not evaluated
- Next recommended human action
```

Example:

```markdown
## Mandos Monitor Summary

**Model:** zuul_2_0 / 2026.05  
**Run:** 2026-05-07T12:01:33Z  
**Status:** Warning

### Findings

- Data quality: Warning, 4 issues.
- Drift: Failed, 2 issues.
- Model performance: Not evaluated because labels are delayed.

### Top issue

`MODEL_SCORE` PSI = 0.31 exceeded the configured failure threshold of 0.25.

### Artifacts

- JSON: artifacts/mandos/zuul_2_0/latest/summary.json
- PDF: artifacts/mandos/zuul_2_0/latest/report.pdf
- Logs: artifacts/mandos/zuul_2_0/latest/run.log

### Caveat

This run identifies distribution drift. It does not prove realized credit performance degradation because final labels are delayed.
```

---

## 16. Implementation Checklist

### 16.1 Minimum viable non-MCP setup

- [ ] Create `.github/copilot-instructions.md`.
- [ ] Create `AGENTS.md`.
- [ ] Create at least one Mandos custom agent under `.github/agents`.
- [ ] Add example YAML configs.
- [ ] Add config validation command.
- [ ] Add `mandos-ai` CLI or Makefile targets.
- [ ] Ensure CLI returns compact JSON.
- [ ] Add tests for config validation.
- [ ] Add tests for JSON output contract.
- [ ] Add docs for approved commands.
- [ ] Add guardrails against arbitrary SQL.
- [ ] Add query tagging.
- [ ] Add artifact path conventions.

### 16.2 Production-readiness checklist

- [ ] Read-only default Snowflake role.
- [ ] Approved connection names.
- [ ] No secrets in YAML.
- [ ] Environment variable validation.
- [ ] Config hash persisted with run.
- [ ] Run ID included in outputs.
- [ ] Logs contain model ID/version and query tag.
- [ ] PDF/JSON artifacts include metadata.
- [ ] Human review process for baseline creation.
- [ ] Alert severity definitions documented.
- [ ] Label-delay semantics documented.
- [ ] Regression tests for known historical failure modes.

### 16.3 Future MCP checklist

- [ ] Extract CLI logic into `MandosAIService`.
- [ ] Define MCP schemas for `profile`, `compare`, `monitor`, `build_baseline`.
- [ ] Allowlist read-only tools first.
- [ ] Require config paths rather than raw arguments.
- [ ] Avoid exposing arbitrary SQL tools.
- [ ] Add sandboxing where supported.
- [ ] Add audit logs for MCP calls.
- [ ] Keep CLI and MCP behavior consistent.

---

## 17. Bottom-Line Recommendation

For Mandos-AI today:

```text
Make Copilot aware of Mandos through instructions and examples.
Expose Mandos operations through thin config-first CLI/Makefile wrappers.
Use built-in Copilot tools for repo work.
Use `execute` only for approved commands.
Do not rely on model reasoning for evidence-grade monitoring claims.
Prepare the wrapper/service layer so it can become MCP later.
```

This gives the team useful Copilot workflows immediately without waiting for MCP, while preserving a clean migration path to structured MCP tools later.

---

## 18. Sources and References

- GitHub Docs — Custom agents configuration: <https://docs.github.com/en/copilot/reference/custom-agents-configuration>
- GitHub Docs — Creating custom agents for Copilot cloud agent: <https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/create-custom-agents>
- GitHub Docs — Extending Copilot cloud agent with MCP: <https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/extend-cloud-agent-with-mcp>
- GitHub Docs — Copilot CLI tool permissions: <https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/allowing-tools>
- VS Code Docs — Use tools with agents: <https://code.visualstudio.com/docs/copilot/agents/agent-tools>
- VS Code Docs — Add and manage MCP servers: <https://code.visualstudio.com/docs/copilot/customization/mcp-servers>
- Model Context Protocol Specification — Tools: <https://modelcontextprotocol.io/specification/draft/server/tools>
- Snowflake Docs — Creating a Session for Snowpark Python: <https://docs.snowflake.com/en/developer-guide/snowpark/python/creating-session>
- Snowflake Docs — Working with DataFrames in Snowpark Python: <https://docs.snowflake.com/en/developer-guide/snowpark/python/working-with-dataframes>
- pandas Docs — 10 minutes to pandas: <https://pandas.pydata.org/docs/user_guide/10min.html>
- Matplotlib Docs — `savefig`: <https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.savefig.html>
- ReportLab PyPI project page: <https://pypi.org/project/reportlab/>
- Pydantic Settings Docs: <https://pydantic.dev/docs/validation/latest/concepts/pydantic_settings/>
