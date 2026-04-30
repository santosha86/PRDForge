# PRDForge — Status

**Last updated:** April 2026
**Phase:** V0.2 complete · V0.3 in progress (closes out V1)
**Latest design:** [PRD V4](./07_PRDForge_PRD_V4.md) — repositions PRDForge as the validation + governance pre-flight layer for spec-driven AI development

## What's working today

- Repo scaffolded per PRD V4 §4.5
- `CLAUDE.md`, `docs/SCHEMAS.md`, `docs/SUITABILITY_RUBRIC.md`, `docs/CLARIFICATION_RUBRIC.md`, `docs/TOPOLOGY_RUBRIC.md`
- **Stage 0** — `@multi-agent-suitability-checker` (Moat 1: runtime refusal)
- **Stage 1** — all four agents:
  - `@prd-parser` (PRD → ProductSpec)
  - `@requirement-analyzer` (R-01…R-NN with acceptance criteria)
  - `@clarification-auditor` (Moat 2: writes `CLARIFICATIONS.md`)
  - `@domain-researcher` (web-search enrichment)
- Router `SKILL.md` with command surface (`check`, `clarify`, `generate`, `trace`, `validate`, `explain`)
- Golden-set test PRDs:
  - `examples/good-fit-ghostcheck-prd.md` (should PROCEED with empty clarifications)
  - `examples/bad-fit-summarizer-prd.md` (should REJECT, single-agent)
  - `examples/bad-fit-zapier-workflow-prd.md` (should REJECT, workflow-automation)
  - `examples/ambiguous-prd.md` (should PROCEED but raise 4–6 clarifications)
- `CLARIFICATIONS.md` on-disk template at `.claude/skills/templates/clarifications-template.md`

## What's next (V0.3 — closes out V1)

Stage 5 lightweight traceability gate + CLI plumbing:

- [ ] `@architecture-reviewer` (lightweight V1 version) — traceability gate only; full review is V2
- [ ] CLI router with `check` / `clarify` / `trace` / `bundle` commands wired end-to-end
- [ ] Output bundle structure: `./out/<slug>/` containing `PRD.md` (immutable copy) + `CLARIFICATIONS.md` + `REQUIREMENTS.md` + `TRACEABILITY.md`
- [ ] End-to-end run on all four golden-set PRDs producing complete artifact bundles
- [ ] V1.0 announcement post

## What's after V1.0 (V2 — Advisory Architecture Brief)

V2 adds 6 agents producing a single markdown advisory document. **No code generation.**

- [ ] `@architecture-planner` — packaging, scale model, memory strategy
- [ ] `@topology-selector` — picks parallel / sequential / hierarchical / hybrid with defended rationale (the keystone)
- [ ] `@agent-architect` — proposes 5–11 agents with mandates and tiers
- [ ] `@schema-designer` — proposes the AgentVerdict contract
- [ ] `@orchestration-aggregator-designer` — router + aggregator pattern
- [ ] `@architecture-brief-writer` — consolidates the above into one markdown brief
- [ ] `prdforge advise` command

## What's deferred (V3 — optional)

A reference Claude Code skill-pack implementation, only if V1 + V2 reception demands it. The product proposition does not depend on this.

## Honest scope

PRDForge is a **personal R&D project** I'm building in the open.

It exists because I've watched enough multi-agent projects collapse under their own weight to want a tool that says "you don't need this complexity" before the engineering starts — and watched enough teams ship the wrong thing because the spec was ambiguous and nobody noticed.

The two disciplines PRDForge encodes:

1. **"Single-agent until proven multi-agent"** — Moat 1, the Suitability Gate. A runtime refusal, not advisory guidance. Validated across the multi-agent systems I've built (each passed the bar).
2. **"Don't build until you know what you're building, and prove what you built against what you asked for"** — Moat 2, the Clarification Auditor + Traceability Gate. Live as of V0.2; traceability gate ships with V0.3.

The V4 repositioning (April 2026) sharpened the product from "PRD-to-skill-pack generator" to "validation + governance pre-flight layer." This narrowed V1 scope, made the audit-trail discipline central rather than a feature, and aligned the output (validated artifact bundle) with emerging AI governance frameworks (EU AI Act, NIST AI RMF, ISO 42001).

## Try it today

```bash
# Suitability check only — fast, cheap (Stage 0 only)
/prdforge check --prd examples/bad-fit-summarizer-prd.md
# → REJECT, SINGLE_AGENT_SUFFICIENT

# Full pre-flight — Stages 0 + 1
/prdforge clarify --prd examples/ambiguous-prd.md --out ./out/supportai/
# → 4–6 clarifications surfaced; user answers in one batch; CLARIFICATIONS.md written
```

## Get notified when V1 ships

Watch this repo, or DM me on [LinkedIn](https://www.linkedin.com/in/santosh-achanta-ds/).
