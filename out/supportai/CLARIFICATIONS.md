---
prd_path: "./examples/ambiguous-prd.md"
generated_at: "2026-05-01T10:18:00Z"
auditor_verdict: "CLARIFICATIONS_RESOLVED"
entry_count: 6
test_artifact: true
test_artifact_note: "User answers below are reasonable test responses provided during a V1.0 smoke test (commit 32ae8cb). Not real product decisions; for bundle-format validation only."
---

# Clarifications — SupportAI

> **Audit log produced by `@clarification-auditor` (Stage 1).**
> The source PRD at `./examples/ambiguous-prd.md` was not modified. This document is the additive record of how ambiguities, contradictions, undefined terms, and gaps were resolved during a PRDForge run.
>
> **Auditor verdict:** `CLARIFICATIONS_RESOLVED`
> **Entries:** 6

---

## C-01 — CONTRADICTION on R-06,R-07

| Field | Value |
|---|---|
| **Raised** | `2026-05-01T10:18:00Z` |
| **Raised by** | `@clarification-auditor` |
| **Relates to** | `R-06,R-07` |
| **Ambiguity type** | `CONTRADICTION` |

**Question:**

R-06 mandates that "all ticket data must remain on-device for privacy reasons" but R-07 requires the urgency-classification step to "use OpenAI GPT-4o," which is an off-device API. These cannot both hold for any ticket payload that includes urgency-relevant content. Which constraint takes priority?

- **Option A:** Drop the OpenAI dependency. Use an on-device model (e.g. local Anthropic-compatible inference via BYOK) so privacy holds.
- **Option B:** Relax R-06 to "BYOK API only — payload may transit a customer-controlled API key but must not be persisted by the provider," and accept the OpenAI dependency.
- **Option C:** Split the payload: only de-identified urgency signals (no PII) leave the device.

**Answer:** Option A — drop OpenAI. Use a BYOK Anthropic Claude API call for the urgency-classification step. Privacy is non-negotiable; the model swap is acceptable.

**Resolution:** `DISAMBIGUATION`

---

## C-02 — UNDEFINED_TERM on R-04

| Field | Value |
|---|---|
| **Raised** | `2026-05-01T10:18:00Z` |
| **Raised by** | `@clarification-auditor` |
| **Relates to** | `R-04` |
| **Ambiguity type** | `UNDEFINED_TERM` |

**Question:**

R-04 references "premium users" and "priority routing" but neither term is defined in the PRD. What distinguishes a premium user, and what does "priority routing" mean operationally?

**Answer:** A premium user is any customer whose account record carries `tier ∈ {gold, platinum}` in the customer database. "Priority routing" means the ticket is inserted at the head of the destination team's queue, ahead of any non-premium tickets currently waiting.

**Resolution:** `DISAMBIGUATION`

---

## C-03 — MISSING_DETAIL on R-09

| Field | Value |
|---|---|
| **Raised** | `2026-05-01T10:18:00Z` |
| **Raised by** | `@clarification-auditor` |
| **Relates to** | `R-09` |
| **Ambiguity type** | `MISSING_DETAIL` |

**Question:**

R-09 states "the system should be fast" without a measurable target. What is the acceptable end-to-end triage latency?

- **Option A:** p95 < 2 seconds (interactive feel)
- **Option B:** p95 < 5 seconds (chat-like)
- **Option C:** p95 < 30 seconds (background processing)

**Answer:** Option B — p95 end-to-end triage latency under 5 seconds, measured from ticket arrival to team-routing decision.

**Resolution:** `DISAMBIGUATION`

---

## C-04 — MISSING_DETAIL on R-11,R-12

| Field | Value |
|---|---|
| **Raised** | `2026-05-01T10:18:00Z` |
| **Raised by** | `@clarification-auditor` |
| **Relates to** | `R-11,R-12` |
| **Ambiguity type** | `MISSING_DETAIL` |

**Question:**

The success criteria "faster mean time to first response" and "fewer mis-routed tickets" lack baselines and target thresholds. The third success criterion ("customer satisfaction improves") is too vague to be a system requirement at all. What measurable targets should be set?

**Answer:**
- **MTFR:** target ≤ 1h median, baseline 4h.
- **Mis-routing rate:** target ≤ 3%, baseline 12%.
- **CSAT:** out of scope for SupportAI itself; tracked at program-level dashboard, not as a SupportAI requirement. Drop the third success criterion.

**Resolution:** `DISAMBIGUATION`

---

## C-05 — GAP on R-02

| Field | Value |
|---|---|
| **Raised** | `2026-05-01T10:18:00Z` |
| **Raised by** | `@clarification-auditor` |
| **Relates to** | `R-02` |
| **Ambiguity type** | `GAP` |

**Question:**

R-02 defines the success path for ticket classification but does not specify behavior on classifier failure or low confidence. Options:

- **Option A:** Route to a default catch-all team (the `escalation` category).
- **Option B:** Hold for human review; do not auto-route.
- **Option C:** Reject with an error to the submitter.

**Answer:** Option B — failed or low-confidence classifications go to a human-review queue, and the system emits a `classifier_failures_total` metric. The user does not see an error.

**Resolution:** `NEW_REQUIREMENT`

**Registered as:** `R-10` — `@requirement-analyzer` was re-run to incorporate this requirement formally. See `REQUIREMENTS.md`.

---

## C-06 — GAP on R-02

| Field | Value |
|---|---|
| **Raised** | `2026-05-01T10:18:00Z` |
| **Raised by** | `@clarification-auditor` |
| **Relates to** | `R-02` |
| **Ambiguity type** | `GAP` |

**Question:**

R-02 lists six valid intent categories. What happens to tickets that don't fit any of them?

**Answer:** Map them to the `escalation` category. (This is distinct from C-05's case: C-05 handles classifier *failure*; C-06 handles successful classifications that don't match any specific category.)

**Resolution:** `DISAMBIGUATION`

---

## Summary

| Resolution | Count |
|---|---|
| `DISAMBIGUATION` | 5 |
| `EXTENSION` | 0 |
| `NEW_REQUIREMENT` | 1 |
| Skipped | 0 |

All clarifications resolved. One new requirement (`R-10`) registered. The validated baseline is now `(PRD ∪ CLARIFICATIONS)` and is ready for downstream execution. Stage 5 will compare delivery coverage against this baseline.

---

*Generated by PRDForge — see [`07_PRDForge_PRD_V4.md`](../../07_PRDForge_PRD_V4.md) for the spec.*
