# CLAUDE.md — PRDForge

This file is the orientation guide for Claude Code (and any other agent host) working inside this repository.

## What this repo is

PRDForge is a **PRD-to-Skill-Pack meta-framework**. It reads a Product Requirements Document and emits a runnable Claude Code multi-agent skill pack — but only after two checkpoints:

1. **Stage 0 — Suitability Gate**: refuses problems that aren't genuinely multi-agent
2. **Stage 1 — Clarification Audit**: surfaces ambiguity in the PRD and produces an audit log of how it was resolved

PRDForge is itself a hybrid multi-agent system, demonstrating the patterns it generates.

**Authoritative spec:** [`06_PRDForge_PRD_V3.md`](./06_PRDForge_PRD_V3.md). When this CLAUDE.md and the PRD disagree, the PRD wins.

## How the pipeline is structured

```
Stage 0  Suitability Gate           1 agent  (the first moat)
Stage 1  Understanding              4 agents (parser, analyzer, auditor, researcher)
Stage 2  Architecture               2 agents (planner, topology-selector — keystone)
Stage 3  Design                     3 agents (agent-architect, schema, orchestration)
Stage 4  Generation                 2 agents (skill-file, doc-test)
Stage 5  Review + Traceability      1 agent  (architecture-reviewer + the second moat)
                                    ─────
                                    12 agents total
```

Sequencing inside the pipeline:
- Stage 0 → Stage 1 → Stage 2 are **sequential**
- Stage 3 fans out **in parallel**
- Stage 4 fans out **in parallel** (skill-file-generator runs once per agent in the roster)
- Stage 5 is **sequential and fail-closed** — bad generations don't ship

## Repository layout

```
.claude/skills/
  prdforge/                          router (SKILL.md)
  stage0-gate/                       multi-agent-suitability-checker.md  ← THE MOAT
  stage1-understanding/              prd-parser, requirement-analyzer, clarification-auditor, domain-researcher
  stage2-architecture/               architecture-planner, topology-selector  ← THE KEYSTONE
  stage3-design/                     agent-architect, schema-designer, orchestration-aggregator-designer
  stage4-generation/                 skill-file-generator, doc-test-generator
  stage5-review/                     architecture-reviewer  ← traceability gate lives here
  templates/                         skill-pack templates the generator stamps out

docs/
  SCHEMAS.md                         all data contracts (Pydantic-shaped)
  SUITABILITY_RUBRIC.md              decision tree for Stage 0
  CLARIFICATION_RUBRIC.md            rules the Stage 1 auditor follows
  TOPOLOGY_RUBRIC.md                 how the keystone picks parallel/sequential/hierarchical/hybrid

examples/                            golden-set PRDs (good-fit + bad-fit) for testing
out/                                 where generated skill packs land
```

## The two moats — non-negotiable

These are the disciplines PRDForge enforces. Do not bypass them when working in this repo.

### Moat 1: Suitability Gate (Stage 0)

A genuine multi-agent problem must satisfy ALL of:
- Naturally decomposable sub-tasks
- Each sub-task benefits from specialized reasoning
- Either specialization or parallelization offers real value
- Coordination cost is acceptable

If any of those fails, Stage 0 must reject with a reason code (`SINGLE_AGENT_SUFFICIENT`, `NOT_AI_PROBLEM`, `WORKFLOW_AUTOMATION`, `SCOPE_TOO_VAGUE`, `SCALE_MISMATCH`). Full rubric: [`docs/SUITABILITY_RUBRIC.md`](./docs/SUITABILITY_RUBRIC.md).

### Moat 2: Audit-grade traceability (Stage 1 + Stage 5)

- The source PRD is **immutable**. Never modified by any agent.
- Clarifications produced by `@clarification-auditor` go to a separate `CLARIFICATIONS.md` audit log.
- Stage 5 fails closed if any requirement in `(PRD ∪ CLARIFICATIONS)` lacks coverage in the generated pack.
- Tight PRDs produce an empty clarifications log — no tax for clarity.

## Conventions

- **Skill files** are markdown with YAML frontmatter. Frontmatter declares `name`, `description`, `inputs`, `outputs`, `tools`, `temperature`, `extended_thinking`. The body is the system prompt.
- **Schemas** live in `docs/SCHEMAS.md` and are Pydantic-shaped (we don't run Pydantic at the skill level, but the shape is the contract).
- **Requirement IDs** are `R-01`, `R-02`, ... emitted by `@requirement-analyzer`. Stable across the run.
- **Clarification IDs** are `C-01`, `C-02`, ... emitted by `@clarification-auditor`.
- **Examples** in `examples/` are the golden set — every change to a Stage 0/1/5 agent must be re-tested against all golden-set PRDs.
- **The PRD is the source of truth.** When in doubt, read [`06_PRDForge_PRD_V3.md`](./06_PRDForge_PRD_V3.md).

## What PRDForge will NOT do

PRDForge stops at "testable." It does not:
- Ship the generated code (use gstack for that)
- Generate Python framework code (MetaGPT does that)
- Modify the source PRD (clarifications go to a separate log)
- Generate skill packs for problems Stage 0 rejected
- Estimate how long the final product will take

## When working in this repo

- Build the agents in the order the V3 build plan specifies (§8.1). Stage 0 first, then Stage 1, etc.
- Do not add agents beyond the 12 in the catalog without updating the PRD. Agent-count discipline is a deliberate constraint.
- Every agent must declare its `temperature`, `tools`, and whether `extended_thinking` is enabled.
- Every agent's output schema must match a definition in `docs/SCHEMAS.md`.
- Test each agent against the golden-set examples in `examples/` before considering it complete.
