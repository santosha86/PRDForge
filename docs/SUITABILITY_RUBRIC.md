# SUITABILITY_RUBRIC.md

The decision tree for `@multi-agent-suitability-checker` (Stage 0). This document is **the rubric the agent reads at runtime** — change this, and you change the moat.

---

## The single question

> **Is this problem actually a multi-agent problem?**

If the answer is no, PRDForge refuses to generate a multi-agent skill pack. This is the front-door moat. Most "AI builder" tools say yes to everything. PRDForge is the one that says no.

---

## Why this matters

Research (Google, Anthropic) shows multi-agent systems applied to non-multi-agent problems can degrade performance by **39–70%**. The wrong topology choice alone tanks task quality by up to 70%. A multi-agent skill pack built for a single-agent problem will be slower, more expensive, and produce worse output than a single well-prompted call.

So Stage 0 is not bureaucratic gatekeeping — it's the highest-leverage feature in the tool. Saying no protects the user from a 6-week mistake.

---

## The decision tree

### PROCEED if **all four** are true

```
✓ Naturally decomposable sub-tasks
    The problem can be sliced into discrete sub-problems with clear boundaries.
    Counter-example: "summarize this transcript" — one indivisible reasoning task.

✓ Each sub-task benefits from specialized reasoning
    Different sub-tasks need different prompts, different evaluation criteria,
    different domain framing. Generalist prompting would be worse.
    Counter-example: same kind of analysis applied to N items.

✓ Specialization OR parallelization offers real value
    Either the agents need to think differently (specialization), or they can
    work simultaneously to compress wall time (parallelization). At least one.
    Counter-example: tasks are sequential AND identical → no value either way.

✓ Coordination cost is acceptable
    The overhead of routing, aggregating, and handling partial failures is
    less than the value gained. For a 3-agent system this is almost always
    fine; for a 12-agent system, only when the value is large.
```

If all four hold → `verdict: PROCEED`, `reason_code: SUITABLE`.

### REJECT — and the reason matters

PRDForge rejects with one of five reason codes. **Each carries a different alternative recommendation.**

#### `SINGLE_AGENT_SUFFICIENT`

The problem is real, AI is the right tool, but **one well-designed prompt would outperform any multi-agent decomposition**.

Triggers:
- Problem is a single reasoning chain (extract, classify, summarize, transform)
- All logic operates on one input → one output with no branching specialization
- No independent subtask structure

Examples that fit this rejection:
- "Summarize this meeting transcript into action items"
- "Extract entities from this contract"
- "Translate this document into Arabic"
- "Score this essay on a rubric"

Alternative recommendation template:
> Write a single Claude skill file with a structured output prompt. Use a strong system message, a few-shot example block, and a Pydantic-shaped output schema. No multi-agent system needed.

#### `NOT_AI_PROBLEM`

The problem is deterministic and doesn't need an LLM at all.

Triggers:
- Pure CRUD operations
- Data pipelines (ETL, format conversion, joins)
- Algorithmic computation with a known closed-form solution
- Anything that should be a database query, a regex, or a script

Examples:
- "Sync these two databases on a schedule"
- "Convert CSV to Parquet"
- "Validate that every record has a non-null email field"

Alternative recommendation template:
> Use regular code (Python script, SQL, dbt model) or a workflow tool (n8n, Airflow, dbt). An LLM here would be slower, more expensive, less reliable, and less testable.

#### `WORKFLOW_AUTOMATION`

The problem is integration glue between systems — moving data, triggering actions — with no real reasoning.

Triggers:
- API-to-API connectors
- Trigger-action automation ("when X happens in Slack, post to Notion")
- Orchestration of deterministic steps
- "If this then that" rules

Examples:
- "When a new lead is added in HubSpot, send a welcome email and add to Mailchimp"
- "Forward GitHub issues from one repo to a Slack channel"

Alternative recommendation template:
> Use an integration platform — Zapier, Make, n8n, or a small custom service. LLMs add latency and non-determinism without solving anything that integration tools haven't already solved cleanly.

#### `SCOPE_TOO_VAGUE`

The PRD doesn't contain enough specificity to determine suitability. This is a **temporary** rejection — the user can refine and resubmit.

Triggers:
- Objective is one sentence with no detail
- No success criteria
- No mention of inputs or outputs
- The PRD reads like a vision statement, not a spec

Output requirement: **must include `clarifying_questions` (a list of 3–7 specific questions)**.

Alternative recommendation template:
> Refine the PRD. Specifically answer: [generated questions]. Resubmit when these are filled in.

This is the rejection most likely to convert into a future PROCEED. It's not a no — it's a "not yet, here's what's missing."

#### `SCALE_MISMATCH`

The problem is real but at the wrong scale for a multi-agent skill pack.

Triggers — too small:
- A single API call with structured output would cover it
- The PRD describes one transformation step, period

Triggers — too big:
- The PRD describes shipping a full SaaS product (multi-tenant, billing, UI, deployment)
- This is gstack territory, not PRDForge

Alternative recommendation template (too small):
> A single Claude API call with structured output is enough. No skill pack needed.

Alternative recommendation template (too big):
> Use [gstack](https://github.com/Y-Combinator/gstack) — it's purpose-built for full software sprints. PRDForge generates the multi-agent core; gstack ships the surrounding product.

---

## Confidence calibration

The agent emits a `confidence` score in `[0.0, 1.0]`.

| Confidence range | Behavior |
|---|---|
| `0.85 – 1.0` | High confidence. Verdict stands. |
| `0.6 – 0.85` | Acceptable. Verdict stands but the reasoning paragraph must explicitly note residual ambiguity. |
| `< 0.6` | **Force `verdict: REJECT` with `reason_code: SCOPE_TOO_VAGUE`** regardless of what the agent's reasoning suggested. Low confidence ⇒ the PRD is not specific enough to commit to a verdict. |

This rule is enforced in the schema validation (see `SCHEMAS.md` §1).

---

## Reasoning paragraph requirements

Every verdict must include a `reasoning` field that:

1. **Cites at least one specific phrase from the PRD** — direct evidence, not paraphrase. Use quotes.
2. **Names which of the four PROCEED criteria pass or fail** — be specific about which gate determined the outcome.
3. **Is one paragraph (3–6 sentences).** No bulleted lists, no headers. Prose.
4. **Avoids hedging.** "Probably," "might be," "could potentially" → these are signals the confidence should be lower, not the reasoning vaguer.

A verdict without PRD evidence in the reasoning is invalid and must be rejected by `@architecture-reviewer` if it slips through.

---

## Worked examples

### Example 1 — PROCEED

**PRD excerpt:** *"GhostCheck evaluates a CV across 11 independent dimensions: ATS compliance, Google presence, bucket fit, channel match, salary alignment, ... Each dimension is scored 0–100 with a defended rationale. Final output is a blind-panel verdict aggregating all 11 scores."*

**Verdict:**
```yaml
verdict: PROCEED
reason_code: SUITABLE
confidence: 0.92
reasoning: >
  The PRD describes "evaluating a CV across 11 independent dimensions" with
  "each dimension scored 0–100 with a defended rationale." This satisfies all
  four PROCEED criteria: dimensions are naturally decomposable, each requires
  domain-specialized reasoning (ATS scoring is unrelated to channel matching),
  parallelization compresses wall time substantially, and coordination cost is
  trivial since the aggregation is a simple weighted sum. Textbook blind-panel
  evaluation.
expected_multi_agent_benefit: >
  Parallel fan-out across 11 specialists; ~10x latency compression vs sequential
  single-agent loop, with measurably better per-dimension reasoning quality.
```

### Example 2 — REJECT (single-agent)

**PRD excerpt:** *"Summarize a meeting transcript into a list of action items, owners, and due dates."*

**Verdict:**
```yaml
verdict: REJECT
reason_code: SINGLE_AGENT_SUFFICIENT
confidence: 0.88
reasoning: >
  The PRD describes "summarize a meeting transcript into a list of action items,
  owners, and due dates" — one indivisible reasoning task with a well-defined
  input (transcript) and a well-defined output (structured list). The first
  PROCEED criterion fails: there are no naturally decomposable sub-tasks. A
  single Claude call with a structured output schema will outperform any
  multi-agent decomposition on both quality and cost.
alternative_recommendation: >
  Write a single Claude skill file with a Pydantic-shaped output schema
  (ActionItem with fields: description, owner, due_date). Use Sonnet 4.6 with
  a clear system prompt and 2-3 few-shot examples.
```

### Example 3 — REJECT (scope too vague)

**PRD excerpt:** *"We want to build an AI assistant for our team."*

**Verdict:**
```yaml
verdict: REJECT
reason_code: SCOPE_TOO_VAGUE
confidence: 0.45
reasoning: >
  The PRD describes "an AI assistant for our team" with no further detail. There
  is no statement of inputs, outputs, success criteria, or specific workflows
  the assistant would handle. Confidence in any verdict is below the 0.6
  threshold, which forces SCOPE_TOO_VAGUE per the rubric.
alternative_recommendation: >
  Refine the PRD with specific use cases, then resubmit.
clarifying_questions:
  - "What specific tasks should the assistant handle? List 3-5 concrete workflows."
  - "Who are the users — engineers, sales, support, leadership?"
  - "What inputs does it consume (Slack messages, tickets, docs, calendar)?"
  - "What does success look like — time saved, error reduction, satisfaction score?"
  - "What are the boundaries — what should it explicitly NOT do?"
```

---

## How to update this rubric

This rubric is part of PRDForge's moat. Changes affect every PROCEED/REJECT decision the system makes. When updating:

1. Add the new rule or example to the relevant section.
2. Re-run the Stage 0 agent against every example in `examples/` and confirm verdicts are stable (or, if intentionally changed, document the delta).
3. Bump the version footer below.
4. Note the change in PRD changelog if it affects observable behavior.

---

**Rubric version:** 1.0 (V3 release)
**Last reviewed:** April 2026
