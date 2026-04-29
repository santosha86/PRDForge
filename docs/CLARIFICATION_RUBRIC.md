# CLARIFICATION_RUBRIC.md

The rubric `@clarification-auditor` (Stage 1) follows when scanning parsed requirements for ambiguity, contradictions, undefined terms, and gaps.

This is **the second moat** — paired with the Stage 0 Suitability Gate. Stage 0 stops you from building the wrong *kind* of system; the clarification audit stops you from building the right kind that solves the wrong *problem*.

---

## The mandate

> **Detect what is missing, contradictory, or undefined — never modify the source PRD.**

The auditor produces `CLARIFICATIONS.md`, an additive audit log alongside the immutable PRD.

---

## When to raise a clarification

Raise a clarification when **any** of these is true for a requirement, the objective, or success criteria:

### `MISSING_DETAIL`
A measurable was specified without a target.
- "fast" with no latency budget
- "accurate" with no precision threshold
- "scale" with no volume target
- "secure" with no threat model

### `CONTRADICTION`
Two requirements assert incompatible things.
- R-03 says "must work offline" and R-07 says "queries the OpenAI API"
- R-12 caps p95 at 200ms and R-15 mandates a synchronous database write

### `UNDEFINED_TERM`
A domain term is used without definition where the meaning is non-obvious.
- "high-trust user" — who qualifies?
- "compliant output" — compliant with what?
- "core workflow" — which one?

### `GAP`
A behavior is implied but not specified.
- The happy path is clear; error/edge behavior isn't.
- Inputs are listed but the format isn't named.
- A persona is mentioned but their entry point isn't.
- Success is defined but failure isn't.

---

## When NOT to raise a clarification

Do not raise on:
- **Stylistic ambiguity** — "the system should be elegant" is fine to leave alone.
- **Author preference** — naming, ordering, formatting choices that don't change behavior.
- **Things already specified elsewhere in the PRD** — read the whole document first.
- **Standard industry conventions** — "REST API" doesn't need a definition; "REST-ish" does.
- **Implementation details below the architecture line** — Stage 1 specifies *what*, not *how*.

The bar: would a senior engineer reading the PRD have to make a judgment call to proceed? If yes → raise. If no → don't.

---

## Interaction model

The auditor surfaces clarifications **in one batch**, not interactively per-requirement. The user answers all of them in a single review pass. This preserves the "validate in a day" promise while still capturing every ambiguity.

If the auditor finds **zero** material ambiguities, it emits:

```yaml
auditor_verdict: NO_CLARIFICATIONS_NEEDED
entries: []
```

and the pipeline proceeds directly. Tight PRDs pay no tax.

---

## Resolution kinds

When the user answers a clarification, the auditor records the resolution kind:

| Kind | When |
|---|---|
| `DISAMBIGUATION` | The answer clarifies an existing requirement. Most common. |
| `EXTENSION` | The answer expands the scope of an existing requirement (more cases, larger surface). |
| `NEW_REQUIREMENT` | The answer surfaces a previously unstated requirement. Auditor must register it as a new `R-*` ID and the analyzer is re-run for consistency. |

`NEW_REQUIREMENT` resolutions are the most consequential — they grow the requirement set. The traceability gate at Stage 5 must cover them like any other requirement.

---

## Skipping clarifications

If the user declines to answer a specific clarification:

```yaml
user_answer: null
user_skipped: true
resolution_kind: null
```

The auditor records the skip but does not block the pipeline. Stage 5 traceability gate flags skipped clarifications as **risk items** in the final report — they are not failures, but the user is told explicitly that those ambiguities flowed through unresolved.

---

## Output structure

See `SCHEMAS.md` §4 — `ClarificationLog`.

The on-disk format is `CLARIFICATIONS.md` with a YAML frontmatter header followed by markdown rendering of each entry, in chronological order. This makes the file both machine-readable (frontmatter + structured entries) and human-readable.

---

## Worked examples

### Example 1 — `MISSING_DETAIL`

**Requirement R-04:** *"The system must respond quickly to user queries."*

**Clarification:**
```yaml
clarification_id: C-01
relates_to: R-04
ambiguity_type: MISSING_DETAIL
question: >
  R-04 specifies "respond quickly" without a measurable target. What is the
  acceptable latency budget? Common targets:
  • p95 < 200ms (interactive UI)
  • p95 < 2s (chat-like)
  • p95 < 30s (background processing)
  Which fits this product?
```

### Example 2 — `CONTRADICTION`

**Requirement R-03:** *"All processing must occur on-device for privacy reasons."*
**Requirement R-08:** *"The summarization step uses GPT-4 via the OpenAI API."*

**Clarification:**
```yaml
clarification_id: C-02
relates_to: "R-03,R-08"
ambiguity_type: CONTRADICTION
question: >
  R-03 mandates on-device processing for privacy, but R-08 specifies the
  summarization step calls the OpenAI API (off-device). These cannot both
  hold. Which constraint takes priority — privacy (drop OpenAI dependency)
  or capability (relax on-device requirement to "BYOK API only")?
```

### Example 3 — `UNDEFINED_TERM`

**Requirement R-09:** *"Premium users get priority routing."*

**Clarification:**
```yaml
clarification_id: C-03
relates_to: R-09
ambiguity_type: UNDEFINED_TERM
question: >
  R-09 references "premium users" but no requirement or section defines
  premium tier criteria. What distinguishes a premium user — paid plan,
  account age, usage volume, manual flag? And what does "priority routing"
  mean operationally — preferred queue, faster model tier, both?
```

### Example 4 — `GAP`

**Requirement R-11:** *"On successful classification, route the ticket to the appropriate team."*

**Clarification:**
```yaml
clarification_id: C-04
relates_to: R-11
ambiguity_type: GAP
question: >
  R-11 describes the success path but doesn't specify behavior on classification
  failure. What happens when the classifier returns low confidence or no match?
  Options:
  • Route to a default catch-all team
  • Hold for human review
  • Reject with an error to the submitter
  Pick one (or describe the failure-handling policy).
```

---

## Calibration targets

The auditor must hit these on the golden-set examples:

| Metric | Target |
|---|---|
| Recall — % of senior-reviewer-flagged ambiguities the auditor surfaces | ≥ 80% |
| False-positive rate — % of auditor flags a senior reviewer would dismiss | ≤ 15% |
| `NO_CLARIFICATIONS_NEEDED` accuracy on the tight-PRD golden examples | 100% |

If the auditor over-asks, tighten the "When NOT to raise" rules. If it under-asks, expand the "When to raise" rules with the missed cases.

---

**Rubric version:** 1.0 (V3 release)
**Last reviewed:** April 2026
