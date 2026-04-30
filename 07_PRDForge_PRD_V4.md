# Product Requirements Document (PRD) — V4
# PRDForge — Validation + Governance Pre-Flight for Spec-Driven AI Development

| Field            | Detail                                                     |
|------------------|------------------------------------------------------------|
| **Document ID**  | PF-PRD-004                                                 |
| **Version**      | 4.0 — Repositioning to governance pre-flight               |
| **Date**         | April 2026                                                 |
| **Author**       | Santosh Achanta                                            |
| **Status**       | Draft — Pre-build (V0.2 already shipped under V3 framing)  |
| **Repo**         | github.com/santosha86/PRDForge                             |
| **License**      | MIT                                                        |
| **Supersedes**   | PF-PRD-003 (V3)                                            |

> **Note on prior versions:** V2 and V3 are preserved untouched in this repository as the audit history of the product's evolution — the same immutability discipline V4 codifies for its users. This is V4 because the product's positioning and scope have changed, not because the moats changed; the two moats (Suitability Gate + Audit Trail) carry forward unchanged.

---

## Changelog V3 → V4

| Change | Why |
|---|---|
| Repositioned product from "PRD-to-skill-pack generator" to **"validation + governance pre-flight layer"** | A landscape scan showed many mature tools already produce running code or generated artifacts from specs. The "generate a skill pack" output was a vanity choice. The defensible lane is the *pre-flight* — refusing unsuitable problems and producing audit-grade artifacts — before any execution tool runs. |
| New primary user: **regulated-industry teams under emerging AI governance regimes** | The EU AI Act's high-risk-AI provisions take effect 2026-08-02. Audit-grade trails on AI-generated systems become a procurement requirement, not a nicety. PRDForge's outputs (immutable spec + clarifications + traceability matrix) are exactly that artifact. |
| V1 scope narrowed to Stages 0–1 + lightweight Stage 5 traceability gate | Smaller, sharper, shippable. Stages 2–4 (architecture / design / generation) become V2 "advisory architecture brief" features. |
| Output reframed as **a structured artifact bundle**, not a runnable skill pack | The bundle is: validated PRD + `CLARIFICATIONS.md` + `TRACEABILITY.md` + (V2) advisory architecture brief. The user takes this bundle to whatever execution tool they prefer. PRDForge is the governance layer; execution is downstream. |
| All comparative positioning against named tools removed | The product stands on its own merits. Comparative analysis is internal strategy material, not user-facing. |
| Added §15 — alignment with emerging AI governance frameworks | Names the specific compliance pressures (EU AI Act, NIST AI RMF, ISO 42001) that PRDForge's outputs help satisfy. |

The Suitability Gate (Stage 0), the Clarification Auditor (Stage 1), the Requirement Traceability Gate (Stage 5), and the topology-selection discipline are unchanged. Their *framing* is now "governance artifacts" rather than "generator stages."

---

## 1. Executive Summary

### 1.1 The Problem

Two failure modes dominate AI-driven software projects in 2026:

1. **Building the wrong kind of system.** Teams default to multi-agent architectures for problems that should be a single well-prompted call, a script, or a workflow tool. The wrong topology choice has documented performance penalties of up to 70%.

2. **Building the right kind of system that solves the wrong problem.** When the spec is ambiguous, an AI-driven build silently picks an interpretation. By delivery, nobody can prove what was asked for, what was built, or where the two diverged. In regulated contexts, this is no longer just a quality issue — it's a compliance failure.

Existing AI-development tools are optimized for *production speed*: they take a spec and produce code as fast as possible. Almost none ask "should this be built at all?" or "is the spec specific enough to build against?" before they start. That's the gap PRDForge fills.

### 1.2 What PRDForge Is

**PRDForge is a runtime governance pre-flight that runs before any spec-driven AI development pipeline.**

It reads a Product Requirements Document and produces three artifacts:

1. **A Suitability Verdict** — proceed-or-refuse decision with reason code and defended rationale.
2. **An Immutable-Spec + Clarifications Bundle** — original PRD untouched, plus an additive `CLARIFICATIONS.md` log of every ambiguity surfaced and how it was resolved.
3. **A Requirement Traceability Matrix** — numbered requirements with acceptance criteria, ready to feed downstream verification.

PRDForge stops there. The user takes the artifact bundle to whatever execution platform they prefer. The defensibility comes from doing the *governance layer* better than anyone else, not from competing on code generation.

### 1.3 The Two Moats

These are non-negotiable. They are PRDForge's product, not features within it.

#### Moat 1 — Runtime Suitability Refusal (Stage 0)

Most decision frameworks for "should this be multi-agent?" are advisory: read a checklist and think about it. PRDForge's Stage 0 is a *runtime gate*. If the problem is not genuinely a multi-agent problem, PRDForge refuses to proceed and returns one of five reason codes (`SINGLE_AGENT_SUFFICIENT`, `NOT_AI_PROBLEM`, `WORKFLOW_AUTOMATION`, `SCOPE_TOO_VAGUE`, `SCALE_MISMATCH`) with an alternative recommendation.

This is the discipline of **"single-agent until proven multi-agent"** encoded as enforceable behavior.

#### Moat 2 — Audit-Grade Traceability (Stage 1 + Stage 5)

The source PRD is immutable. Clarifications produced by the Stage 1 auditor go to a separate, append-only log (`CLARIFICATIONS.md`). At delivery, the Stage 5 traceability gate validates that every numbered requirement in `(PRD ∪ CLARIFICATIONS)` has explicit coverage in the downstream output, and emits a coverage matrix.

This is the discipline of **"don't build until you know what you're building, and prove what you built against what you asked for"** encoded as a fail-closed gate.

### 1.4 Why Now

Three convergent forces:

1. **Regulatory pressure.** The EU AI Act's high-risk AI system provisions take effect 2026-08-02. They require demonstrable traceability between requirements and AI-generated outputs. NIST AI RMF and ISO 42001 push in the same direction. Buyers in medical, finance, government, automotive, and other regulated verticals will need audit-grade trails as a procurement criterion. Few existing tools produce that artifact natively.

2. **Spec-driven development is mainstream.** The shift from "vibe coding" to writing specs first has happened. Teams now treat specs as the primary artifact. But the existing tooling assumes specs are *correct and complete* — which they rarely are without scrutiny. PRDForge fills the validation step that's missing.

3. **Multi-agent over-engineering is documented.** Public research quantifies the performance cost of misapplied multi-agent designs (39–70% degradation). Senior engineers want a tool that enforces restraint. There is no good answer in the market today; the closest things are advisory blog posts.

### 1.5 The Killer Feature

> **"Validate the spec, refuse the unsuitable, prove what you built — before a single line of code is generated."**

You give PRDForge a PRD on Monday morning. By midday you have a verdict, a clarifications log, and a traceability-ready requirement matrix. If PRDForge said `PROCEED`, you take the bundle to your execution tool. If PRDForge said `REJECT`, you saved your team weeks. Either way, you have an audit trail you can show a regulator.

---

## 2. Target Users

| Persona                                      | Pain                                                                             | Why PRDForge Helps                                                                       |
|----------------------------------------------|----------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|
| **Regulated-industry AI lead** (primary)     | Needs auditable spec→delivery traceability for AI-generated systems by 2026-08-02 | Produces the immutable-spec + clarifications + traceability triple as a coherent bundle  |
| **Solo multi-agent builder**                 | Spends weeks building before knowing if the architecture is sound                | Stage 0 verdict in seconds; clarifications batch in minutes                              |
| **ML/AI engineer evaluating a new idea**     | Unsure if a problem warrants a multi-agent system at all                         | Runtime refusal with defended reason code                                                |
| **Architecture reviewer at scale**           | Junior engineers ship over-engineered LangGraph DAGs that should have been one prompt | Use PRDForge as a CI step on the PRD before any build is approved                         |
| **Internal platform team** (governance)      | Need a uniform pre-flight gate across many product teams' AI initiatives         | Drop-in, language-agnostic, output is plain markdown + JSON                              |
| **Procurement / vendor-evaluation team**     | Need to assess whether a vendor's AI proposal is sound before contract           | Run the vendor's spec through Stage 0 + 1; the verdict is part of the evaluation memo    |

Explicit non-users — PRDForge is **not** for:

- Teams that want their tool to *generate code*. PRDForge stops at the validated artifact bundle.
- Teams whose specs are tight, whose architectures are settled, and who just want to ship. Use a code-generation tool directly.
- Problems that aren't AI problems at all. PRDForge will tell you so and refuse to proceed.

---

## 3. Scope

### 3.1 V1 — Validation Pre-Flight (already partially shipped)

| ID    | Feature                                  | Status (April 2026) |
|-------|------------------------------------------|---------------------|
| V1-01 | PRD parser → ProductSpec                 | ✅ Shipped (V0.2)   |
| V1-02 | **Suitability Gate (Stage 0)**           | ✅ Shipped (V0.1)   |
| V1-03 | Requirement analyzer → R-01…R-NN         | ✅ Shipped (V0.2)   |
| V1-04 | **Clarification Auditor → `CLARIFICATIONS.md`** | ✅ Shipped (V0.2)   |
| V1-05 | Domain researcher (web-search enrichment)| ✅ Shipped (V0.2)   |
| V1-06 | **Stage 5 Traceability Gate** (lightweight) | ⏳ V0.3              |
| V1-07 | CLI / router skill                       | ⏳ V0.3              |
| V1-08 | Output bundle to `out/<slug>/` (PRD copy + clarifications + traceability) | ⏳ V0.3 |

**V1 ships when V0.3 lands.** Output of V1 is the validated artifact bundle, not generated code.

### 3.2 V2 — Advisory Architecture Brief (Q3 2026)

| ID    | Feature                                  |
|-------|------------------------------------------|
| V2-01 | Architecture planner — packaging, scale model, memory strategy (advisory) |
| V2-02 | Topology selector (advisory) — picks parallel / sequential / hierarchical / hybrid with defended rationale |
| V2-03 | Agent roster proposal — recommends 5–11 agents with mandates and tiers |
| V2-04 | Schema designer — proposes the agent output contract |
| V2-05 | Architecture brief markdown — the consolidated advisory document |

**V2 output is a *report*, not a build.** It tells you what to build and why. Execution is still downstream.

### 3.3 V3 — (deferred) Skill-pack reference implementation

If V1 + V2 land well and there is demand, V3 can produce a reference Claude Code skill pack alongside the architecture brief. This is explicitly *not* a V1/V2 feature. The product proposition does not depend on it.

### 3.4 Explicitly Out of Scope (Forever)

| Item | Rationale |
|---|---|
| Shipping the generated code | PRDForge is governance, not execution |
| QA / browser testing of generated systems | Downstream tools' job |
| Multi-tenant hosting / SaaS | Local-first, BYOK |
| Auto-deployment | User deploys whatever they build using PRDForge's bundle as input |
| Estimating product timelines | Not the tool's mandate |
| Recommending specific frameworks | Output is framework-agnostic |
| Generating skill packs for problems Stage 0 rejected | Refusal is a feature; it does not have a workaround flag |

### 3.5 What PRDForge Will Not Do (Anti-Features)

To prevent scope creep, the following are explicit non-goals:

- Will not silently rewrite the source PRD. Ever. All interpretations go to `CLARIFICATIONS.md`.
- Will not produce a suitability verdict without PRD-grounded evidence (cited quote required in reasoning).
- Will not pass Stage 5 if any requirement lacks coverage in the downstream output.
- Will not infer requirements that the PRD does not state.
- Will not optimize for code generation speed (not its lane).

---

## 4. Architecture

PRDForge is itself a hybrid multi-agent system that demonstrates the disciplines it enforces.

### 4.1 High-Level Flow (V1)

```
PRD.md input
     │
     ▼
┌────────────────────────────────────────┐
│ STAGE 0 — Suitability Gate             │
│   @multi-agent-suitability-checker     │ ← MOAT 1: Runtime refusal
│   Output: SuitabilityVerdict           │
└─────────────────┬──────────────────────┘
          REJECT  │  PROCEED
            │     │
            ▼     ▼
     Explain   STAGE 1 — Understanding (Sequential)
     & exit   ┌────────────────────────────────────┐
              │  @prd-parser                        │
              │  @requirement-analyzer              │ → R-01…R-NN
              │  @clarification-auditor             │ ← MOAT 2: Audit log
              │     ↓                                │
              │  CLARIFICATIONS.md                   │
              │  @domain-researcher                  │
              └─────────────────┬───────────────────┘
                                ▼
              STAGE 5 — Traceability Gate (lightweight)
              ┌────────────────────────────────────┐
              │  Cross-check every R-* in           │
              │  (PRD ∪ CLARIFICATIONS) has         │
              │  acceptance criteria, source, and   │
              │  resolved ambiguities. Fails closed │
              │  on unresolved gaps.                │
              └─────────────────┬───────────────────┘
                                ▼
                   ✅ Validated artifact bundle at ./out/<slug>/
                   ├─ PRD.md (copy of source, immutable)
                   ├─ CLARIFICATIONS.md
                   ├─ REQUIREMENTS.md (R-01…R-NN)
                   └─ TRACEABILITY.md (matrix scaffold)
```

**Stages 2–4 are deferred to V2.** The full advisory architecture brief is a follow-on, not a blocker for V1.

### 4.2 The Suitability Gate (Moat 1) — Unchanged from V3

A multi-agent problem must satisfy **all four** criteria:

- Naturally decomposable sub-tasks
- Each sub-task benefits from specialized reasoning
- Specialization OR parallelization offers real value
- Coordination cost is acceptable

If any fails, refuse with a reason code:

| Reason code | When |
|---|---|
| `SINGLE_AGENT_SUFFICIENT` | One well-prompted call beats any decomposition |
| `NOT_AI_PROBLEM` | Deterministic / data-pipeline / closed-form |
| `WORKFLOW_AUTOMATION` | Integration glue with no real reasoning step |
| `SCOPE_TOO_VAGUE` | PRD is too thin to commit (returns clarifying questions) |
| `SCALE_MISMATCH` | Too small (one API call) or too big (full SaaS) |

Full rubric in [`docs/SUITABILITY_RUBRIC.md`](./docs/SUITABILITY_RUBRIC.md).

### 4.3 The Audit Trail (Moat 2) — Unchanged from V3

Three artifacts, three roles:

| Document | Status | Mutability |
|---|---|---|
| `PRD.md` (input) | Source intent | **Immutable** — never modified |
| `CLARIFICATIONS.md` (generated) | Audit log of resolved ambiguities | Append-only during Stage 1 |
| `TRACEABILITY.md` (generated at delivery) | Coverage matrix | Replaceable per run |

Stage 5 fails closed if any requirement in `(PRD ∪ CLARIFICATIONS)` lacks coverage. Tight PRDs produce empty `CLARIFICATIONS.md`. No tax for clarity.

### 4.4 Tech Stack

| Layer | Tech |
|---|---|
| Runtime | Claude Code (local-first, BYOK) |
| Model | Claude Sonnet 4.6 default; extended thinking on keystone agents |
| Parsing | MarkItDown (PDF / DOCX / MD / HTML) |
| Web research | Claude Code web search |
| Output | Plain markdown + YAML frontmatter (framework-agnostic) |
| CLI | Bun (planned) — fast install, one binary |
| Privacy | Local-first, BYOK, zero telemetry |

### 4.5 Repository Layout — Unchanged from V3

(See repository for current state. Layout matches V3 §4.5; only documentation files change in this V4 slice.)

---

## 5. Agent Catalog (V1 — 6 agents, V2 adds 6 more)

### 5.1 V1 — Currently Shipping (Stages 0, 1, 5)

| Agent | Stage | Role | Status |
|---|---|---|---|
| `@multi-agent-suitability-checker` | 0 | Runtime refusal gate | ✅ Live |
| `@prd-parser` | 1 | PRD → ProductSpec | ✅ Live |
| `@requirement-analyzer` | 1 | Numbered requirements | ✅ Live |
| `@clarification-auditor` | 1 | Audit log of ambiguities | ✅ Live |
| `@domain-researcher` | 1 | External context enrichment | ✅ Live |
| `@architecture-reviewer` (lightweight) | 5 | Traceability gate only in V1; full review is V2 | ⏳ V0.3 |

### 5.2 V2 — Planned (Stages 2–4)

| Agent | Stage | Role |
|---|---|---|
| `@architecture-planner` | 2 | Packaging, scale model, memory strategy |
| `@topology-selector` | 2 | Topology choice with defended rationale (the keystone) |
| `@agent-architect` | 3 | Proposes 5–11 agents with mandates |
| `@schema-designer` | 3 | Proposes AgentVerdict contract |
| `@orchestration-aggregator-designer` | 3 | Router + aggregator pattern |
| `@architecture-brief-writer` | 4 | Consolidates 2+3 into a single advisory markdown brief |

V2 is a single advisory document, not a full skill-pack generator. Lower risk, sharper deliverable.

---

## 6. Data Schemas

Authoritative schemas live in [`docs/SCHEMAS.md`](./docs/SCHEMAS.md). V4 changes:

- `SuitabilityVerdict` — unchanged
- `ProductSpec` — unchanged
- `RequirementSet` — unchanged
- `ClarificationLog` — unchanged
- `ReviewReport` — V1 uses a simplified version (traceability matrix only); V2 will introduce the full review report.

---

## 7. CLI Specification

```
prdforge check        Stage 0 only — runtime suitability verdict (~10s, ~$0.05)
prdforge clarify      Stage 0 + 1 — produces validated artifact bundle
prdforge trace        Stage 5 — emits traceability matrix for an existing bundle
prdforge bundle       Show what's in a generated bundle (PRD + clarifications + matrix)
prdforge advise       (V2) Run advisory architecture brief on a validated bundle
prdforge examples     List reference PRDs
```

V1 ships with the first four commands. `advise` is V2.

---

## 8. Build Plan

### 8.1 V0.3 (current sprint) — Complete V1
- [ ] `@architecture-reviewer` (lightweight — traceability gate only)
- [ ] CLI router with `check` / `clarify` / `trace` / `bundle` commands
- [ ] Output bundle structure and file emission
- [ ] End-to-end run on all four golden-set PRDs (good-fit + 3 bad-fit) producing artifact bundles
- [ ] Update README to reflect V1 = validated bundle output

### 8.2 V2 — Advisory Brief (Q3 2026)
- [ ] Architecture planner + topology selector
- [ ] Agent architect + schema designer + orchestration designer
- [ ] Architecture brief writer
- [ ] `prdforge advise` command

### 8.3 V3 — (Optional) Reference Skill-Pack Implementation
Decide based on V1+V2 reception.

---

## 9. Non-Functional Requirements

### 9.1 Performance

| Metric                                         | Target                           |
|------------------------------------------------|----------------------------------|
| Suitability check (Stage 0 only)               | < 10 seconds                     |
| Clarify-only run (Stage 0 + 1)                 | < 90 seconds (excluding user-answer time) |
| Token cost per V1 run                          | < $0.30 on Claude Sonnet 4.6     |
| Suitability gate accuracy on golden set        | 100%                             |
| Clarification auditor recall on golden set     | ≥ 80% of senior-reviewer flags   |
| Traceability gate: zero unmapped requirements  | 100% (fail-closed)               |

### 9.2 Quality Gates (V1 — Lightweight)

A V1 artifact bundle must pass:

1. Stage 0 returned `PROCEED` (else only the verdict is emitted, no bundle).
2. Every requirement has at least one testable acceptance criterion *or* an explicit "pending clarification" marker.
3. Every clarification entry has either a `user_answer` or a `user_skipped: true` flag.
4. Skipped clarifications are surfaced in `TRACEABILITY.md` as risk items.
5. Source PRD is bit-for-bit identical to the input file.

V2 introduces additional gates for the architecture brief.

### 9.3 Privacy & Compliance

- Local-first, BYOK, zero telemetry
- MIT license
- All output artifacts are plain markdown + JSON; no proprietary formats
- Designed to be auditable by external reviewers without running PRDForge itself

---

## 10. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Suitability gate has false positives (rejects good PRDs) | Extensive golden-set testing; confidence threshold tunable; appeal path via PRD revision |
| Suitability gate has false negatives (accepts bad PRDs) | Stage 5 lightweight review can flag downstream issues; community feedback loop |
| Users complain "too restrictive" | The restriction is the product; restate the value prop in the verdict reasoning |
| Clarification auditor over-asks | Tune to "material ambiguity only"; golden-set tests for false-positives |
| Audit-trail seen as bureaucracy | Empty `CLARIFICATIONS.md` is the common case for tight PRDs — no tax for clarity |
| Repositioning loses momentum from earlier "skill pack" framing | V1 still ships in V0.3; V4 doc is honest about the pivot |
| Regulated-industry adoption is slow | Solo/ML-engineer persona remains valid; product works for them too |
| Compliance landscape evolves faster than V1 ships | V1 outputs are framework-agnostic markdown + JSON; adapts to whatever the regulator finalizes |

---

## 11. Success Metrics

### 11.1 V1 Product Metrics (90-day post-V1.0)

| Metric                                          | Target |
|-------------------------------------------------|--------|
| GitHub stars                                    | 1,000  |
| Verdict-only runs (`prdforge check`)            | 200    |
| Full clarify runs (validated bundles produced)  | 50     |
| Correctly-rejected PRDs (public case studies)   | 5      |
| Suitability gate accuracy on community PRDs     | ≥ 90%  |
| Inbound interest from regulated-industry buyers | ≥ 3    |

### 11.2 Personal / Career Metrics

| Metric                                                       | 90-day target |
|--------------------------------------------------------------|---------------|
| Principal-level recruiter conversations triggered by PRDForge | ≥ 2           |
| Speaking / podcast invitations                                | ≥ 1           |
| Public "how I built this" write-up                            | Yes           |
| Citations in compliance / governance content                  | ≥ 1           |

### 11.3 Narrative Metrics

- At least one "PRDForge rejected my idea, and it was right" public testimonial.
- At least one regulated-industry team that adopts the artifact bundle as part of their AI procurement process.
- At least one architecture decision PRDForge surfaced that hand-building would have missed.

---

## 12. Launch Strategy

### 12.1 Two Narrative Tracks

**Track A — The "no" story** (general developer audience):
> "I built a tool that refuses to build the wrong thing. Here's why a tool that says no earns more trust than one that says yes to everything."

**Track B — The compliance story** (regulated-industry audience):
> "What an audit trail for AI-generated systems should look like — and why most tools still don't produce one."

Track A targets developer Twitter / Hacker News / personal blog. Track B targets compliance/governance content channels and direct outreach to regulated-industry buyers.

### 12.2 Launch Sequence (post-V1.0)

| Week | Beat |
|------|------|
| 1 | "I built a multi-agent governance tool. Fed it my own first idea. It refused. Here's why." |
| 2 | "What an EU AI Act-compatible audit trail for AI-generated systems looks like — with a working example." |
| 4 | Side-by-side: same PRD run through PRDForge vs. without. The traceability matrix is the differentiator. |
| 6 | "PRDForge surfaced ambiguities in 4 of 5 PRDs from a senior team I tested it against. Here's the audit log." |
| 8 | The "rejected my own idea" deep-dive post — the moat narrative. |

---

## 13. Open Questions

| #  | Question                                                              | Decision By |
|----|-----------------------------------------------------------------------|-------------|
| Q1 | Should V1 ship without `@architecture-reviewer` (Stage 5) or wait for it? | V0.3 |
| Q2 | Should the bundle output include a JSON-only manifest for CI consumption? | V0.3 |
| Q3 | Should `prdforge check` accept a confidence-threshold flag for tunable strictness? | V1.0 |
| Q4 | When/whether to ship V3 (reference skill-pack implementation) | After V2 reception |
| Q5 | Should there be a public, opt-in registry of rejected PRDs (with reasons) as a teaching corpus? | V1.1 |

---

## 14. Appendix

### 14.1 Why a Tool That Says "No" Earns Trust

In a category where misuse degrades performance by up to 70%, refusal is not friction — it is the highest-value feature. A tool that refuses calibrates user expectations honestly: users learn that a `PROCEED` verdict from PRDForge means something.

### 14.2 Why an Immutable Spec + Appendable Audit Log Is the Right Shape

This shape mirrors how mature engineering organizations already work: source specs in version control, decisions logged in ADRs, requirements traced through delivery. PRDForge formalizes the pattern for AI-driven development specifically.

### 14.3 Why "Validate the spec, refuse the unsuitable, prove what you built"

Three verbs, one product. Each verb maps to one of PRDForge's stages:

- **Validate** → Stage 1 (parser + analyzer + clarification auditor)
- **Refuse** → Stage 0 (suitability gate)
- **Prove** → Stage 5 (traceability gate)

If a future feature does not fit one of those three verbs, it does not belong in PRDForge.

---

## 15. Alignment with Emerging AI Governance Frameworks

PRDForge produces artifacts that materially help satisfy requirements in several governance frameworks. PRDForge is not a compliance product (it does not certify anything), but its outputs are the kind of evidence regulators ask for.

### 15.1 EU AI Act (effective 2026-08-02 for high-risk AI systems)

- **Article 12** — record-keeping and audit-trail requirements. PRDForge's `CLARIFICATIONS.md` and `TRACEABILITY.md` directly map to this.
- **Article 11** — technical documentation. The validated artifact bundle is the technical-documentation seed.
- **Article 17** — quality management system. The fail-closed Stage 5 gate fits the QMS verification step.

### 15.2 NIST AI Risk Management Framework

PRDForge's outputs support the **MEASURE** function (specifically requirement traceability) and the **MANAGE** function (decisions and the rationale behind them, captured in `CLARIFICATIONS.md`).

### 15.3 ISO/IEC 42001 (AI Management Systems)

The audit-trail discipline aligns with ISO 42001's requirement for documented decisions and traceable specifications throughout the AI development lifecycle.

PRDForge's defensibility against any future tool entering this space is the **integrated triple** — refusal verdict + clarifications log + traceability matrix as a coherent, single-run output. Standalone versions of each exist; the integrated form does not, and that is the lane V4 commits to.

---

*End of Document V4*
