# Product Requirements Document (PRD) — V3
# PRDForge — PRD-to-Skill-Pack Meta-Framework

| Field            | Detail                                                     |
|------------------|------------------------------------------------------------|
| **Document ID**  | PF-PRD-003                                                 |
| **Version**      | 3.0 — Adds Clarification Audit Trail                       |
| **Date**         | April 2026                                                 |
| **Author**       | Santosh Achanta                                            |
| **Status**       | Draft — Pre-build                                          |
| **Repo**         | github.com/santosha86/PRDForge                             |
| **License**      | MIT                                                        |
| **Supersedes**   | PF-PRD-002                                                 |

---

## Changelog V2 → V3

| Change | Why |
|---|---|
| Added new agent `@clarification-auditor` in Stage 1 | Closes the loop on "validated baseline" — without it, traceability has nothing to compare against |
| Added §4.7 — Clarification Audit Trail | Documents the new artifact (`CLARIFICATIONS.md`) and the rationale for an additive log over a rewritten PRD |
| Added §6.7 — `ClarificationLog` schema | Structured Q&A — disambiguation vs. extension vs. new-requirement |
| Added §3.6 — The Three-Document Audit Model | Original PRD stays immutable; CLARIFICATIONS.md is additive; generated pack is verified against the union |
| Added Quality Gate #11 — Requirement Traceability | Every numbered requirement in (PRD ∪ CLARIFICATIONS) must map to coverage in the pack |
| Agent count: **11 → 12** | Acknowledged tradeoff. Research still holds (3–5 optimal); 12 is a deliberate exception with a different cognitive shape — auditor *detects what's missing*, analyzer *extracts what's there* |
| Added §1.8 — The Second Differentiator | Audit-grade traceability is a separate moat alongside Stage 0 — together they form the discipline thesis |

---

## 1. Executive Summary

### 1.1 Problem Statement

A person with an idea for a multi-agent product faces two expensive options today:

1. **Build the Python/framework version first** (LangGraph, CrewAI, AutoGen) — 3–8 weeks of engineering before they know if the idea has legs. Most ideas die here.
2. **Hand-build a Claude Code skill pack** — faster (5 days) but still requires expert architectural decisions: which topology, how many agents, what contracts, what fail-closed behavior. Getting these wrong tanks performance by up to 70% (Google / Anthropic research).

Both paths waste time because the **idea validation loop is buried under architecture decisions.** You can't know if your multi-agent concept works until you've already built half a product.

And there's a third, quieter failure mode: **building exactly what the spec said, when the spec was ambiguous to begin with.** Without a validated baseline, "the system did what the PRD asked" is meaningless.

### 1.2 Product Vision

PRDForge reads a PRD and produces a **testable skill pack** — but only after two checkpoints:

1. **Stage 0: Suitability Gate** — Is this even a multi-agent problem?
2. **Stage 1: Clarification Audit** — Is the PRD specific enough to build against?

If both pass, PRDForge generates a runnable skill pack and produces a third artifact alongside it: a `CLARIFICATIONS.md` audit log of every ambiguity surfaced and how the user resolved it.

The skill pack PRDForge generates is deliberately minimal: enough to run end-to-end, produce real output, and tell you whether the idea is worth pursuing further.

Critical design choice: **PRDForge is itself a hybrid multi-agent system, demonstrating the exact patterns it generates.**

### 1.3 The First Differentiator: Multi-Agent Suitability Gate (Stage 0)

Before PRDForge designs anything, a gate agent answers one question:

> **"Is this problem actually a multi-agent problem?"**

If the answer is no, PRDForge refuses to generate a skill pack and explains why. Most "AI builder" tools happily produce multi-agent bloat for problems that should be single-agent. PRDForge is the one that says no.

### 1.4 Why This Exists

Four gaps in the April-2026 landscape:

1. **Y Combinator's gstack (80K stars) is a general software factory** — it ships software but does NOT design multi-agent architectures.
2. **MetaGPT outputs Python code** — not Claude Code skill packs.
3. **Anthropic's official skills repo is a registry** — not a generator.
4. **No existing tool rejects PRDs that shouldn't be multi-agent**, and **no existing tool produces an audit trail of how it interpreted the PRD.**

PRDForge fills the intersection.

### 1.5 The Killer Feature

> **"Validate the idea in a day, not a month — with an audit trail of every interpretation made."**

You give PRDForge a PRD on Monday morning. Monday evening, you have a runnable skill pack, a clarifications log of every ambiguity it surfaced, and a traceability matrix showing which requirements landed where in the generated pack.

### 1.6 Positioning vs gstack (Unchanged from V2)

| Dimension | gstack | PRDForge |
|---|---|---|
| Primary user | Technical founder shipping a product | Multi-agent builder testing an idea |
| Input | Natural-language idea | Structured PRD document |
| Output | Working SaaS | A testable skill pack + audit log |
| Topology awareness | N/A | Keystone decision |
| Audit trail of interpretation | N/A | First-class artifact (V3) |
| Ideal outcome | "I shipped a v1" | "I know if my multi-agent idea has legs, and I can prove what we built vs. what I asked for" |

### 1.7 Success Definition

- Product: PRDForge generates GhostCheck from its PRD with ≥ 90% fidelity to the hand-built version.
- Product: Stage 0 correctly rejects 3 "not actually multi-agent" test PRDs.
- Product: Clarification auditor surfaces ≥ 80% of ambiguities a senior reviewer would flag (golden-set test).
- Signal: 1,500 GitHub stars within 90 days.
- Career: At least one Principal-level conversation explicitly triggered by PRDForge's existence.

### 1.8 The Second Differentiator: Audit-Grade Traceability (NEW in V3)

Stage 0 is a *front-door* moat — it stops you from building the wrong kind of system. The clarification auditor and traceability gate are a *back-door* moat — they stop you from building the right kind of system that *solves the wrong problem*.

Together, they form PRDForge's discipline thesis:

> **Don't build until you know what you're building, and prove what you built against what you asked for.**

No other generator in the category produces an audit trail of how it interpreted the spec. This is the second moat.

---

## 2. Target Users

| Persona                          | Pain                                              | Why PRDForge Helps                                                |
|----------------------------------|---------------------------------------------------|--------------------------------------------------------------------|
| Solo multi-agent builder         | Spends weeks designing before shipping            | PRD in → testable skill pack out in ~15 minutes                   |
| ML/AI engineer exploring agents  | Unsure if a problem is even suitable for agents   | Stage 0 gate gives an honest verdict before any build             |
| Enterprise AI lead evaluating ideas | Budget doesn't support multi-week spikes       | Spin up testable skill pack in a day, decide fast                 |
| Regulated-industry teams (medical, finance, government) | Need provable spec→delivery traceability for audit | CLARIFICATIONS.md log + traceability gate produce audit-grade evidence |
| Teacher / trainer of multi-agent | Junior engineers over-engineer with LangGraph     | PRDForge demonstrates correct architecture including "no, don't"  |
| Claude Code skill pack author    | Doesn't know where to start on new product        | Fork PRDForge, run on their PRD, get working scaffolding          |

Explicit **non-users** — PRDForge is NOT for:

- Founders shipping SaaS products (use gstack)
- Teams that already know their architecture (just write the skill pack)
- Problems that aren't multi-agent in nature (PRDForge will reject them)

---

## 3. Scope

### 3.1 V1 — Core Generator with Suitability Gate + Clarification Audit (Week 1–2)

| ID      | Feature                        | Description                                                                |
|---------|--------------------------------|----------------------------------------------------------------------------|
| V1-01   | PRD Parser                     | PDF/DOCX/MD → structured `ProductSpec` JSON                               |
| V1-02   | **Multi-Agent Suitability Gate** (Stage 0) | Reject-or-proceed decision with rationale           |
| V1-03   | Requirement Analyzer           | Extract numbered requirements (`R-01…R-N`) with acceptance criteria       |
| V1-04   | **Clarification Auditor (NEW)** | Detect ambiguity / contradictions / gaps; produce `CLARIFICATIONS.md`     |
| V1-05   | Domain Researcher              | Identify industry, failure modes, existing competitors (via web search)   |
| V1-06   | Architecture Planner           | Decide packaging, scale model, memory strategy                            |
| V1-07   | Topology Selector              | Choose parallel / sequential / hierarchical / hybrid — with evidence      |
| V1-08   | Agent Architect                | Propose 5–11 agents with mandates, tiers, tool needs                      |
| V1-09   | Schema Designer                | Design the agent output contract for this product                         |
| V1-10   | Orchestration + Aggregator Designer (combined) | Router SKILL.md + combination logic                |
| V1-11   | Skill File Generator           | Write actual .md files for every proposed agent                           |
| V1-12   | Doc + Test Generator (combined)| README, ROADMAP, SCHEMAS.md + sample inputs                               |
| V1-13   | **Architecture Reviewer + Traceability Gate (UPDATED)** | Fail-closed validator + requirement coverage check       |
| V1-14   | CLI                            | `prdforge generate --prd X.md --out ./ghostcheck/`                         |

**Note on agent count:** V3 PRDForge has **12 agents** (was 11 in V2). The clarification auditor is a deliberate exception — it has a different cognitive shape (detecting what's *missing*) than the analyzer (extracting what's there).

### 3.2 V1.1 — Idea Validation Loop (Week 3)

| ID        | Feature                    | Description                                                           |
|-----------|----------------------------|-----------------------------------------------------------------------|
| V1.1-01   | Auto-run generated skill pack | Execute the generated pack on sample input immediately after generation |
| V1.1-02   | Validation Report          | Is the output sensible? Does the topology chosen actually fit?         |
| V1.1-03   | Idea Scorecard             | Shareable PNG: "Multi-agent fit = 8/10, Topology = parallel fan-out, First output sample" |
| V1.1-04   | Refinement Pass            | User feedback → regenerate specific stages                             |

### 3.3 V1.2 — Portfolio & Cross-Pack Consistency (Week 4)

(Unchanged from V2.)

### 3.4 Explicitly Out of Scope (Unchanged from V2)

(See V2.)

### 3.5 What PRDForge Does NOT Do (Anti-Features)

(See V2 — extended only by: PRDForge **will not silently rewrite the source PRD**. All clarifications go to `CLARIFICATIONS.md`.)

### 3.6 The Three-Document Audit Model (NEW in V3)

PRDForge produces and verifies against three artifacts:

| Document | Status | Mutability | Source of |
|---|---|---|---|
| **`PRD.md`** (input) | Source intent | **Immutable** — never modified | What the user asked for |
| **`CLARIFICATIONS.md`** (generated, optional) | Audit log | Append-only during Stage 1 | How ambiguities were resolved |
| **Generated skill pack** (output) | Implementation | Replaceable across runs | What was built |

At Stage 5, the architecture reviewer compares **(PRD ∪ CLARIFICATIONS) → generated pack** and emits a coverage matrix. This is the "definition of done" for a PRDForge run.

If `CLARIFICATIONS.md` is empty, the run still passes — it just means the PRD was tight enough to build directly. **Tight PRDs pay no tax.**

---

## 4. Architecture

PRDForge is a hybrid multi-agent system demonstrating the patterns it generates.

### 4.1 High-Level Flow (Updated for V3)

```
PRD.md input
     │
     ▼
┌────────────────────────────────────────┐
│ STAGE 0 — Suitability Gate             │
│   @multi-agent-suitability-checker     │ ← FIRST MOAT (can reject)
│   Output: VERDICT + reasoning          │
└─────────────────┬──────────────────────┘
          REJECT  │  PROCEED
            │     │
            ▼     ▼
     Explain   STAGE 1 — Understanding (Sequential)
     & exit   ┌────────────────────────────────────┐
              │  @prd-parser                        │
              │  @requirement-analyzer              │ → R-01…R-N
              │  @clarification-auditor (NEW)       │ ← SECOND MOAT
              │     ↓                                │   (asks user; writes
              │  CLARIFICATIONS.md                   │    CLARIFICATIONS.md)
              │  @domain-researcher                  │
              └─────────────────┬───────────────────┘
                                ▼
              STAGE 2 — Architecture (Hierarchical)
              ┌────────────────────────────────────┐
              │  @architecture-planner (supervisor)│
              │      └─→ @topology-selector        │ ← KEYSTONE
              └─────────────────┬───────────────────┘
                                ▼
              STAGE 3 — Design (Parallel Fan-Out)
              ┌────────────────────────────────────┐
              │  @agent-architect                  │
              │  @schema-designer                  │
              │  @orchestration-aggregator-designer│
              └─────────────────┬───────────────────┘
                                ▼
              STAGE 4 — Generation (Parallel)
              ┌────────────────────────────────────┐
              │  @skill-file-generator             │
              │  @doc-test-generator               │
              └─────────────────┬───────────────────┘
                                ▼
              STAGE 5 — Review (Sequential, fail-closed)
              ┌────────────────────────────────────┐
              │  @architecture-reviewer            │
              │  + Traceability Gate (NEW):        │
              │    every R-* in (PRD ∪ CLARIF.)    │
              │    must map to coverage            │
              └─────────────────┬───────────────────┘
                                ▼
                   ✅ Complete skill pack
                   + CLARIFICATIONS.md
                   + traceability matrix
                   at ./out/<slug>/
```

### 4.2 The Topology Decision — The Keystone

(Unchanged from V2 — see §4.2 there.)

### 4.3 Stage 0 — The Suitability Gate

(Unchanged from V2 — see §4.3 there.)

### 4.4 Tech Stack

(Unchanged from V2 — see §4.4 there.)

### 4.5 Repository Layout (Updated)

```
prdforge/
  .claude/
    skills/
      prdforge/
        SKILL.md
      stage0-gate/
        multi-agent-suitability-checker.md
      stage1-understanding/
        prd-parser.md
        requirement-analyzer.md
        clarification-auditor.md             # NEW IN V3
        domain-researcher.md
      stage2-architecture/
        architecture-planner.md
        topology-selector.md
      stage3-design/
        agent-architect.md
        schema-designer.md
        orchestration-aggregator-designer.md
      stage4-generation/
        skill-file-generator.md
        doc-test-generator.md
      stage5-review/
        architecture-reviewer.md
      templates/
        skill-pack-template/
        agent-skill-template.md
        schemas-template.md
        clarifications-template.md           # NEW IN V3
        readme-template.md
  examples/
    good-fit-ghostcheck-prd.md
    good-fit-bidforge-prd.md
    bad-fit-summarizer-prd.md
    bad-fit-crud-app-prd.md
    bad-fit-zapier-workflow-prd.md
    ambiguous-prd.md                         # NEW IN V3 (forces clarifications)
  out/
    .gitkeep
  docs/
    SCHEMAS.md
    TOPOLOGY_RUBRIC.md
    SUITABILITY_RUBRIC.md
    CLARIFICATION_RUBRIC.md                  # NEW IN V3
    TRACEABILITY_GATE.md                     # NEW IN V3
    WHY_NOT_GSTACK.md
    HARNESS_ENGINEERING.md
    ROADMAP.md
    ZERO_TRUST.md
  CLAUDE.md
  README.md
  STATUS.md
  LICENSE
```

### 4.6 Key Design Choices

(See V2 — extended in V3 with:)

| Choice | Rationale |
|---|---|
| Clarification auditor is its own agent, not folded into analyzer | Different cognitive shape — extraction vs. ambiguity-detection. Distinct prompts, temperatures, outputs. |
| Source PRD is never modified | Audit-trail discipline; matches regulated-industry spec management |
| `CLARIFICATIONS.md` is append-only during a run | Each Q&A timestamped + tagged; supports replay and review |
| Tight PRDs produce empty clarifications log | Friction scales with input ambiguity; well-written PRDs pay no tax |
| Traceability gate at Stage 5 (not earlier) | Coverage check needs the *generated pack* to compare against |

### 4.7 Clarification Audit Trail (NEW in V3)

The `@clarification-auditor` runs after `@requirement-analyzer` in Stage 1. Its mandate is narrow and distinct from the analyzer's: **detect what's missing, contradictory, or undefined — not what's there.**

**Decision tree for `@clarification-auditor`:**

```
For each requirement R-* and the objective:

  IF a key term is undefined        → raise UNDEFINED_TERM
  IF two requirements conflict      → raise CONTRADICTION
  IF a measurable lacks a target    → raise MISSING_DETAIL
                                       (e.g., "fast" with no latency budget)
  IF a behavior is implied not spec → raise GAP
                                       (e.g., what happens on error?)
  IF nothing ambiguous              → emit no clarification

If zero clarifications raised across all requirements:
  → emit "PRD is unambiguous. No clarifications needed."
  → user confirms once; pipeline proceeds
```

**Interaction model:**

The auditor surfaces clarifications **in one batch**, not per-requirement-one-at-a-time. The user answers them in a single review pass. This preserves the "validate in a day" promise while still capturing every ambiguity.

**Output: `CLARIFICATIONS.md`** — structured, append-only, audit-grade. See §6.7 for schema.

**What the auditor will NOT do:**

- Will not modify the source PRD
- Will not invent requirements the PRD didn't suggest
- Will not block on stylistic ambiguity (only material ambiguity)
- Will not ask the user to confirm things that are already clear

---

## 5. Agent Catalog (12 agents across 6 stages)

### 5.1 Stage 0 — Suitability Gate (1 agent)

(Unchanged from V2.)

### 5.2 Stage 1 — Understanding (4 agents, Sequential) — UPDATED

#### 1.1 `@prd-parser`
Convert unstructured PRD to `ProductSpec`. No tools. Fails closed on malformed input.

#### 1.2 `@requirement-analyzer`
Extract discrete functional requirements with acceptance criteria. Emits numbered list `R-01…R-N`. No tools.

#### 1.3 `@clarification-auditor` (NEW IN V3)
**Mandate:** Detect ambiguity, contradictions, undefined terms, and gaps in the parsed requirements. Never modifies the source PRD.
**Input:** `ProductSpec` + numbered requirements from `@requirement-analyzer`.
**Output:** `ClarificationLog` (see §6.7) → written to `CLARIFICATIONS.md`.
**Tools:** None.
**Temperature:** 0.2 (consistency matters).
**Extended thinking:** Enabled.
**User interaction:** One batch — surfaces all clarifications, user answers in single review pass.
**Failure handling:** If user declines to answer a clarification, the auditor records `user_skipped: true` and the analyzer's original requirement stands as-is. Stage 5 traceability gate flags skipped clarifications as risk items but does not fail the run.

#### 1.4 `@domain-researcher`
Enrich with external context — competitors, failure modes, industry norms. Tools: Web search.

### 5.3 Stage 2 — Architecture (2 agents, Hierarchical)
(Unchanged from V2.)

### 5.4 Stage 3 — Design (3 agents, Parallel Fan-Out)
(Unchanged from V2.)

### 5.5 Stage 4 — Generation (2 agents, Parallel Fan-Out)
(Unchanged from V2.)

### 5.6 Stage 5 — Review (1 agent, Sequential, Fail-Closed) — UPDATED

#### 5.1 `@architecture-reviewer`
Validates generated skill pack against quality gates (see §9.2 — now 11 gates including the new traceability gate). On fail, routes back to Stage 2 or 3 with specific fix hints. Maximum 2 retry cycles before final fail.

**New responsibility (V3):** produce the **traceability matrix** mapping every requirement in `(PRD ∪ CLARIFICATIONS)` to the agent(s) and feature(s) in the generated pack that cover it. Any unmapped requirement → fail.

---

## 6. Data Schemas

### 6.1 `SuitabilityVerdict` (Unchanged from V2)
### 6.2 `ProductSpec` (Unchanged from V2)
### 6.3 `TopologyChoice` (Unchanged from V2)
### 6.4 `ArchitecturePlan` (Unchanged from V2)
### 6.5 `AgentRoster` (Unchanged from V2)
### 6.6 `ReviewReport` (Updated — adds `traceability_matrix` field)

```python
class ReviewReport(BaseModel):
    verdict: Literal["PASS", "FAIL"]
    failed_gates: list[str]
    fix_hints: list[str]
    retry_count: int
    traceability_matrix: list[RequirementCoverage]   # NEW IN V3

class RequirementCoverage(BaseModel):
    requirement_id: str           # e.g. "R-07"
    requirement_text: str
    source: Literal["PRD", "CLARIFICATION"]
    covered_by: list[str]          # agent names or feature paths
    coverage_status: Literal["FULL", "PARTIAL", "NONE"]
    notes: str | None
```

### 6.7 `ClarificationLog` (NEW IN V3)

```python
class ClarificationEntry(BaseModel):
    clarification_id: str             # "C-01", "C-02", ...
    raised_by: str                    # "@clarification-auditor"
    timestamp: datetime               # ISO 8601
    relates_to: str                   # "R-07" or "OBJECTIVE"
    ambiguity_type: Literal[
        "MISSING_DETAIL",
        "CONTRADICTION",
        "UNDEFINED_TERM",
        "GAP",
    ]
    question: str                     # what the auditor asked
    user_answer: str | None           # what the user replied
    user_skipped: bool                # true if user declined to answer
    resolution_kind: Literal[
        "DISAMBIGUATION",             # clarifies existing requirement
        "EXTENSION",                  # extends scope of existing req
        "NEW_REQUIREMENT",            # surfaces a new requirement
    ] | None
    new_requirement_id: str | None    # if resolution_kind == NEW_REQUIREMENT

class ClarificationLog(BaseModel):
    prd_path: str
    generated_at: datetime
    auditor_verdict: Literal[
        "NO_CLARIFICATIONS_NEEDED",
        "CLARIFICATIONS_RESOLVED",
        "CLARIFICATIONS_PARTIALLY_SKIPPED",
    ]
    entries: list[ClarificationEntry]
```

---

## 7. CLI Specification

```
prdforge check         Run Stage 0 only — is this multi-agent suitable?
prdforge clarify       Run Stage 0 + Stage 1 only — produce CLARIFICATIONS.md (NEW IN V3)
prdforge generate      Full pipeline if suitable, else fails with reason
prdforge regen         Re-generate one stage after PRD updates
prdforge diff          Show what changed between two generations
prdforge explain       Walk through the architecture decisions made
prdforge trace         Show traceability matrix for a generated pack (NEW IN V3)
prdforge validate      Run the generated pack on sample input (V1.1)
prdforge examples      List reference PRDs (good-fit + bad-fit)
```

### 7.1 New Commands (V3)

**Clarify only (cheap, fast — see ambiguities before committing to full generate):**
```bash
prdforge clarify --prd ideas/my-idea-prd.md --out ./out/myidea/
# Runs Stage 0 + Stage 1
# Produces: CLARIFICATIONS.md with auditor's questions
# User answers, then re-runs `prdforge generate` for full pipeline
```

**Trace coverage:**
```bash
prdforge trace --pack ./out/ghostcheck/
# Shows: requirement-id → covered-by(agent/feature) → status
# Reads the traceability_matrix from the pack's review report
```

---

## 8. Build Plan

### 8.1 V1 — Weeks 1–2 (Updated)

**Week 1 — Suitability gate + understanding (now includes auditor)**
| Day | Work                                                                 |
|-----|----------------------------------------------------------------------|
| 1   | Repo scaffold, CLAUDE.md, SCHEMAS.md, SUITABILITY_RUBRIC.md, CLARIFICATION_RUBRIC.md, TOPOLOGY_RUBRIC.md |
| 2   | **Stage 0: @multi-agent-suitability-checker** (the moat)             |
| 3   | Stage 1 agents: @prd-parser, @requirement-analyzer                   |
| 4   | **Stage 1: @clarification-auditor (NEW)** + @domain-researcher       |
| 5   | Stage 2: @topology-selector + @architecture-planner. End-to-end Stage 0→1→2 on GhostCheck PRD |

**Week 2 — Design, generation, review with traceability**
| Day | Work                                                                 |
|-----|----------------------------------------------------------------------|
| 6   | Stage 3: @agent-architect + @schema-designer                         |
| 7   | Stage 3: @orchestration-aggregator-designer                          |
| 8   | Stage 4: @skill-file-generator                                       |
| 9   | Stage 4: @doc-test-generator                                         |
| 10  | **Stage 5: @architecture-reviewer + Traceability Gate**. End-to-end run generating GhostCheck. Compare to hand-built. |

### 8.2 V1.1 — Week 3 (Unchanged)
### 8.3 V1.2 — Week 4 (Unchanged)

---

## 9. Non-Functional Requirements

### 9.1 Performance (Updated)

| Metric                                         | Target                           |
|------------------------------------------------|----------------------------------|
| Suitability check (Stage 0 only)               | < 10 seconds                     |
| Clarify-only run (Stage 0 + 1)                 | < 90 seconds (excluding user-answer time) |
| Full generation end-to-end                     | < 15 minutes                     |
| Token cost per generation                      | < $3 on Claude Sonnet 4.6        |
| Generated skill pack works first try           | ≥ 70% of the time                |
| Suitability gate accuracy on golden set        | 100%                             |
| Clarification auditor recall on golden set     | ≥ 80% of senior-reviewer flags   |
| Traceability gate: zero unmapped requirements  | 100% (fail-closed)               |

### 9.2 Quality Gates (Updated — now 11 gates)

A generated skill pack must pass ALL of:

1. Every agent has a narrow, one-sentence mandate
2. Every agent has YAML frontmatter with capabilities
3. Output schema is consistent across all agents
4. Aggregator matches topology choice
5. Router SKILL.md references the correct topology
6. Capability manifests exist for every tool-using agent
7. README has quickstart + architecture diagram
8. HARNESS_ENGINEERING.md documents the moat
9. Sample inputs exist for at least one agent
10. The generated pack actually runs end-to-end on its sample input (V1.1)
11. **Traceability gate (NEW IN V3):** every requirement in `(PRD ∪ CLARIFICATIONS)` maps to at least one agent or feature in the generated pack with `coverage_status: FULL` or `PARTIAL`

If any fail → block shipment, route specific fix hint back to stage. Max 2 retry cycles.

### 9.3 Privacy & Compliance (Unchanged)

---

## 10. Risks and Mitigations (Updated — V3 additions)

| Risk                                              | Mitigation                                                       |
|---------------------------------------------------|------------------------------------------------------------------|
| (V2 risks unchanged)                              |                                                                  |
| Clarification auditor over-asks (annoys users)    | Tune to "material ambiguity only"; golden-set tests for false-positives |
| Clarification auditor under-asks (misses things)  | Stage 5 traceability gate catches downstream coverage misses     |
| User skips clarifications, then complains pack is wrong | Skipped clarifications flagged in traceability matrix as risk items |
| Traceability gate too strict (blocks reasonable packs) | `coverage_status: PARTIAL` is allowed; only `NONE` fails       |
| Audit log seen as bureaucracy                     | Empty `CLARIFICATIONS.md` is the common case for tight PRDs — no tax for clarity |

---

## 11. Success Metrics (Unchanged from V2)

---

## 12. Launch Strategy (Updated)

### 12.1 The Key Posts (Week 3–6)

The story now has two beats:

1. **The "no" post** (Week 3, from V2): *"PRDForge rejected my own idea. Here's why it was right."*
2. **The "audit" post (NEW)** (Week 5): *"PRDForge surfaced 14 ambiguities in a PRD I thought was tight. Here's the audit log — and the requirement I would have shipped wrong."*

Both stories land the same theme: a tool that refuses to pretend the spec is complete when it isn't.

---

## 13. Open Questions (V3 additions)

| #  | Question                                                              | Decision By |
|----|-----------------------------------------------------------------------|-------------|
| Q1–Q5 | (V2 questions, unchanged)                                          | —           |
| Q6 | Should `prdforge clarify` accept programmatic answers (JSON file) for CI use? | V1.1 |
| Q7 | Should the auditor support a `--severity-threshold` flag (only ask about CRITICAL gaps)? | Week 2 |
| Q8 | When user updates the PRD after clarifications, should CLARIFICATIONS.md be invalidated automatically? | Week 2 |

---

## 14. Appendix

### 14.1 — 14.7 (See V2)

### 14.8 Why the Audit Trail Is the Second Moat (NEW IN V3)

The first moat (Stage 0) is the *front door*: it stops you from building the wrong kind of system entirely.

The second moat (clarification auditor + traceability gate) is the *back door*: it stops you from building the right kind of system that solves the wrong problem.

Together they enforce one discipline:

> **Don't build until you know what you're building, and prove what you built against what you asked for.**

Every other generator in this category will happily build whatever the spec said, even when the spec was ambiguous, even when the delivery diverges. PRDForge is the one that produces evidence.

In regulated industries — medical, finance, government — this audit trail isn't a nicety. It's a procurement requirement. PRDForge's V3 design opens the door to those buyers without changing the core product.

---

*End of Document V3*
