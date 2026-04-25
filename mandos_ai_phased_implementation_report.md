# Mandos-AI Phased Implementation Report

## Executive Summary

Mandos-AI should begin as a **GitHub Copilot custom agent that accelerates Mandos adoption**, not as a fully autonomous monitoring platform. Today, no models have officially launched on Mandos yet. **Zuul 2.0** is the first meaningful pre-launch pilot, and users are primarily trying to understand how to configure Mandos, set up baselines, assign FeatureRules, segment data, run `monitor().report()`, and interpret the resulting PDF.

That reality should drive the implementation path.

The near-term goal is:

> **Help new users set up Mandos correctly, run validation-style monitoring reports, understand the findings, and prepare evidence for Model Validation.**

The long-term goal is:

> **Use persisted Snowflake monitoring evidence to let Mandos-AI explain model health trends, recurring issues, alert materiality, and investigation paths.**

GitHub Copilot custom agents are a strong starting point because they can be defined with Markdown agent profiles using YAML frontmatter, including instructions, tool access, and eventually MCP configuration. This gives Mandos a path from one simple agent to a future multi-agent system.

---

## 1. Current Mandos-AI Context

### 1.1 Current Mandos Adoption State

Current state:

```text
- No models have officially launched on Mandos yet.
- Zuul 2.0 is preparing to become the first major Mandos launch candidate.
- The Zuul team has already been using monitor().report() PDFs for validation before release.
- Most future users are still learning Mandos basics.
- The highest-friction areas are setup, config design, baselines, FeatureRules, segmentation, and report interpretation.
```

This means Mandos-AI should **not** start with:

```text
- Full MCP-backed Snowflake querying
- Multi-agent orchestration
- Dashboard integration
- Autonomous monitoring execution
- Trend analysis over long production history
```

Those are future phases.

It should start with:

```text
- Mandos setup guidance
- Config generation
- Baseline explanation
- FeatureRules guidance
- Segmentation guidance
- monitor().report() explanation
- Zuul 2.0 validation support
```

### 1.2 Product Thesis

Mandos-AI should become:

> **A trusted, evidence-aware model monitoring assistant for Mandos users.**

It should help users move from:

```text
“I don’t know how to use Mandos.”
```

to:

```text
“I have a validated config, a baseline, FeatureRules, a report, and a clear summary of what matters.”
```

Eventually, once Mandos has persisted production history, it should help users move from:

```text
“This model has 47 monitoring signals.”
```

to:

```text
“Only three issues are material; two are recurring; one started after the March data change; here is the evidence and recommended investigation path.”
```

---

## 2. Mandos-AI Core Doctrine

This doctrine should be documented before implementation and treated as non-negotiable.

### 2.1 Evidence Rule

Mandos-AI must follow this rule:

> **Mandos-AI may reason from memory, but it must not report facts from memory.**

Facts include:

```text
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
```

Facts must come from:

```text
- Mandos config files
- Mandos codebase definitions
- Mandos generated reports
- Mandos persisted Snowflake tables
- Approved Mandos documentation
- User-provided files
- Trusted external documentation when relevant
```

The core rule:

```text
No metric → no claim.
No source → no fact.
No persisted evidence → no production conclusion.
```

### 2.2 Response Taxonomy

Mandos-AI should label output using three categories:

```text
FACT
A sourced observation from data, config, code, report, or persisted evidence.

INTERPRETATION
Reasoned explanation of what the facts likely mean.

RECOMMENDATION
Suggested next action, investigation path, or remediation step.
```

Example:

```text
FACT
The April monitoring report shows SCORE_BAND PSI = 0.31.

INTERPRETATION
That is a meaningful score distribution shift under common PSI conventions and should be investigated if it is persistent, material, or concentrated near decision cutoffs.

RECOMMENDATION
Start by checking segment mix, score-band movement near cutoffs, feature drift, and upstream data quality issues.
```

### 2.3 Alert Fatigue Doctrine

Mandos-AI should not amplify every alert.

It should classify findings as:

```text
Action required
Watchlist
Informational
Suppressed / likely noise
```

Core triage rules:

```text
- Not every threshold breach is an incident.
- Persistent + material + worsening issues should be prioritized.
- One root cause should become one story, not ten alerts.
- Suppression should be transparent, never silent.
- Record everything, escalate only what matters.
```

---

## 3. Recommended Target Architecture

### 3.1 Near-Term Architecture

Near-term Mandos-AI should be simple:

```text
GitHub Copilot custom agent
        ↓
Mandos repo instructions and docs
        ↓
Mandos examples, configs, notebooks, scripts
        ↓
User-reviewed generated artifacts
```

This phase does **not** require MCP or Bedrock.

### 3.2 Mid-Term Architecture

Once usage is proven:

```text
GitHub Copilot custom agent
        ↓
Mandos docs + codebase + generated examples
        ↓
Optional local Mandos utilities
        ↓
Report explanation and custom reporting scripts
```

### 3.3 Long-Term Architecture

Once Mandos has persisted production history:

```text
GitHub Copilot custom agent
        ↓
Mandos MCP server
        ↓
Read-only controlled tools
        ↓
MANDOS_BASELINES
MANDOS_RUNS
MANDOS_METRICS
MANDOS_ISSUES
VW_* semantic views
        ↓
Trend analysis, issue recurrence, alert triage
```

---

## 4. Implementation Phases

---

## Phase 0 — Mandos-AI Foundation

### Objective

Make the Mandos repo agent-ready.

This phase creates durable instructions, doctrine, and reference material that Copilot will use to provide consistent Mandos-specific answers.

### Why This Phase Matters

Without this phase, Mandos-AI will behave like a generic coding assistant. It may produce plausible but inaccurate guidance. The goal is to make the agent understand Mandos’ opinionated design:

```text
- Snowflake-first monitoring
- Config-driven workflows
- Baseline persistence
- FeatureRules as hard constraints
- profile / compare / monitor separation
- monitor().report() as formal production-style reporting
- evidence-first claims
- alert triage over alert amplification
```

### Deliverables

```text
.github/
  copilot-instructions.md

.github/agents/
  mandos-analyst.agent.md

docs/agent_context/
  mandos_ai_doctrine.md
  mandos_api_quickstart.md
  mandos_config_schema.md
  baseline_setup_guide.md
  feature_rules_guide.md
  segmentation_guide.md
  monitor_report_guide.md
  alert_triage_guide.md
  zuul_2_validation_context.md
```

### `copilot-instructions.md` Responsibilities

This file should define repo-wide rules:

```text
- Mandos is a model monitoring and data quality library.
- Prefer simple, config-driven workflows.
- Do not invent metric values, thresholds, or model facts.
- Use FACT / INTERPRETATION / RECOMMENDATION when summarizing evidence.
- Favor Snowflake pushdown for large data.
- Pull back only small summaries.
- Avoid unnecessary abstraction.
- Follow KISS, YAGNI, clarity over cleverness.
- Generated code should be readable by data scientists, not only software engineers.
```

### `mandos-analyst.agent.md` Responsibilities

The first custom agent should be a single, broad Mandos guide:

```text
Agent name:
mandos-analyst

Primary responsibilities:
- Help users understand Mandos concepts.
- Help users configure baselines.
- Help users define FeatureRules.
- Help users choose segments.
- Generate starter configs.
- Generate runbook scripts.
- Generate notebooks.
- Explain monitor().report() outputs.
- Help distinguish material issues from noise.
```

### Acceptance Criteria

Phase 0 is complete when:

```text
- Copilot can answer Mandos setup questions using project-specific terminology.
- Copilot understands the difference between profile(), compare(), and monitor().
- Copilot knows that facts must come from data, configs, code, reports, or persisted evidence.
- Copilot can generate a starter Mandos config without hallucinating unsupported fields.
```

---

## Phase 1 — Mandos Setup Coach

### Objective

Help new users onboard to Mandos.

This is the highest-value initial feature because most users are still learning how to get started.

### Target User Questions

```text
How do I set up a baseline?
What is a good baseline?
How do I assign FeatureRules?
How do I segment my data?
What fields are required in the config?
How do I run monitor().report()?
How do I validate my model before launch?
```

### Core Workflow

Mandos-AI should guide users through this flow:

```text
1. Identify model type and usage.
2. Identify baseline table.
3. Identify current/scored table.
4. Identify entity key.
5. Identify run timestamp column.
6. Identify score column.
7. Identify optional target column.
8. Identify model features.
9. Identify important segments.
10. Define or draft FeatureRules.
11. Generate config.
12. Generate notebook or script.
13. Explain how to run monitor().report().
14. Provide validation checklist.
```

### Starter Config Output

Mandos-AI should produce configs like:

```yaml
MODEL_ID: zuul_2
MODEL_VERSION: "2.0"
MODEL_USAGE: classification_probability

BASELINE_TABLE: MLHUB_ORIGINATIONS.ZUUL_2_BASELINE
SCORED_TABLE: MLHUB_ORIGINATIONS.ZUUL_2_SCORED_VALIDATION

ENTITY_KEY: APPLICATION_ID
RUN_TIMESTAMP_COLUMN: SCORE_TS
SCORE_COLUMN: ZUUL_SCORE
TARGET_COLUMN: null

FEATURE_LIST:
  - FEATURE_A
  - FEATURE_B
  - FEATURE_C

CRITICAL_SEGMENTS:
  - PRODUCT
  - CHANNEL
  - STATE
  - RISK_TIER

FEATURE_RULES:
  FEATURE_A:
    dtype: FLOAT
    nullable: false
    valid_range:
      min: 0
      max: 1
    imputation:
      type: constant
      value: 0

  FEATURE_B:
    dtype: STRING
    nullable: false
    valid_categories:
      - A
      - B
      - C
```

Generated rules should always be marked as **drafts requiring model-owner review**.

### Starter Notebook Output

Mandos-AI should generate a notebook with sections:

```text
1. Purpose
2. Environment setup
3. Load Mandos config
4. Validate config
5. Run baseline setup or baseline check
6. Run monitor
7. Generate monitor().report()
8. Review summary
9. Review issues
10. Prepare validation notes
```

### Acceptance Criteria

Phase 1 is complete when a new user can ask:

```text
Help me set up Mandos for my model.
```

And receive:

```text
- A valid starter config
- A clear list of missing required inputs
- A runnable notebook or script
- A baseline setup explanation
- FeatureRules guidance
- Segmentation guidance
- monitor().report() instructions
```

---

## Phase 2 — Zuul 2.0 Validation Companion

### Objective

Use Zuul 2.0 as the first real Mandos-AI proving ground.

This should be the first applied use case because Zuul 2.0 is already using `monitor().report()` PDFs for validation before release.

### Key Principle

Mandos-AI should be honest about Zuul’s current state:

```text
Zuul 2.0 does not yet have long-term production monitoring history in Mandos.
Therefore, Mandos-AI can support pre-launch validation, config review, FeatureRules review, report explanation, and readiness assessment.
It cannot yet provide production trend analysis.
```

### Capabilities

Mandos-AI should support:

```text
- Review Zuul 2.0 config.
- Explain baseline choice.
- Review FeatureRules.
- Review segmentation strategy.
- Explain monitor().report() PDF sections.
- Summarize report findings.
- Identify material pre-launch risks.
- Separate likely noise from action-worthy findings.
- Draft Model Validation evidence summaries.
```

### Zuul-Specific Docs

Create:

```text
docs/agent_context/zuul_2_validation_context.md
examples/configs/zuul_2_starter_config.yaml
examples/notebooks/zuul_2_monitor_report_walkthrough.ipynb
examples/scripts/run_zuul_2_mandos_report.py
```

### Example User Prompts

```text
Review the Zuul 2.0 config. What is missing?
Does this baseline make sense for validation?
Are these FeatureRules strong enough?
What segments should Zuul monitor?
Summarize this monitor().report() PDF for Model Validation.
Which report findings are action-worthy before launch?
Which findings are likely noise?
```

### Acceptance Criteria

Phase 2 is complete when the Zuul team can use Mandos-AI to:

```text
- Understand their Mandos config.
- Understand their baseline.
- Understand their report.
- Prepare validation notes.
- Identify launch blockers versus watchlist items.
```

---

## Phase 3 — Reusable Mandos Onboarding Kit

### Objective

Convert the Zuul learning into reusable onboarding material for every future model team.

The goal is to avoid the Mandos core team having to explain the same setup concepts repeatedly.

### Deliverables

```text
docs/onboarding/
  model_owner_quickstart.md
  mandos_onboarding_checklist.md
  validation_readiness_checklist.md
  config_review_checklist.md
  feature_rules_examples.md
  segmentation_examples.md
  common_questions.md
  common_config_mistakes.md
```

### Common Questions To Cover

```text
What is a baseline?
When should I refresh a baseline?
What are FeatureRules?
Are FeatureRules the same as thresholds?
How do I choose segments?
What is the difference between data quality and drift?
What does monitor().report() include?
What should I show Model Validation?
What if my model does not have labels yet?
What if I only have scored data?
What if some features do not have policy rules?
```

### Config Review Checklist

Mandos-AI should check for:

```text
- Missing MODEL_ID
- Missing MODEL_VERSION
- Missing BASELINE_TABLE
- Missing SCORED_TABLE
- Missing ENTITY_KEY
- Missing SCORE_COLUMN
- Missing feature list
- FeatureRules missing for critical inputs
- Segments with high cardinality
- Target column included when labels are not available
- Weak or missing valid ranges
- Undefined category expectations
- Baseline table that does not represent the intended reference population
```

### Acceptance Criteria

Phase 3 is complete when:

```text
- A second model team can onboard using the docs and examples.
- Mandos-AI can answer common setup questions consistently.
- Mandos-AI can generate a credible first-pass config without direct core-team involvement.
```

---

## Phase 4 — Report Explainer and Validation Evidence Assistant

### Objective

Make `monitor().report()` easier to understand and easier to convert into validation evidence.

This is important because Zuul is already using the PDF for validation. Mandos-AI should make that report more valuable.

### Capabilities

Mandos-AI should help users answer:

```text
What does this report say?
What are the most important findings?
What should we fix before release?
What belongs in the validation summary?
Which findings are material?
Which are watchlist?
Which are likely noise?
What assumptions should we disclose?
```

### Standard Response Pattern

For report explanation, Mandos-AI should use:

```text
FACTS
Findings sourced directly from the report or result object.

INTERPRETATION
What those findings likely mean.

RECOMMENDATIONS
What the model owner should inspect, fix, document, or monitor.
```

### Report Summary Structure

Mandos-AI should generate:

```text
1. Executive summary
2. Model / run context
3. Baseline context
4. Data quality findings
5. Drift findings
6. Model performance findings, if labels exist
7. Proxy findings, if available
8. Segment-level findings
9. Material issues
10. Watchlist items
11. Likely noise / suppressed items
12. Recommended next steps
13. Appendix-ready evidence
```

### Acceptance Criteria

Phase 4 is complete when Mandos-AI can take report outputs and produce:

```text
- Model-owner summary
- Model Validation summary
- Executive summary
- Action item list
- Watchlist list
- Noise/suppression rationale
```

---

## Phase 5 — Custom Reporting and Visualization Builder

### Objective

Allow users to create custom reports and charts beyond the built-in `monitor().report()`.

The built-in report should remain the standard, official, repeatable report. Mandos-AI custom reporting should be audience-specific and investigation-specific.

### Report Types

Mandos-AI should support:

```text
Executive Summary Report
Short leadership-facing report focused on health, risk, and actions.

Model Validation Evidence Pack
Formal, sourced, with assumptions, limitations, thresholds, and appendix.

Model Owner Deep Dive
Technical report focused on features, segments, DQ, drift, and performance.

Alert Triage Report
Focused on action-required, watchlist, informational, and suppressed items.

Drift Investigation Report
Focused on PSI, CSI, score drift, feature drift, and segment mix.

Data Quality Investigation Report
Focused on FeatureRules, invalid values, missingness, cap/floor spikes, and schema issues.

Performance / Proxy Report
Focused on AUC, KS, RMSE, approval rate, delinquency proxies, score bands, and delayed-label monitoring.
```

### Initial Chart Library

Start with high-value charts:

```text
1. Issue severity summary
2. Action / watchlist / informational / suppressed issue counts
3. FeatureRules pass/fail table
4. Top data quality issues by feature
5. Top drifting features
6. Baseline vs. current score distribution
7. PSI by feature
8. Missing-rate delta by feature
9. Segment-level issue heatmap
10. Score-band population near decision cutoffs
```

### Custom Report Generation Flow

```text
1. User states audience and purpose.
2. Mandos-AI identifies available evidence.
3. Mandos-AI proposes report outline.
4. Mandos-AI proposes chart set.
5. User accepts or edits.
6. Mandos-AI generates script/notebook/report template.
7. User runs and reviews.
8. Mandos-AI helps refine language and visuals.
```

### Acceptance Criteria

Phase 5 is complete when a user can ask:

```text
Create a Model Validation evidence pack for Zuul 2.0 focused on data quality and drift.
```

And Mandos-AI produces:

```text
- Report outline
- Required data inputs
- Chart recommendations
- Python report-generation script
- Narrative template
- Appendix structure
```

---

## Phase 6 — Alert Triage and Noise Reduction

### Objective

Prevent Mandos from creating alert fatigue.

This should become a signature Mandos-AI capability.

### Why This Matters

Mandos is intentionally broad in detection. It may surface many DQ, drift, performance, and proxy signals. But users do not need every signal escalated equally.

Mandos-AI should help users answer:

```text
What should I actually worry about?
What can wait?
What is just noise?
What is recurring?
What is material?
What is new?
What is worsening?
What is likely one root cause?
```

### Triage Dimensions

Mandos-AI should prioritize issues using:

```text
- Severity
- Persistence
- Magnitude
- Distance from threshold
- Affected row count
- Affected row percentage
- Segment importance
- Feature importance
- Proximity to decision cutoffs
- Relationship to other issues
- New versus recurring
- Improving versus worsening
- Known model or data pipeline changes
```

### Triage Output

Mandos-AI should produce:

```text
Action Required
Issues requiring investigation now.

Watchlist
Issues that are not urgent but should be monitored.

Informational
Contextual findings not requiring action.

Suppressed / Likely Noise
Low-materiality or unreliable findings still recorded but not escalated.
```

### Example Response

```text
FACT
The report identified 18 findings: 3 critical, 7 warning, 5 informational, and 3 low-volume segment findings.

INTERPRETATION
Only 2 issues appear material for launch readiness. Both affect high-volume segments and are related to score distribution movement. The remaining warnings are either first-time breaches or low-volume segment effects.

RECOMMENDATION
Prioritize the score distribution shift and the missingness increase in critical bureau features. Keep the low-volume segment findings in the appendix as watchlist items.
```

### Acceptance Criteria

Phase 6 is complete when Mandos-AI can transform a noisy report into:

```text
- Prioritized issue list
- Watchlist list
- Suppression rationale
- Action plan
- Appendix-ready full issue inventory
```

---

## Phase 7 — Persistence-Aware Historical Monitoring Analyst

### Objective

Use Mandos’ future Snowflake persistence as the analytical moat.

This phase should begin only after at least one or more models have multiple persisted monitoring runs.

### Required Persisted Evidence

At minimum:

```text
MANDOS_BASELINES
MANDOS_RUNS
```

Preferably also:

```text
MANDOS_METRICS
MANDOS_ISSUES
VW_MODEL_RUN_HISTORY
VW_MODEL_HEALTH_TREND
VW_FEATURE_DQ_TREND
VW_FEATURE_DRIFT_TREND
VW_RECURRING_ISSUES
VW_SEGMENT_HEALTH_TREND
```

### User Questions

```text
What changed between March and April?
What changed over the last quarter?
What changed over the last year?
When did the model begin degrading?
Which issues are recurring?
Which features keep failing?
What was the last healthy run?
Did the latest run look worse than the prior three?
Did Zuul improve after remediation?
```

### Tool Concepts

```text
mandos.get_latest_run_summary
mandos.get_model_history
mandos.get_baseline_context
mandos.compare_runs
mandos.summarize_period
mandos.get_metric_trend
mandos.get_recurring_issues
mandos.triage_issues
mandos.detect_change_points
```

### Query Behavior

Mandos-AI should distinguish between:

```text
Point-to-point comparison
Example: March vs. April.

Period trend analysis
Example: last quarter, last year.

Change-point analysis
Example: when did degradation begin?

Recurring issue analysis
Example: what keeps failing?

Pre/post event comparison
Example: before and after model deployment, baseline refresh, or data pipeline change.
```

### Acceptance Criteria

Phase 7 is complete when Mandos-AI can answer:

```text
What changed over the last quarter?
```

With:

```text
- Number of runs analyzed
- Selected date window
- Health trend
- Major metric movements
- Recurring issues
- New issues
- Resolved issues
- Materiality assessment
- Recommended investigation path
```

---

## Phase 8 — MCP-Backed Controlled Tool Execution

### Objective

Allow GitHub Copilot to query Mandos evidence through controlled tools rather than relying only on generated code and documents.

This is where MCP becomes important.

### Why MCP

MCP lets Copilot connect to external tools and services. For Mandos, this means Copilot can eventually call a controlled Mandos MCP server that exposes approved tools for querying persisted monitoring evidence.

### Recommended First MCP Tools

Start read-only:

```text
mandos.validate_config
mandos.describe_baseline
mandos.list_model_runs
mandos.get_latest_run_summary
mandos.get_metric_trend
mandos.compare_runs
mandos.summarize_period
mandos.get_recurring_issues
mandos.triage_issues
```

### Explicitly Avoid in First MCP Release

```text
- No raw customer-level data access.
- No arbitrary SQL generation against production tables.
- No writes to MANDOS_* tables.
- No baseline creation.
- No official report generation without explicit user confirmation.
```

### Tool Safety Tiers

```text
Tier 1: Read-only
Safe by default.
Examples: validate config, list runs, describe baseline.

Tier 2: Analysis
Requires user awareness.
Examples: compare runs, summarize period, triage issues.

Tier 3: Write / official action
Requires explicit confirmation.
Examples: create baseline, persist run, generate official report, update registry.
```

### Acceptance Criteria

Phase 8 is complete when:

```text
- Copilot can call read-only Mandos tools.
- Tool outputs include source metadata.
- Mandos-AI can cite returned data as FACTS.
- Users can inspect what was queried.
- No write actions are available without explicit approval.
```

---

## Phase 9 — Multi-Agent Mandos-AI System

### Objective

Split the single Mandos agent into specialized agents only after real usage proves the need.

Do not start here.

### Future Agents

```text
mandos-analyst
Main entrypoint and router.

mandos-config-architect
Config, baseline, FeatureRules, segmentation.

mandos-report-builder
Custom reports, charts, evidence packs.

mandos-validation-reviewer
Model Validation readiness and evidence quality.

mandos-monitoring-analyst
Trend analysis from persisted Snowflake history.

mandos-engineer
Codebase implementation, tests, docs, refactoring.
```

### When To Split

Split only when:

```text
- Users frequently ask clearly different types of questions.
- The single agent instructions become too large.
- Tool access needs differ by role.
- Report generation and config review require different constraints.
- Engineering tasks should be separated from model-owner tasks.
```

### Acceptance Criteria

Phase 9 is complete when:

```text
- Each agent has a clear purpose.
- Each agent has limited tool access.
- Handoffs are documented.
- The user experience remains simple.
```

---

## 5. Suggested Sprint Plan

### Sprint 1 — Agent Foundation

Build:

```text
.github/copilot-instructions.md
.github/agents/mandos-analyst.agent.md
docs/agent_context/mandos_ai_doctrine.md
docs/agent_context/mandos_api_quickstart.md
docs/agent_context/mandos_config_schema.md
```

Outcome:

```text
Mandos-AI can explain Mandos concepts and answer basic setup questions.
```

---

### Sprint 2 — Setup Coach

Build:

```text
docs/agent_context/baseline_setup_guide.md
docs/agent_context/feature_rules_guide.md
docs/agent_context/segmentation_guide.md
examples/configs/generic_classification_config.yaml
examples/configs/generic_regression_config.yaml
examples/notebooks/01_mandos_setup_walkthrough.ipynb
```

Outcome:

```text
Mandos-AI can generate starter configs and setup notebooks.
```

---

### Sprint 3 — Zuul 2.0 Validation Companion

Build:

```text
docs/agent_context/zuul_2_validation_context.md
examples/configs/zuul_2_starter_config.yaml
examples/notebooks/zuul_2_monitor_report_walkthrough.ipynb
examples/scripts/run_zuul_2_mandos_report.py
```

Outcome:

```text
Zuul team can use Mandos-AI for pre-launch validation support.
```

---

### Sprint 4 — Report Explainer

Build:

```text
docs/agent_context/monitor_report_guide.md
examples/reports/report_explainer_prompt.md
examples/reports/model_validation_summary_template.md
```

Outcome:

```text
Mandos-AI can explain monitor().report() outputs and draft validation summaries.
```

---

### Sprint 5 — Alert Triage

Build:

```text
docs/agent_context/alert_triage_guide.md
examples/reports/alert_triage_template.md
examples/visualizations/issue_triage_charts.py
```

Outcome:

```text
Mandos-AI can distinguish action-required issues from watchlist and likely noise.
```

---

### Sprint 6 — Custom Reporting

Build:

```text
docs/agent_context/custom_reporting_guide.md
examples/reports/custom_report_template.md
examples/visualizations/basic_report_charts.py
examples/notebooks/custom_mandos_report_walkthrough.ipynb
```

Outcome:

```text
Mandos-AI can generate audience-specific report outlines, charts, and report scripts.
```

---

### Sprint 7 — Persistence Design

Build:

```text
docs/agent_context/persisted_monitoring_history_guide.md
docs/design/mandos_ai_mcp_tools.md
docs/design/mandos_semantic_views.md
```

Outcome:

```text
Mandos-AI is ready for future persisted-history analysis once production runs exist.
```

---

### Sprint 8 — MCP Proof of Concept

Build:

```text
mcp/
  mandos_mcp_server.py
  tools/
    validate_config.py
    get_latest_run_summary.py
    get_model_history.py
    compare_runs.py
    get_metric_trend.py
    triage_issues.py
```

Outcome:

```text
Copilot can call read-only Mandos tools against safe sample or development data.
```

---

## 6. Initial `mandos-analyst.agent.md` Sketch

```markdown
---
name: mandos-analyst
description: Helps users configure, run, interpret, and report with Mandos model monitoring workflows.
tools: ["read", "search", "edit"]
---

You are the Mandos Analyst agent.

You help data scientists use Mandos for:
- baseline setup
- FeatureRules
- segmentation
- profile / compare / monitor workflows
- monitor().report() PDFs
- validation evidence summaries
- custom reports and charts
- alert triage and noise reduction

Mandos principles:
- Mandos is evidence-first.
- No metric means no claim.
- No source means no fact.
- Facts must come from configs, code, reports, persisted Mandos tables, or user-provided evidence.
- You may reason and recommend, but you must label facts, interpretations, and recommendations clearly.
- Do not invent metric values, thresholds, baseline definitions, or model results.
- FeatureRules are hard data quality policies.
- Not every threshold breach is material.
- Record everything, escalate only what matters.
- Prefer config-driven workflows over hardcoded scripts.
- Prefer Snowflake pushdown for large data.
- Pull back only small summary outputs.

When helping a user set up Mandos:
1. Identify model type and usage.
2. Identify baseline and scored tables.
3. Identify entity key, timestamp, score, target, features, and segments.
4. Help draft FeatureRules.
5. Generate a starter config.
6. Generate a runnable notebook or script.
7. State assumptions and required user review.

When explaining reports:
- Use FACT / INTERPRETATION / RECOMMENDATION.
- Prioritize material findings.
- Separate action-required, watchlist, informational, and likely noise.
- Do not claim production trends unless persisted monitoring history exists.
```

---

## 7. Risks and Mitigations

### Risk 1 — Mandos-AI Hallucinates Monitoring Facts

Mitigation:

```text
No metric → no claim.
No source → no fact.
```

Require FACT / INTERPRETATION / RECOMMENDATION.

### Risk 2 — Users Overtrust Draft FeatureRules

Mitigation:

Label generated FeatureRules as:

```text
Draft rules requiring model-owner and validation review.
```

### Risk 3 — Alert Fatigue

Mitigation:

Build triage early. Do not wait until after broad rollout.

### Risk 4 — MCP Introduced Too Early

Mitigation:

Do not build MCP until the setup/report workflows are useful and there is persisted evidence worth querying.

### Risk 5 — Multi-Agent Complexity Too Early

Mitigation:

Start with one `mandos-analyst` agent. Split later only when usage patterns justify it.

### Risk 6 — Raw Data Access Concerns

Mitigation:

Mandos-AI should query Mandos evidence first:

```text
MANDOS_* tables
VW_* semantic views
Reports
Configs
```

Raw scored data should be explicitly out of scope for early phases.

---

## 8. Success Metrics

### Adoption Metrics

```text
- Number of users using Mandos-AI setup guidance
- Number of configs generated or reviewed
- Number of notebooks/scripts generated
- Reduction in repeated setup questions to Mandos core team
```

### Quality Metrics

```text
- Config review accuracy
- Reduction in missing required config fields
- Reduction in invalid FeatureRules
- Number of issues caught before validation review
```

### Reporting Metrics

```text
- Number of monitor().report() PDFs explained
- Number of validation summaries generated
- Number of custom reports generated
- User satisfaction with report clarity
```

### Alert Triage Metrics

```text
- Number of findings triaged
- Percentage classified as action / watchlist / informational / suppressed
- User agreement with triage classification
- Reduction in perceived alert noise
```

### Future Persistence Metrics

```text
- Number of historical trend questions answered
- Number of recurring issues detected
- Number of change-point analyses performed
- Number of model health narratives generated from persisted evidence
```

---

## 9. Final Recommended Roadmap

```text
Phase 0
Mandos-AI doctrine and repo instructions.

Phase 1
Single Mandos Analyst custom agent.

Phase 2
Setup coach for configs, baselines, FeatureRules, segments, and monitor().report().

Phase 3
Zuul 2.0 validation companion.

Phase 4
Reusable onboarding kit for future model teams.

Phase 5
Report explainer and validation evidence assistant.

Phase 6
Custom reporting and visualization builder.

Phase 7
Alert triage and noise reduction.

Phase 8
Persistence-aware historical monitoring analyst.

Phase 9
MCP-backed controlled tool execution.

Phase 10
Multi-agent Mandos-AI system.
```

---

## 10. Recommended First Release

The first release should be:

> **Mandos-AI v0.1: the Mandos Setup and Validation Companion.**

That version should be excellent at helping users understand Mandos, configure their model, run `monitor().report()`, explain the PDF, and prepare validation evidence. Once Zuul 2.0 launches and Mandos starts accumulating persisted runs, the same foundation can evolve into a Snowflake-backed historical monitoring analyst.

---

## Appendix A — Suggested Repository Structure

```text
.github/
  copilot-instructions.md
  agents/
    mandos-analyst.agent.md

docs/
  agent_context/
    mandos_ai_doctrine.md
    mandos_api_quickstart.md
    mandos_config_schema.md
    baseline_setup_guide.md
    feature_rules_guide.md
    segmentation_guide.md
    monitor_report_guide.md
    alert_triage_guide.md
    custom_reporting_guide.md
    persisted_monitoring_history_guide.md
    zuul_2_validation_context.md

  onboarding/
    model_owner_quickstart.md
    mandos_onboarding_checklist.md
    validation_readiness_checklist.md
    config_review_checklist.md
    feature_rules_examples.md
    segmentation_examples.md
    common_questions.md
    common_config_mistakes.md

  design/
    mandos_ai_mcp_tools.md
    mandos_semantic_views.md

examples/
  configs/
    generic_classification_config.yaml
    generic_regression_config.yaml
    zuul_2_starter_config.yaml

  notebooks/
    01_mandos_setup_walkthrough.ipynb
    zuul_2_monitor_report_walkthrough.ipynb
    custom_mandos_report_walkthrough.ipynb

  scripts/
    run_mandos_monitor.py
    generate_mandos_report.py
    run_zuul_2_mandos_report.py

  reports/
    report_explainer_prompt.md
    model_validation_summary_template.md
    alert_triage_template.md
    custom_report_template.md

  visualizations/
    basic_report_charts.py
    issue_triage_charts.py

mcp/
  mandos_mcp_server.py
  tools/
    validate_config.py
    get_latest_run_summary.py
    get_model_history.py
    compare_runs.py
    get_metric_trend.py
    triage_issues.py
```

---

## Appendix B — Core Mandos-AI Product Pillars

```text
1. Setup Coach
Help users configure baselines, FeatureRules, segments, and monitoring jobs.

2. EDA / Data Analyst
Help users explore Pandas and Snowflake data using Mandos workflows.

3. Report Explainer
Help users understand monitor().report() PDFs and prepare validation summaries.

4. Custom Reporting + Visualization Builder
Create audience-specific evidence reports, charts, and narratives.

5. Alert Triage Analyst
Weed out noise, reduce alert fatigue, and prioritize what matters.

6. Historical Monitoring Analyst
Query persisted Mandos runs and baselines for trend analysis once history exists.
```

---

## Appendix C — Mandos-AI Doctrine Summary

```text
Mandos persists the evidence.
Mandos-AI explains the evidence.

Facts are retrieved.
Interpretations are reasoned.
Recommendations are proposed.

No metric → no claim.
No source → no fact.
No persisted evidence → no production conclusion.
Not every breach → alert.
Persistent + material + worsening → prioritize.
One root cause → one story, not ten alerts.
Suppress noise transparently, never silently.
Record everything, escalate only what matters.
```
