# PRDForge

**A PRD-to-Skill-Pack meta-framework that reads your Product Requirements Document and emits a runnable Claude Code multi-agent skill pack.**

> Status: Active development. V1 shipping soon — see [STATUS.md](./STATUS.md).

---

## The Problem

Most multi-agent projects fail not because the agents are wrong — but because the problem **never needed multi-agent in the first place**.

A skill that could have been a 200-line CLI gets hammered into a 7-agent LangGraph DAG with a Pydantic schema layer, a state machine, and a vector store. Six weeks later, the demo is no better than a single well-prompted call.

## The Idea

PRDForge reads a PRD and answers two questions before writing a single line of agent code:

1. **Should this even be multi-agent?** → A Stage 0 Suitability Gate
2. **If yes, what's the smallest harness that fits?** → A skill pack scaffold

If the Suitability Gate says single-agent is enough, PRDForge refuses to generate a multi-agent system — and tells you why. This is the differentiator: PRDForge is the only tool in this space that says **no**.

## Architecture (V1 plan)

```
PRD (markdown)
   │
   ▼
┌─────────────────────────────┐
│  Stage 0: Suitability Gate  │ ← fail-closed: refuses unsuitable problems
└─────────────┬───────────────┘
              │ (approved)
              ▼
┌─────────────────────────────┐
│  Stage 1: Agent Decomposer  │ ← extracts agent boundaries from PRD
└─────────────┬───────────────┘
              ▼
┌─────────────────────────────┐
│  Stage 2: Schema Synthesizer│ ← AgentVerdict contracts per agent
└─────────────┬───────────────┘
              ▼
┌─────────────────────────────┐
│  Stage 3: Harness Emitter   │ ← Claude Code skill pack output
└─────────────┬───────────────┘
              ▼
        runnable skill pack
        ready to fork-and-go
```

## Why Skill Packs (not Python)

The output of PRDForge is a **fork-and-go** skill pack — markdown agent files, a router, a schema doc — designed to run inside Claude Code with zero installation. The same files port cleanly to a Python runtime later if you need multi-tenancy or custom orchestration.

This is the same architecture that powers [GhostCheck](https://github.com/santosha86/Ghost_Check) — proven across 11 agents in production.

## Why It Matters

Suitability Gating saves engineering teams **weeks of wasted build time** on problems that should have stayed simple. The cost of building the wrong harness compounds — every new agent makes the system harder to debug, slower to iterate, and harder to reason about.

PRDForge encodes the discipline of **"single-agent until proven multi-agent"** into the build pipeline.

## Roadmap

- **V0.1** — PRD parser + Suitability Gate (in progress)
- **V0.2** — Agent decomposition with AgentVerdict schema synthesis
- **V0.3** — Harness emitter for Claude Code skill packs
- **V1.0** — End-to-end PRD → runnable skill pack
- **V1.1** — Python runtime emitter (LangGraph target)

## Status

See [STATUS.md](./STATUS.md) for the live build log.

## Author

Built by [Santosh Achanta](https://www.linkedin.com/in/santosh-achanta-ds/) — Principal AI Engineer, Saudi Arabia.

## License

MIT (planned). To be added with V1.0 release.
