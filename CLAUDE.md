# CLAUDE.md — PRDForge

This file is the orientation guide for Claude Code (and any other agent host) working inside this repository.

## What this repo is

PRDForge is a **validation + governance pre-flight layer for spec-driven AI development.** It reads a Product Requirements Document and produces a validated artifact bundle — a verdict, an audit log, and a requirement traceability matrix — but only after two checkpoints:

1. **Stage 0 — Suitability Gate**: refuses problems that aren't genuinely multi-agent (Moat 1)
2. **Stage 1 — Clarification Audit**: surfaces ambiguity in the PRD and produces an audit log of how it was resolved (Moat 2)

PRDForge does **not** generate runnable code. Its output is markdown + JSON, framework-agnostic, designed to feed whatever execution tool the user prefers downstream.

PRDForge is itself a hybrid multi-agent system, demonstrating the disciplines it enforces.

**Authoritative spec:** [`07_PRDForge_PRD_V4.md`](./07_PRDForge_PRD_V4.md). When this CLAUDE.md and the PRD disagree, the PRD wins. Earlier versions ([V3](./06_PRDForge_PRD_V3.md), [V2](./05_PRDForge_PRD_V2.md)) are preserved as audit history of the product's evolution — same immutability discipline V4 codifies for users.

## How the V1 pipeline is structured

```
Stage 0  Suitability Gate           1 agent  (Moat 1: runtime refusal)
Stage 1  Understanding              4 agents (parser, analyzer, auditor, researcher)
Stage 5  Traceability Gate          1 agent  (lightweight V1 version)
                                    ─────
                                    6 agents total in V1
```

V2 adds 6 more agents in Stages 2–4 (architecture / design) producing a single advisory markdown brief. V2 is **not a code generator**; it produces a report. V3 (optional, deferred) might add a reference skill-pack implementation if V1 + V2 reception demands it.

Sequencing inside the V1 pipeline:
- Stage 0 → Stage 1 → Stage 5 are **sequential**
- Stage 1 has 4 sequential sub-agents
- Stage 5 is **fail-closed** — bundles with unresolved coverage gaps don't ship

## Repository layout

```
.claude/skills/
  prdforge/                          router (SKILL.md)
  stage0-gate/                       multi-agent-suitability-checker.md  ← MOAT 1
  stage1-understanding/              prd-parser, requirement-analyzer, clarification-auditor, domain-researcher
  stage2-architecture/               (V2) architecture-planner, topology-selector
  stage3-design/                     (V2) agent-architect, schema-designer, orchestration-aggregator-designer
  stage4-generation/                 (V2) architecture-brief-writer
  stage5-review/                     (V0.3) architecture-reviewer  ← traceability gate lives here (MOAT 2 part 2)
  templates/                         skill-pack templates retained from V3 era; relabel to "advisory templates" in V2

docs/
  SCHEMAS.md                         all data contracts (Pydantic-shaped)
  SUITABILITY_RUBRIC.md              decision tree for Stage 0
  CLARIFICATION_RUBRIC.md            rules the Stage 1 auditor follows
  TOPOLOGY_RUBRIC.md                 (V2) advisory rubric for the topology selector

examples/                            golden-set PRDs (good-fit + bad-fit + ambiguous) for testing
out/                                 where validated artifact bundles land
```

## The two moats — non-negotiable

These are the disciplines PRDForge enforces. They are the product, not features within it. Do not bypass them when working in this repo.

### Moat 1: Suitability Gate (Stage 0)

A genuine multi-agent problem must satisfy ALL of:
- Naturally decomposable sub-tasks
- Each sub-task benefits from specialized reasoning
- Either specialization or parallelization offers real value
- Coordination cost is acceptable

If any fails, Stage 0 must reject with a reason code (`SINGLE_AGENT_SUFFICIENT`, `NOT_AI_PROBLEM`, `WORKFLOW_AUTOMATION`, `SCOPE_TOO_VAGUE`, `SCALE_MISMATCH`). Full rubric: [`docs/SUITABILITY_RUBRIC.md`](./docs/SUITABILITY_RUBRIC.md).

### Moat 2: Audit-grade traceability (Stage 1 + Stage 5)

- The source PRD is **immutable**. Never modified by any agent.
- Clarifications produced by `@clarification-auditor` go to a separate `CLARIFICATIONS.md` audit log.
- Stage 5 fails closed if any requirement in `(PRD ∪ CLARIFICATIONS)` lacks coverage in the downstream output.
- Tight PRDs produce an empty clarifications log — no tax for clarity.

## Conventions

- **Skill files** are markdown with YAML frontmatter. Frontmatter declares `name`, `description`, `inputs`, `outputs`, `tools`, `temperature`, `extended_thinking`. The body is the system prompt.
- **Schemas** live in `docs/SCHEMAS.md` and are Pydantic-shaped (we don't run Pydantic at the skill level, but the shape is the contract).
- **Requirement IDs** are `R-01`, `R-02`, ... emitted by `@requirement-analyzer`. Stable across the run.
- **Clarification IDs** are `C-01`, `C-02`, ... emitted by `@clarification-auditor`.
- **Examples** in `examples/` are the golden set — every change to a Stage 0/1/5 agent must be re-tested against all golden-set PRDs.
- **The PRD V4 is the source of truth.** When in doubt, read [`07_PRDForge_PRD_V4.md`](./07_PRDForge_PRD_V4.md).
- **Product positioning never names competitor tools.** Comparative analysis is internal strategy material, not user-facing.

## What PRDForge will NOT do

PRDForge stops at "validated and audit-ready." It does not:
- Generate runnable code (V1 / V2)
- Modify the source PRD (clarifications go to a separate log)
- Generate skill packs for problems Stage 0 rejected
- Estimate how long the final product will take
- Recommend specific frameworks (output is framework-agnostic)

## When working in this repo

- Build the agents in the order the V4 build plan specifies (§8). Stage 5 (V0.3) is the priority right now to close out V1.
- Do not add agents beyond the 12 in the catalog without updating the PRD. Agent-count discipline is a deliberate constraint.
- Every agent must declare its `temperature`, `tools`, and whether `extended_thinking` is enabled.
- Every agent's output schema must match a definition in `docs/SCHEMAS.md`.
- Test each agent against the golden-set examples in `examples/` before considering it complete.
- **Never name competitor tools in any product file** (README, PRDs from V4 onwards, SKILL files, STATUS.md). Internal/strategy notes only.
