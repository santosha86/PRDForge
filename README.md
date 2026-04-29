# PRDForge

**A PRD-to-Skill-Pack meta-framework that reads your Product Requirements Document and emits a runnable Claude Code multi-agent skill pack — with an audit trail of every interpretation it made along the way.**

> Status: Active development. V1 shipping soon — see [STATUS.md](./STATUS.md).
> Latest spec: [PRD V3](./06_PRDForge_PRD_V3.md) · supersedes [V2](./05_PRDForge_PRD_V2.md).

---

## The Problem

Most multi-agent projects fail for one of two reasons:

1. **The problem never needed multi-agent in the first place.** A skill that could have been a 200-line CLI gets hammered into a 7-agent LangGraph DAG with a Pydantic schema layer, a state machine, and a vector store. Six weeks later, the demo is no better than a single well-prompted call.

2. **The spec was ambiguous, and the build silently picked an interpretation.** Six weeks in, the team realizes the system was built against a reading of the PRD that nobody actually agreed to. There's no audit trail of how the spec was interpreted, so there's no way to prove what was delivered against what was asked.

PRDForge attacks both.

## The Idea

PRDForge reads a PRD and runs two checkpoints before writing a single line of agent code:

1. **Should this even be multi-agent?** → A Stage 0 **Suitability Gate**
2. **Is the PRD specific enough to build against?** → A Stage 1 **Clarification Auditor**

If the Suitability Gate says single-agent is enough, PRDForge **refuses** to generate a multi-agent system — and tells you why.

If the Clarification Auditor finds ambiguity, contradictions, or gaps, it raises them in **one batch** and writes the user's answers to a structured `CLARIFICATIONS.md` log. The original PRD is never modified — interpretations are tracked in an additive audit log alongside it.

If the PRD is already tight, the audit log is empty. **Tight PRDs pay no tax.**

This is the differentiator: PRDForge is the only tool in this space that says **no**, and the only one that produces **provable spec→delivery traceability**.

## Architecture (V3 plan)

```
PRD (markdown)
   │
   ▼
┌──────────────────────────────────┐
│  Stage 0: Suitability Gate       │ ← fail-closed: refuses unsuitable problems
└─────────────┬────────────────────┘
              │ (approved)
              ▼
┌──────────────────────────────────┐
│  Stage 1: Clarification Auditor  │ ← detects ambiguity, gaps, contradictions
│           (+ Requirement Parser) │   writes CLARIFICATIONS.md (audit log)
└─────────────┬────────────────────┘
              │ (validated baseline)
              ▼
┌──────────────────────────────────┐
│  Stage 2: Agent Decomposer       │ ← extracts agent boundaries
└─────────────┬────────────────────┘
              ▼
┌──────────────────────────────────┐
│  Stage 3: Schema Synthesizer     │ ← AgentVerdict contracts per agent
└─────────────┬────────────────────┘
              ▼
┌──────────────────────────────────┐
│  Stage 4: Harness Emitter        │ ← Claude Code skill pack output
└─────────────┬────────────────────┘
              ▼
┌──────────────────────────────────┐
│  Stage 5: Traceability Gate      │ ← every requirement maps to coverage
└─────────────┬────────────────────┘
              ▼
        runnable skill pack
        + CLARIFICATIONS.md
        + traceability matrix
        ready to fork-and-go
```

## The Three-Document Audit Model

PRDForge produces and verifies against three artifacts:

| Document | Status | Mutability |
|---|---|---|
| **`PRD.md`** (input) | Source intent | **Immutable** — never modified |
| **`CLARIFICATIONS.md`** (generated, optional) | Audit log of resolved ambiguities | Append-only during Stage 1 |
| **Generated skill pack** (output) | Implementation | Replaceable across runs |

At Stage 5, the architecture reviewer compares **(PRD ∪ CLARIFICATIONS) → generated pack** and emits a coverage matrix. Any unmapped requirement → fail-closed. This is the "definition of done" for a PRDForge run.

## Why Skill Packs (not Python)

The output of PRDForge is a **fork-and-go** skill pack — markdown agent files, a router, a schema doc — designed to run inside Claude Code with zero installation. The same files port cleanly to a Python runtime later if you need multi-tenancy or custom orchestration.

This is the same architecture that powers [GhostCheck](https://github.com/santosha86/Ghost_Check) — proven across 11 agents in production.

## Why It Matters

Two disciplines, encoded in the pipeline:

- **"Single-agent until proven multi-agent."** Suitability Gating saves engineering teams weeks of wasted build time on problems that should have stayed simple. The cost of the wrong harness compounds — every extra agent makes the system harder to debug, slower to iterate, harder to reason about.
- **"Don't build until you know what you're building, and prove what you built against what you asked for."** The clarification auditor and traceability gate produce audit-grade evidence of how the spec was interpreted. In regulated industries — medical, finance, government — this isn't a nicety; it's a procurement requirement.

## Roadmap

- **V0.1** — PRD parser + Suitability Gate (in progress)
- **V0.2** — Requirement analyzer + **Clarification Auditor** (writes `CLARIFICATIONS.md`)
- **V0.3** — Agent decomposition with AgentVerdict schema synthesis
- **V0.4** — Harness emitter for Claude Code skill packs
- **V0.5** — Traceability gate (Stage 5 architecture reviewer)
- **V1.0** — End-to-end PRD → runnable skill pack + audit log + coverage matrix
- **V1.1** — Python runtime emitter (LangGraph target)

## Status

See [STATUS.md](./STATUS.md) for the live build log.
Latest design: [PRD V3](./06_PRDForge_PRD_V3.md).

## Author

Built by [Santosh Achanta](https://www.linkedin.com/in/santosh-achanta-ds/) — Principal AI Engineer, Saudi Arabia.

## License

MIT (planned). To be added with V1.0 release.
