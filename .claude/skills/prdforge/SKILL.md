---
name: prdforge
description: >
  Router for PRDForge — the validation + governance pre-flight layer for
  spec-driven AI development. Coordinates the V1 pipeline (Stages 0, 1, 5)
  and exposes user commands: check (Stage 0 only), clarify (Stages 0+1),
  trace (Stage 5 traceability matrix), bundle (inspect a generated bundle).
  V2 will add advise (architecture brief).
commands:
  - /prdforge check
  - /prdforge clarify
  - /prdforge trace
  - /prdforge bundle
  - /prdforge advise   # (V2)
authoritative_spec: 07_PRDForge_PRD_V4.md
---

# /prdforge — Router

PRDForge reads a PRD and produces a validated artifact bundle: a suitability verdict, a clarifications log, and a requirement traceability matrix. **It does not generate code.** Output is markdown + JSON, framework-agnostic. The bundle feeds whatever downstream execution tool the user prefers.

PRDForge enforces two disciplines:

1. **Runtime Suitability Refusal (Moat 1, Stage 0)** — refuses to proceed for problems that should be single-agent, deterministic code, or workflow automation.
2. **Audit-grade traceability (Moat 2, Stages 1 + 5)** — produces an immutable PRD copy + appendable `CLARIFICATIONS.md` log + requirement coverage matrix. Aligns with emerging governance frameworks (EU AI Act, NIST AI RMF, ISO 42001).

The PRD V4 is the source of truth: [`07_PRDForge_PRD_V4.md`](../../../07_PRDForge_PRD_V4.md). V2 and V3 are preserved as audit history.

---

## Commands

### `/prdforge check --prd <path>`

**Stage 0 only.** Cheap (~$0.05), fast (~10s).

Runs `@multi-agent-suitability-checker` and emits a `SuitabilityVerdict`. Use this before committing to a full clarify run.

```
@multi-agent-suitability-checker
   ↓
SuitabilityVerdict (PROCEED | REJECT)
```

If `REJECT`, the response includes `reason_code`, `reasoning`, and `alternative_recommendation`. The pipeline stops here and no bundle is produced.

### `/prdforge clarify --prd <path> --out <dir>`

**Stages 0 + 1.** Produces the validated artifact bundle.

```
@multi-agent-suitability-checker → must PROCEED
   ↓
@prd-parser → ProductSpec
   ↓
@requirement-analyzer → numbered requirements (R-01…R-NN)
   ↓
@clarification-auditor → CLARIFICATIONS.md (audit log)
   ↓ (user answers any clarifications in one batch)
   ↓
@domain-researcher → enriched context
```

Writes `<out>/PRD.md` (immutable copy of the source), `<out>/CLARIFICATIONS.md`, and `<out>/REQUIREMENTS.md`. Together they form the validated baseline ready for downstream execution.

### `/prdforge trace --bundle <path> --delivery <path>`

**Stage 5 traceability gate.** Validates that every requirement in `(PRD ∪ CLARIFICATIONS)` is covered in the downstream delivery, and emits `<bundle>/TRACEABILITY.md`.

Fail-closed on any requirement with `coverage_status: NONE`.

### `/prdforge bundle --show <path>`

Inspects a generated bundle and prints a summary: verdict, requirement count, clarification count, traceability status (if `TRACEABILITY.md` exists).

### `/prdforge advise --bundle <path>` *(V2 — not yet implemented)*

Will produce an advisory architecture brief (topology recommendation, agent roster proposal, schema design) as a single markdown report. **Not** a code generator.

---

## V1 pipeline overview

```
[Stage 0] @multi-agent-suitability-checker          (sequential, gate)
    │ PROCEED
    ▼
[Stage 1] @prd-parser
          @requirement-analyzer                      (sequential)
          @clarification-auditor      ← writes CLARIFICATIONS.md
          @domain-researcher
    ▼
[Stage 5] @architecture-reviewer (lightweight)       (sequential, fail-closed)
          → emits TRACEABILITY.md
    ▼
    Validated artifact bundle ready for downstream execution
```

---

## Implementation status

This is **V0.2**. Currently implemented:

- ✅ `Stage 0 — @multi-agent-suitability-checker` (Moat 1: runtime refusal)
- ✅ `Stage 1 — @prd-parser`
- ✅ `Stage 1 — @requirement-analyzer` (emits stable R-* IDs)
- ✅ `Stage 1 — @clarification-auditor` (Moat 2 part 1: writes CLARIFICATIONS.md)
- ✅ `Stage 1 — @domain-researcher` (web-search enrichment)
- ⏳ `Stage 5 — @architecture-reviewer` (lightweight V1 traceability gate) — **V0.3, in progress**

`/prdforge check` and `/prdforge clarify` are runnable end-to-end today. `/prdforge trace` ships in V0.3 alongside the Stage 5 reviewer. V1.0 is V0.3 → public release.

---

## Design constraints

These are non-negotiable and enforced via [CLAUDE.md](../../../CLAUDE.md) and the rubrics in [`docs/`](../../../docs/):

- The source PRD is **immutable**. No agent modifies it.
- Stage 0 can refuse. The pipeline must respect a `REJECT` verdict.
- Stage 5 fails closed. A bundle with any unmapped requirement does not pass.
- Tight PRDs produce empty `CLARIFICATIONS.md`. No tax for clarity.
- Output is markdown + JSON only. Framework-agnostic. No code generation in V1 or V2.
- Agent count is capped at 12 across all 6 stages (V1 uses 6, V2 adds 6 more).
- Product docs never name competitor tools. Positioning stands on its own merits.
