# Mandos-AI Agent Architecture and Tooling Design

## Purpose

This document expands the Mandos-AI phased implementation report with a concrete agent architecture, individual agent responsibilities, tool boundaries, handoff patterns, and implementation sequencing.

The key recommendation is to **start with one broad GitHub Copilot custom agent** and split into multiple specialized agents only after real user behavior proves the need.

Today, Mandos is still in early adoption:

- No models have officially launched on Mandos yet.
- Zuul 2.0 is preparing to become the first major launch candidate.
- Users are still learning how to configure Mandos, define baselines, assign FeatureRules, segment data, run `monitor().report()`, and interpret report outputs.
- The immediate opportunity is setup guidance, validation support, report explanation, and alert triage.

Therefore, the first agent should be:

```text
mandos-analyst
```

Future specialized agents should be introduced only after the initial Mandos-AI workflows are stable and useful.

---

## Core Mandos-AI doctrine

All agents must follow the same evidence-first doctrine.

### Evidence rule

Mandos-AI may reason from memory, but it must not report facts from memory.

Facts include:

- Metric values
- Metric definitions
- Thresholds
- FeatureRules
- Baseline definitions
- Model health scores
- Run dates
- Segment-level findings
- Data quality failures
- Drift results
- Model performance values
- Proxy metric values
- Report conclusions

Facts must come from:

- Mandos config files
- Mandos codebase definitions
- Mandos generated reports
- Mandos persisted Snowflake tables
- Approved Mandos documentation
- User-provided files
- Trusted external documentation when relevant

Core rules:

```text
No metric -> no claim.
No source -> no fact.
No persisted evidence -> no production conclusion.
```

### Response taxonomy

When summarizing evidence, every Mandos-AI agent should separate:

```text
FACT
A sourced observation from data, config, code, report, or persisted evidence.

INTERPRETATION
Reasoned explanation of what the facts likely mean.

RECOMMENDATION
Suggested next action, diagnostic path, or remediation step.
```

### Alert-fatigue doctrine

Mandos is intentionally broad in detection and narrow in escalation.

Agents should classify findings into:

```text
Action Required
Watchlist
Informational
Suppressed / Likely Noise
```

Mandos-AI should help users determine:

- Is this issue new?
- Is it persistent?
- Is it worsening?
- Is it material?
- Is it concentrated in a high-volume or high-risk segment?
- Is it near a business decision boundary?
- Is it tied to other issues?
- Is it likely one root cause expressed through many downstream alerts?

---

## Agent strategy

## Phase A: Single-agent beginning

Start with one agent:

```text
mandos-analyst.agent.md
```

This avoids over-engineering and keeps the user experience simple.

The initial `mandos-analyst` agent should support:

- Mandos concept explanations
- Baseline setup guidance
- FeatureRules guidance
- Segmentation guidance
- Config generation
- Notebook/script generation
- `monitor().report()` explanation
- Zuul 2.0 validation support
- Basic alert triage
- Custom report planning

This is the correct starting point because most users are not yet asking advanced production-history questions. They are asking how to get started.

## Phase B: Specialized agent split

Split into specialized agents only when one or more of the following becomes true:

- The `mandos-analyst` instructions become too large.
- Users frequently ask clearly different categories of questions.
- Tool access needs differ by role.
- Engineering tasks need separation from model-owner tasks.
- MCP tools need stricter access control by agent.
- Report-building workflows become materially different from setup workflows.

---

# Future agent catalog

## 1. `mandos-analyst`

### Role

Primary entrypoint and router for all Mandos-AI workflows.

### Target users

- Model owners
- Data scientists
- Model monitoring users
- Zuul 2.0 pilot team
- New Mandos adopters

### Primary responsibilities

- Explain Mandos concepts.
- Help users choose the right workflow: `profile`, `compare`, or `monitor`.
- Help users set up Mandos jobs.
- Route specialized requests to the right future agent.
- Keep answers grounded in Mandos doctrine.

### Typical user questions

```text
How do I set up Mandos for my model?
What is a baseline?
How do I define FeatureRules?
Should I use profile(), compare(), or monitor()?
How do I generate monitor().report()?
What should I worry about in this report?
```

### Inputs

- User description of model
- Existing config files
- Mandos documentation
- Example notebooks
- Report artifacts
- Future: persisted monitoring results

### Outputs

- Explanations
- Setup checklists
- Starter configs
- Notebook/script outlines
- Report summaries
- Handoff recommendations

### Initial tools

Built-in Copilot tools only:

```text
read
search
edit
```

Optional later tools:

```text
mandos.validate_config
mandos.get_latest_run_summary
mandos.get_baseline_context
mandos.triage_issues
```

### Prohibited behavior

- Do not invent metric values.
- Do not invent thresholds.
- Do not claim a model is healthy/unhealthy without sourced evidence.
- Do not query raw production data unless explicitly approved through controlled tools.
- Do not write to Mandos persistence tables.

---

## 2. `mandos-config-architect`

### Role

Specialist for Mandos config, baselines, FeatureRules, and segmentation.

### Target users

- Model owners onboarding a new model
- Data scientists preparing for validation
- Mandos core team reviewing config quality

### Primary responsibilities

- Generate starter configs.
- Review existing configs.
- Identify missing required fields.
- Suggest FeatureRules drafts.
- Identify risky or weak baseline definitions.
- Recommend segmentation strategy.
- Flag overcomplicated or noisy configurations.

### Typical user questions

```text
Review this config. What is missing?
Are these FeatureRules strong enough?
What segments should I monitor?
Is this baseline appropriate?
What config fields are required for a classification model?
```

### Inputs

- YAML config
- Model metadata
- Feature list
- Baseline/current table names
- Feature metadata
- Report output if available
- Future: Snowflake table metadata

### Outputs

- Config review
- Config diff recommendations
- Draft config file
- FeatureRules draft
- Segmentation recommendation
- Validation-readiness checklist

### Future MCP tools

Read-only tools:

```text
mandos.validate_config
mandos.get_config_schema
mandos.describe_baseline
snowflake.describe_table
snowflake.list_allowed_columns
```

Analysis tools:

```text
mandos.suggest_feature_rules
mandos.check_feature_coverage
mandos.check_segment_cardinality
```

Write tools, later and gated:

```text
mandos.create_config_draft
mandos.save_config_template
```

### Tool safety

This agent should not create official baselines or persist official monitoring runs. It may generate draft artifacts for user review.

---

## 3. `mandos-eda-analyst`

### Role

Specialist for using Mandos to explore Pandas or Snowflake data.

### Target users

- Data scientists exploring model development data
- Teams preparing baseline/current comparisons
- Users who want Mandos-style EDA before production monitoring

### Primary responsibilities

- Help users run Mandos profiling workflows.
- Guide Pandas and Snowflake/Snowpark setup.
- Explain data quality findings.
- Identify suspicious patterns.
- Recommend next EDA steps.

### Typical user questions

```text
How do I use Mandos to explore this Pandas DataFrame?
Can I profile a Snowflake table?
Which features have the worst data quality?
Do I have whitespace issues or invalid categories?
What should I inspect before creating a baseline?
```

### Inputs

- Pandas DataFrame code/context
- Snowflake table names
- FeatureRules
- Profile outputs
- Compare outputs

### Outputs

- EDA notebook
- Profile workflow
- Data quality summary
- Issue interpretation
- Recommended follow-up checks

### Future MCP tools

Read-only tools:

```text
snowflake.describe_table
snowflake.get_column_profile_summary
mandos.validate_profile_request
```

Analysis tools:

```text
mandos.profile_pandas_summary
mandos.profile_snowflake_table_summary
mandos.compare_profile_summaries
```

### Prohibited behavior

- Do not pull full large Snowflake tables into memory.
- Do not expose customer-level records in default workflows.
- Do not treat ad hoc EDA observations as official monitoring evidence unless persisted.

---

## 4. `mandos-report-builder`

### Role

Specialist for custom reporting, charting, and validation evidence narratives.

### Target users

- Model owners preparing reports
- Zuul 2.0 validation users
- Model Validation stakeholders
- Leadership-facing users needing concise summaries

### Primary responsibilities

- Explain standard `monitor().report()` PDFs.
- Propose custom report structures.
- Recommend charts based on available evidence.
- Generate report scripts or notebooks.
- Create audience-specific narratives.
- Preserve sourced FACT / INTERPRETATION / RECOMMENDATION separation.

### Typical user questions

```text
Summarize this report for Model Validation.
Create an executive summary.
Create a custom report focused on data quality and drift.
What charts should I include?
Can we make a report similar to last month's PDF?
```

### Inputs

- Mandos report PDF or result object
- Report output tables
- Model config
- User-selected audience
- Prior report examples
- Future: persisted metrics/history

### Outputs

- Custom report outline
- Chart recommendations
- Python report-generation script
- Executive summary
- Validation evidence summary
- Appendix structure

### Future MCP tools

Read-only tools:

```text
mandos.get_report_metadata
mandos.get_run_summary
mandos.get_metric_snapshot
mandos.get_issue_inventory
```

Analysis tools:

```text
mandos.generate_chart_data
mandos.summarize_report_findings
mandos.build_report_outline
```

Write tools, later and gated:

```text
mandos.export_custom_report_pdf
mandos.export_custom_report_html
```

### First chart families

- Health score summary
- Issue severity summary
- Action/watchlist/informational/suppressed issue counts
- FeatureRules pass/fail table
- Top data quality issues
- Top drifting features
- Baseline vs. current score distribution
- Segment-level issue heatmap
- Alert triage table

---

## 5. `mandos-alert-triage-analyst`

### Role

Specialist for alert fatigue reduction and issue prioritization.

### Target users

- Model owners reviewing noisy monitoring output
- Mandos core team tuning alerting strategy
- Validation users assessing materiality

### Primary responsibilities

- Prioritize issues.
- Identify likely noise.
- Identify recurring issues.
- Group related alerts into root-cause stories.
- Separate action-required findings from watchlist items.
- Provide transparent suppression rationale.

### Typical user questions

```text
What should I worry about in the latest run?
Which issues are material?
Which threshold breaches are probably noise?
Are these alerts connected?
What should be on the watchlist?
```

### Inputs

- Mandos report output
- Issue inventory
- Metric values
- Thresholds
- Segment volumes
- Feature importance metadata if available
- Run history if available

### Outputs

- Prioritized issue list
- Watchlist
- Suppression rationale
- Root-cause grouping hypothesis
- Recommended investigation path

### Future MCP tools

Read-only tools:

```text
mandos.get_issue_inventory
mandos.get_metric_snapshot
mandos.get_segment_volume_context
mandos.get_threshold_context
```

Analysis tools:

```text
mandos.triage_issues
mandos.group_related_issues
mandos.get_recurring_issues
mandos.compute_materiality_score
```

### Triage dimensions

- Severity
- Persistence
- Magnitude
- Distance from threshold
- Affected row count
- Affected row percentage
- Segment importance
- Feature importance
- Proximity to decision cutoffs
- New versus recurring
- Improving versus worsening
- Known pipeline/model/config changes

---

## 6. `mandos-monitoring-analyst`

### Role

Specialist for historical monitoring analysis from persisted Mandos evidence.

### Target users

- Model owners after launch
- Mandos dashboard users
- Model Validation users
- Leadership stakeholders reviewing production health

### Primary responsibilities

- Analyze model health over time.
- Compare runs.
- Summarize quarters/years.
- Detect recurring issues.
- Detect change points.
- Explain when degradation began.
- Recommend investigation anchors.

### Typical user questions

```text
What changed between March and April?
What changed over the last quarter?
What changed over the last year?
When did this model start degrading?
Which issues keep recurring?
What was the last healthy run?
Did remediation help?
```

### Inputs

- `MANDOS_BASELINES`
- `MANDOS_RUNS`
- `MANDOS_METRICS`
- `MANDOS_ISSUES`
- Curated `VW_*` views
- Baseline/config metadata

### Outputs

- Point-to-point comparison
- Period trend summary
- Change-point analysis
- Recurring issue summary
- Model health narrative
- Recommended investigation sequence

### Future MCP tools

Read-only tools:

```text
mandos.get_model_history
mandos.get_latest_run_summary
mandos.get_baseline_context
mandos.get_metric_trend
mandos.get_recurring_issues
```

Analysis tools:

```text
mandos.compare_runs
mandos.summarize_period
mandos.detect_change_points
mandos.explain_health_change
```

### Important limitation

This agent should not be released until there are real persisted monitoring runs. Before launch, it should clearly state that production trend analysis is unavailable.

---

## 7. `mandos-validation-reviewer`

### Role

Specialist for Model Validation readiness and evidence quality.

### Target users

- Model owners preparing for validation
- Mandos core team
- Validation reviewers

### Primary responsibilities

- Review whether the evidence package is sufficient.
- Identify missing assumptions.
- Identify weak baseline justification.
- Identify missing FeatureRules.
- Identify missing segment monitoring.
- Draft validation-ready summaries.
- Ensure claims are evidence-backed.

### Typical user questions

```text
Is this report ready for Model Validation?
What evidence is missing?
What assumptions should we document?
Does this prove the model is ready for launch?
What should go in the validation memo?
```

### Inputs

- Config
- Baseline documentation
- `monitor().report()` PDF
- Issue summary
- FeatureRules
- Model metadata
- Custom evidence report

### Outputs

- Validation-readiness checklist
- Evidence gap analysis
- Draft validation summary
- Risk register
- Required follow-up items

### Future MCP tools

Read-only tools:

```text
mandos.validate_config
mandos.get_report_metadata
mandos.get_baseline_context
mandos.get_issue_inventory
```

Analysis tools:

```text
mandos.assess_validation_readiness
mandos.check_evidence_completeness
mandos.generate_validation_summary
```

### Guardrail

This agent should not claim that Model Validation has approved anything. It can help prepare evidence, not approve the model.

---

## 8. `mandos-engineer`

### Role

Developer-facing agent for Mandos codebase implementation and quality.

### Target users

- Mandos developers
- Core team maintainers
- Copilot/Amazon Q coding workflows

### Primary responsibilities

- Inspect codebase quality.
- Write or update tests.
- Review design against KISS, YAGNI, SOLID.
- Improve documentation.
- Find bugs.
- Implement approved features.

### Typical user questions

```text
Review this module for maintainability.
Find test gaps.
Add unit tests for this function.
Refactor this code without overengineering.
Make this mypy-compliant.
```

### Inputs

- Mandos source code
- Tests
- Docs
- Existing prompts
- Design docs

### Outputs

- Code review findings
- Test plans
- Refactor proposals
- Implementation PR changes
- Documentation updates

### Tools

Built-in Copilot tools:

```text
read
search
edit
terminal, if allowed
```

Future tools:

```text
pytest.run
ruff.check
mypy.check
coverage.report
```

### Important separation

This agent should be separate from user-facing Mandos model monitoring agents. Model owners should not be routed into codebase refactoring workflows.

---

# Tool access model

## Tool tiers

## Tier 0: No external tools

Used in early phases.

Capabilities:

- Read documentation
- Search repo
- Generate configs
- Generate notebooks
- Explain reports if user provides report content

Best for:

- Setup coach
- Zuul 2.0 companion
- Onboarding

## Tier 1: Read-only Mandos tools

Safe default tools.

Examples:

```text
mandos.validate_config
mandos.get_config_schema
mandos.describe_baseline
mandos.list_model_runs
mandos.get_latest_run_summary
mandos.get_metric_trend
```

Best for:

- Config review
- Baseline explanation
- Historical run lookup
- Report explanation

## Tier 2: Analysis tools

Read-only but computationally more involved.

Examples:

```text
mandos.compare_runs
mandos.summarize_period
mandos.triage_issues
mandos.detect_change_points
mandos.generate_chart_data
```

Best for:

- Trend analysis
- Alert triage
- Custom report data prep

## Tier 3: Write or official-action tools

Requires explicit user confirmation and likely platform approval.

Examples:

```text
mandos.create_baseline
mandos.persist_monitoring_run
mandos.export_official_report
mandos.update_model_registry
```

Best for:

- Later operational assistant
- Dashboard integration
- Controlled service workflows

Early Mandos-AI should avoid Tier 3.

---

# Agent-to-tool matrix

| Agent | Early built-in tools | Future read-only MCP tools | Future analysis tools | Future write tools |
|---|---|---|---|---|
| `mandos-analyst` | read, search, edit | validate_config, get_latest_run_summary, get_baseline_context | triage_issues | none initially |
| `mandos-config-architect` | read, search, edit | validate_config, get_config_schema, describe_table | suggest_feature_rules, check_segment_cardinality | save_config_draft |
| `mandos-eda-analyst` | read, search, edit | describe_table, get_column_profile_summary | profile_snowflake_table_summary, compare_profile_summaries | none initially |
| `mandos-report-builder` | read, search, edit | get_report_metadata, get_run_summary, get_issue_inventory | generate_chart_data, build_report_outline | export_custom_report_pdf, gated |
| `mandos-alert-triage-analyst` | read, search, edit | get_issue_inventory, get_threshold_context | triage_issues, group_related_issues | none |
| `mandos-monitoring-analyst` | read, search | get_model_history, get_metric_trend, get_recurring_issues | compare_runs, summarize_period, detect_change_points | none |
| `mandos-validation-reviewer` | read, search, edit | get_baseline_context, get_report_metadata, get_issue_inventory | assess_validation_readiness | none |
| `mandos-engineer` | read, search, edit, terminal if allowed | none required | pytest, ruff, mypy, coverage | code edits only by user request |

---

# Recommended handoff patterns

## Setup to validation

```text
User: Help me set up Mandos for Zuul 2.0.
mandos-analyst -> mandos-config-architect
mandos-config-architect creates/reviews config.
mandos-analyst returns setup summary and next steps.
mandos-validation-reviewer checks evidence readiness.
```

## Report to triage

```text
User: What should I worry about in this report?
mandos-report-builder extracts findings.
mandos-alert-triage-analyst classifies materiality.
mandos-report-builder creates summary language.
```

## Historical trend to custom report

```text
User: What changed over the last quarter? Create a report.
mandos-monitoring-analyst summarizes period.
mandos-alert-triage-analyst prioritizes findings.
mandos-report-builder generates custom report outline and charts.
```

## Engineering separation

```text
User: Implement the missing tool.
mandos-analyst hands off to mandos-engineer.
mandos-engineer handles code/tests/docs.
```

---

# Implementation sequence

## Step 1: Create only `mandos-analyst`

The first release should have a single custom agent.

Files:

```text
.github/copilot-instructions.md
.github/agents/mandos-analyst.agent.md
docs/agent_context/*.md
```

Do not create multiple agents yet.

## Step 2: Add agent tests through golden prompts

Create a prompt test catalog:

```text
docs/agent_eval/golden_prompts.md
```

Example tests:

```text
Prompt: Help me set up Mandos for a classification model.
Expected: asks for baseline/scored tables, score column, entity key, features, segments, FeatureRules.

Prompt: What changed over the last year for Zuul?
Expected: refuses to claim trend history because no production runs exist yet.

Prompt: The report has 20 warnings. Is the model broken?
Expected: explains alert triage and asks for/report evidence before factual claims.
```

## Step 3: Add specialized docs before specialized agents

Before creating new agents, create role-specific docs:

```text
docs/agent_context/config_architect_guide.md
docs/agent_context/report_builder_guide.md
docs/agent_context/alert_triage_guide.md
docs/agent_context/monitoring_history_guide.md
```

Let the single `mandos-analyst` use those docs first.

## Step 4: Split agents only when needed

Once usage grows, split the most valuable workflow first.

Likely first split:

```text
mandos-config-architect
mandos-report-builder
```

Then later:

```text
mandos-monitoring-analyst
mandos-alert-triage-analyst
mandos-validation-reviewer
mandos-engineer
```

---

# Success criteria

## Initial single-agent release

A new Mandos user can ask:

```text
Help me set up Mandos for my model.
```

And receive:

- A clear setup path
- A starter config
- A notebook/script outline
- Baseline guidance
- FeatureRules guidance
- Segmentation guidance
- `monitor().report()` instructions
- A checklist of assumptions requiring user review

## Zuul 2.0 pilot

The Zuul team can ask:

```text
Summarize this monitor().report() PDF for validation.
```

And receive:

- Facts from the report
- Interpretations clearly labeled
- Recommendations clearly labeled
- Action items
- Watchlist items
- Likely noise/suppression rationale

## Future persistence-aware release

A model owner can ask:

```text
What changed over the last quarter?
```

And receive:

- Date window used
- Number of runs analyzed
- Health trend
- Top worsening metrics
- Recurring issues
- New/resolved issues
- Materiality assessment
- Recommended investigation path

---

# Final recommendation

Do not start with many agents.

Start with:

```text
mandos-analyst.agent.md
```

Then build:

1. Strong doctrine
2. Strong setup guidance
3. Strong Zuul 2.0 validation support
4. Strong report explanation
5. Strong alert triage
6. Strong custom reporting
7. Persistence-aware tools
8. Specialized agents

Mandos-AI should first become the best possible onboarding and validation companion. Once real monitoring history exists, it can become a true historical model health analyst.
