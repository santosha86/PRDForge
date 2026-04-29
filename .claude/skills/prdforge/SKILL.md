---
name: prdforge
description: >
  Router for PRDForge — the PRD-to-Skill-Pack meta-framework. Coordinates the
  six pipeline stages and exposes user commands: check (Stage 0 only),
  clarify (Stage 0 + 1), generate (full pipeline), trace (coverage matrix),
  validate (run generated pack), explain (architecture decisions).
commands:
  - /prdforge check
  - /prdforge clarify
  - /prdforge generate
  - /prdforge trace
  - /prdforge validate
  - /prdforge explain
authoritative_spec: 06_PRDForge_PRD_V3.md
---

# /prdforge — Router

PRDForge reads a PRD and produces a runnable Claude Code multi-agent skill pack. It enforces two disciplines along the way:

1. **Suitability Gate (Stage 0)** — refuses to build multi-agent systems for problems that should be single-agent.
2. **Audit-grade traceability (Stage 1 + Stage 5)** — produces a `CLARIFICATIONS.md` log of every interpretation made and a coverage matrix proving every requirement landed somewhere in the generated pack.

The PRD is the source of truth: [`06_PRDForge_PRD_V3.md`](../../../06_PRDForge_PRD_V3.md).

---

## Commands

### `/prdforge check --prd <path>`

**Stage 0 only.** Cheap (~$0.05), fast (~10s).

Runs `@multi-agent-suitability-checker` on the PRD and emits a `SuitabilityVerdict`. Use this before committing to a full generate run.

```
@multi-agent-suitability-checker
   ↓
SuitabilityVerdict (PROCEED | REJECT)
```

If `REJECT`, the response includes `reason_code`, `reasoning`, and `alternative_recommendation`. The pipeline stops here.

### `/prdforge clarify --prd <path> --out <dir>`

**Stage 0 + Stage 1.** Produces the validated baseline without committing to full generation.

```
@multi-agent-suitability-checker → must PROCEED
   ↓
@prd-parser → ProductSpec
   ↓
@requirement-analyzer → numbered requirements (R-01…R-N)
   ↓
@clarification-auditor → CLARIFICATIONS.md (audit log)
   ↓ (user answers any clarifications in one batch)
   ↓
@domain-researcher → enriched context
```

Writes `<out>/CLARIFICATIONS.md` and `<out>/REQUIREMENTS.md`. The user can review before running `/prdforge generate` for the full pipeline.

### `/prdforge generate --prd <path> --out <dir> [--explain]`

**Full pipeline.** Stages 0 → 5. Aborts at Stage 0 if `REJECT`.

Produces:
- The runnable skill pack at `<out>/`
- `<out>/CLARIFICATIONS.md` — audit log
- `<out>/TRACEABILITY.md` — requirement → coverage matrix from Stage 5

### `/prdforge trace --pack <path>`

Reads the traceability matrix from a generated pack's review report and prints requirement-by-requirement coverage. Used to audit a previously generated pack.

### `/prdforge validate --pack <path> --sample <input>`

(V1.1) Runs the generated pack on a sample input end-to-end and reports whether the output is sensible. Closes the validation loop.

### `/prdforge explain --pack <path>`

Walks through every architecture decision PRDForge made — topology choice, agent roster rationale, schema design, rejected alternatives. Reads the decision artifacts emitted at each stage.

---

## Pipeline overview

```
[Stage 0] @multi-agent-suitability-checker          (sequential, gate)
    │ PROCEED
    ▼
[Stage 1] @prd-parser
          @requirement-analyzer                      (sequential)
          @clarification-auditor      ← writes CLARIFICATIONS.md
          @domain-researcher
    ▼
[Stage 2] @architecture-planner (supervisor)         (hierarchical)
            └─ @topology-selector ← keystone
    ▼
[Stage 3] @agent-architect
          @schema-designer                           (parallel fan-out)
          @orchestration-aggregator-designer
    ▼
[Stage 4] @skill-file-generator (per agent in roster) (parallel)
          @doc-test-generator
    ▼
[Stage 5] @architecture-reviewer + traceability gate (sequential, fail-closed)
    │ PASS → ship pack
    │ FAIL → route fix-hint upstream (max 2 retry cycles)
```

---

## Implementation status

This is **V0.1**. Currently implemented:

- ✅ `Stage 0 — @multi-agent-suitability-checker` (the moat)
- ⏳ Stage 1, 2, 3, 4, 5 — see [STATUS.md](../../../STATUS.md) for the live build log

When invoked today, only `/prdforge check` produces a real verdict. The other commands surface a "not yet implemented" message and point at the build status.

---

## Design constraints

These are non-negotiable and enforced via [CLAUDE.md](../../../CLAUDE.md) and the rubrics in [`docs/`](../../../docs/):

- The source PRD is **immutable**. No agent modifies it.
- Stage 0 can refuse. The pipeline must respect a `REJECT` verdict.
- Stage 5 fails closed. A pack with any unmapped requirement does not ship.
- Tight PRDs produce empty `CLARIFICATIONS.md`. No tax for clarity.
- Agent count is capped at 12 across all 6 stages.
