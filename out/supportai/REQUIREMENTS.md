---
prd_path: "./examples/ambiguous-prd.md"
generated_at: "2026-05-01T10:15:00Z"
analyzer: "@requirement-analyzer"
requirement_count: 12
ids: [R-01, R-02, R-03, R-04, R-05, R-06, R-07, R-08, R-09, R-10, R-11, R-12]
---

# Requirements — SupportAI

> **Authoritative numbered requirement list produced by `@requirement-analyzer` (Stage 1).**
> IDs are stable for the duration of this PRDForge run and are referenced by Stage 5's traceability gate. Source PRD: `./examples/ambiguous-prd.md` (immutable).

---

## Summary

| Priority | Count |
|---|---|
| MUST  | 12 |
| SHOULD | 0 |
| COULD | 0 |

> ⚠️ **5 requirements have acceptance criteria flagged as "pending clarification."** The auditor (Stage 1) raised matching ambiguities — see `CLARIFICATIONS.md`.

---

## R-01 — Accept multi-channel ticket input

| Field | Value |
|---|---|
| **Priority** | `MUST` |
| **Source** | `3. Functional Requirements` |

**Description:**

Accept incoming customer support tickets from email, chat, and the in-app help widget.

**Acceptance criteria:**

- [ ] The system accepts a ticket payload originating from an email integration.
- [ ] The system accepts a ticket payload originating from a chat integration.
- [ ] The system accepts a ticket payload originating from the in-app help widget.

---

## R-02 — Classify ticket intent

| Field | Value |
|---|---|
| **Priority** | `MUST` |
| **Source** | `3. Functional Requirements` |

**Description:**

Classify each ticket into one of six intent categories: `bug`, `billing`, `account`, `feature_request`, `how_to`, `escalation`.

**Acceptance criteria:**

- [ ] Every classified ticket is assigned exactly one of the six listed categories.
- [ ] Acceptance criteria pending clarification: behavior on tickets that fit none of the six categories is not specified in the PRD (see `C-06` in `CLARIFICATIONS.md`).

---

## R-03 — Assess ticket urgency

| Field | Value |
|---|---|
| **Priority** | `MUST` |
| **Source** | `3. Functional Requirements` |

**Description:**

Assess each ticket's urgency on a 0–3 scale where 0 = low and 3 = critical.

**Acceptance criteria:**

- [ ] Every ticket receives an integer urgency score in the inclusive range [0, 3].

---

## R-04 — Identify premium users; route premium tickets with priority

| Field | Value |
|---|---|
| **Priority** | `MUST` |
| **Source** | `3. Functional Requirements` |

**Description:**

Identify whether the customer who opened the ticket is a premium user, and route premium tickets with priority over non-premium tickets.

**Acceptance criteria:**

- [ ] Acceptance criteria pending clarification: the term "premium user" is not defined in the PRD (see `C-02` in `CLARIFICATIONS.md`). The operational meaning of "priority routing" is also undefined.

---

## R-05 — Recommend response template

| Field | Value |
|---|---|
| **Priority** | `MUST` |
| **Source** | `3. Functional Requirements` |

**Description:**

Recommend a response template for each ticket, drawn from the team's existing template library.

**Acceptance criteria:**

- [ ] Every ticket is paired with a single recommended template ID from the existing library.
- [ ] If no template matches, the recommendation field is null (no fabricated template).

---

## R-06 — On-device data residency

| Field | Value |
|---|---|
| **Priority** | `MUST` |
| **Source** | `3. Functional Requirements` |

**Description:**

All ticket data must remain on the user's device for privacy reasons.

**Acceptance criteria:**

- [ ] No ticket payload is transmitted to a third-party service in a default-config run.
- [ ] Acceptance criteria pending clarification: this requirement directly conflicts with R-07 (see `C-01` in `CLARIFICATIONS.md`).

---

## R-07 — Use OpenAI GPT-4o for urgency classification

| Field | Value |
|---|---|
| **Priority** | `MUST` |
| **Source** | `3. Functional Requirements` |

**Description:**

Use OpenAI's GPT-4o model for the urgency-classification step (per the PRD: "fastest model available").

**Acceptance criteria:**

- [ ] Acceptance criteria pending clarification: this requirement directly conflicts with R-06 (see `C-01` in `CLARIFICATIONS.md`).

---

## R-08 — Route ticket to correct team

| Field | Value |
|---|---|
| **Priority** | `MUST` |
| **Source** | `3. Functional Requirements` |

**Description:**

Route each ticket to the correct downstream team based on its classification.

**Acceptance criteria:**

- [ ] Every classified ticket is delivered to a single named team queue.
- [ ] Routing follows a deterministic intent-to-team mapping (mapping not specified in the PRD; assumed to be config-driven).

---

## R-09 — Bounded latency

| Field | Value |
|---|---|
| **Priority** | `MUST` |
| **Source** | `3. Functional Requirements` |

**Description:**

The system must be fast.

**Acceptance criteria:**

- [ ] Acceptance criteria pending clarification: "fast" is specified without a measurable target (see `C-03` in `CLARIFICATIONS.md`).

---

## R-10 — Failed-classification fallback path

| Field | Value |
|---|---|
| **Priority** | `MUST` |
| **Source** | `CLARIFICATION:C-05` |

**Description:**

Classifications that fail or fall below confidence threshold route to a human-review queue and emit a metric.

**Acceptance criteria:**

- [ ] Low-confidence or failed classifications are sent to a human-review queue.
- [ ] Each failure increments a `classifier_failures_total` counter metric.
- [ ] User-facing behavior on failure does not surface an internal error to the customer.

---

## R-11 — Faster mean time to first response

| Field | Value |
|---|---|
| **Priority** | `MUST` |
| **Source** | `5. Success Criteria` |

**Description:**

Mean time to first response should improve relative to the pre-SupportAI baseline.

**Acceptance criteria:**

- [ ] Acceptance criteria pending clarification: the PRD specifies "faster" without a baseline or target (see `C-04` in `CLARIFICATIONS.md`).

---

## R-12 — Reduced mis-routing rate

| Field | Value |
|---|---|
| **Priority** | `MUST` |
| **Source** | `5. Success Criteria` |

**Description:**

Fewer tickets are routed to the wrong team relative to the pre-SupportAI baseline.

**Acceptance criteria:**

- [ ] Acceptance criteria pending clarification: the PRD specifies "fewer mis-routed tickets" without a baseline or target rate (see `C-04` in `CLARIFICATIONS.md`).

---

## Provenance notes

- `R-10` was registered after the user resolved a `NEW_REQUIREMENT` clarification (`C-05`). See `CLARIFICATIONS.md` for the original ambiguity.
- `R-11` and `R-12` derive from the PRD's qualitative success criteria. They are listed as requirements per the analyzer's policy ("each measurable success criterion becomes a requirement"); their acceptance criteria are flagged pending the auditor's `MISSING_DETAIL` clarification (`C-04`).
- The success criterion "customer satisfaction improves" was not extracted as a separate `R-*` because it is too high-level to map to a single requirement; it would belong in a separate program-level dashboard, not the SupportAI system spec.

---

*Generated by PRDForge — see [`07_PRDForge_PRD_V4.md`](../../07_PRDForge_PRD_V4.md) for the spec.*
