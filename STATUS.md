# PRDForge — Status

**Last updated:** April 2026
**Phase:** V0.2 — Stage 1 Understanding complete (the second moat is live)
**Latest design:** [PRD V3](./06_PRDForge_PRD_V3.md) — adds Clarification Audit Trail + Traceability Gate

## What's working today

- Repo scaffolded per PRD V3 §4.5
- `CLAUDE.md`, `docs/SCHEMAS.md`, `docs/SUITABILITY_RUBRIC.md`, `docs/CLARIFICATION_RUBRIC.md`, `docs/TOPOLOGY_RUBRIC.md`
- **Stage 0** — `@multi-agent-suitability-checker` (the first moat — refuses bad-fit PRDs)
- **Stage 1** — all four agents:
  - `@prd-parser` (PRD → ProductSpec)
  - `@requirement-analyzer` (R-01…R-N with acceptance criteria)
  - `@clarification-auditor` (the second moat — writes `CLARIFICATIONS.md`)
  - `@domain-researcher` (web-search enrichment for Stage 2)
- Router `SKILL.md` with command surface (`check`, `clarify`, `generate`, `trace`, `validate`, `explain`)
- Golden-set test PRDs:
  - `examples/good-fit-ghostcheck-prd.md` (should PROCEED with empty clarifications)
  - `examples/bad-fit-summarizer-prd.md` (should REJECT, single-agent)
  - `examples/bad-fit-zapier-workflow-prd.md` (should REJECT, workflow-automation)
  - `examples/ambiguous-prd.md` (should PROCEED but raise 4–6 clarifications)
- `CLARIFICATIONS.md` on-disk template at `.claude/skills/templates/clarifications-template.md`

## What's next (V0.2 → V0.3)

Stage 2 — Architecture (hierarchical):

- [ ] `@architecture-planner` (supervisor) — packaging, scale model, memory strategy
- [ ] `@topology-selector` (THE KEYSTONE) — picks parallel / sequential / hierarchical / hybrid with defended rationale
- [ ] End-to-end Stage 0 → 1 → 2 dry run on GhostCheck PRD; compare topology choice to hand-built reality

## What's after V0.3

- **V0.4** — Stage 3 (Design): `@agent-architect`, `@schema-designer`, `@orchestration-aggregator-designer`
- **V0.5** — Stage 4 (Generation): `@skill-file-generator`, `@doc-test-generator`
- **V0.6** — Stage 5 (Review + Traceability Gate): `@architecture-reviewer` with the requirement-coverage check
- **V1.0** — End-to-end PRD → runnable skill pack + `CLARIFICATIONS.md` + traceability matrix
- **V1.1** — Auto-run validation loop on the generated pack

## Honest scope

PRDForge is a **personal R&D project** I'm building in the open.

It exists because I've watched enough multi-agent projects collapse under their own weight to want a tool that says "you don't need this complexity" before the engineering starts — and watched enough teams ship the wrong thing because the spec was ambiguous and nobody noticed.

The two disciplines PRDForge encodes:

1. **"Single-agent until proven multi-agent"** — the Suitability Gate. Validated across the multi-agent systems I've built (GhostCheck, Fleet Dispatch AI, AI Interview Platform, ContentOS). Each passed the bar.
2. **"Don't build until you know what you're building, and prove what you built against what you asked for"** — the Clarification Auditor + Traceability Gate. **Now live in V0.2** — auditor in place, traceability gate ships with V0.6.

## Try it today

After V0.2 you can run two commands end-to-end:

```bash
# Cheap, fast — Stage 0 only
/prdforge check --prd examples/bad-fit-summarizer-prd.md
# → REJECT, SINGLE_AGENT_SUFFICIENT

# Full Stage 0 + Stage 1 — produces CLARIFICATIONS.md
/prdforge clarify --prd examples/ambiguous-prd.md --out ./out/supportai/
# → 4–6 clarifications surfaced; user answers in one batch; audit log written
```

## Get notified when V1 ships

Watch this repo, or DM me on [LinkedIn](https://www.linkedin.com/in/santosh-achanta-ds/).
