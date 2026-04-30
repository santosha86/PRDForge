# PRDForge — Status

**Last updated:** April 2026
**Phase:** V1.0 ready (V0.3 just shipped — full V1 pipeline runnable end-to-end)
**Latest design:** [PRD V4](./07_PRDForge_PRD_V4.md) — repositions PRDForge as the validation + governance pre-flight layer for spec-driven AI development

## What's working today

The full V1 pipeline is now implemented:

- **Stage 0** — `@multi-agent-suitability-checker` (Moat 1: runtime refusal)
- **Stage 1** — all four agents:
  - `@prd-parser` (PRD → ProductSpec)
  - `@requirement-analyzer` (R-01…R-NN with acceptance criteria)
  - `@clarification-auditor` (Moat 2 part 1: writes `CLARIFICATIONS.md`)
  - `@domain-researcher` (web-search enrichment)
- **Stage 5** — `@architecture-reviewer` (Moat 2 part 2: lightweight V1 traceability gate, fail-closed)
- **Router** `SKILL.md` with complete orchestration for all four V1 commands
- **Bundle format** spec at `docs/BUNDLE_FORMAT.md`
- **Output templates** for `CLARIFICATIONS.md`, `REQUIREMENTS.md`, `TRACEABILITY.md`
- **Schemas + rubrics** at `docs/SCHEMAS.md`, `SUITABILITY_RUBRIC.md`, `CLARIFICATION_RUBRIC.md`, `TOPOLOGY_RUBRIC.md`

Golden-set test PRDs (one per scenario):
- `examples/good-fit-ghostcheck-prd.md` (should PROCEED, empty clarifications)
- `examples/bad-fit-summarizer-prd.md` (should REJECT, `SINGLE_AGENT_SUFFICIENT`)
- `examples/bad-fit-crud-app-prd.md` (should REJECT, `NOT_AI_PROBLEM`)
- `examples/bad-fit-zapier-workflow-prd.md` (should REJECT, `WORKFLOW_AUTOMATION`)
- `examples/ambiguous-prd.md` (should PROCEED but raise 4–6 clarifications)

## V1 commands — runnable today

```bash
# Stage 0 only — fast, cheap suitability verdict
/prdforge check --prd examples/bad-fit-summarizer-prd.md
# → REJECT, SINGLE_AGENT_SUFFICIENT

# Stages 0 + 1 — produces validated artifact bundle
/prdforge clarify --prd examples/ambiguous-prd.md --out ./out/supportai/
# → 4–6 clarifications surfaced; user answers in one batch
# → bundle written: PRD.md + REQUIREMENTS.md + CLARIFICATIONS.md + _metadata.json

# Stage 5 — validates downstream coverage, fail-closed
/prdforge trace --bundle ./out/supportai/ --delivery ./path/to/built/system/
# → TRACEABILITY.md emitted; verdict PASS or FAIL with fix hints

# Inspect a bundle at a glance
/prdforge bundle --show ./out/supportai/
```

## What's next (V1.0 → V1.1)

Polish + adoption work, not new agents:

- [ ] Run all five golden-set PRDs end-to-end through the actual pipeline (manual smoke test)
- [ ] Tighten any agent prompts that produce schema-invalid output
- [ ] Cut the V1.0 release tag
- [ ] Write the launch posts (Track A — "the no" story; Track B — compliance/governance angle)
- [ ] Outreach to 3 regulated-industry contacts for early-adopter feedback

## What's after V1.0 (V2 — Advisory Architecture Brief, Q3 2026)

V2 adds 6 agents producing a single markdown advisory document. **Still no code generation.**

- [ ] `@architecture-planner` — packaging, scale model, memory strategy
- [ ] `@topology-selector` — picks parallel / sequential / hierarchical / hybrid with defended rationale (the keystone)
- [ ] `@agent-architect` — proposes 5–11 agents with mandates and tiers
- [ ] `@schema-designer` — proposes the AgentVerdict contract
- [ ] `@orchestration-aggregator-designer` — router + aggregator pattern
- [ ] `@architecture-brief-writer` — consolidates the above into one markdown brief
- [ ] `prdforge advise` command

## What's deferred (V3 — optional)

A reference implementation of a downstream skill pack, only if V1 + V2 reception demands it. The product proposition does not depend on this.

## Honest scope

PRDForge is a **personal R&D project** I'm building in the open.

It exists because I've watched enough multi-agent projects collapse under their own weight to want a tool that says "you don't need this complexity" before the engineering starts — and watched enough teams ship the wrong thing because the spec was ambiguous and nobody noticed.

The two disciplines PRDForge encodes:

1. **"Single-agent until proven multi-agent"** — Moat 1, the Suitability Gate. A runtime refusal, not advisory guidance.
2. **"Don't build until you know what you're building, and prove what you built against what you asked for"** — Moat 2, the Clarification Auditor + Traceability Gate. **Both halves now live as of V0.3.**

The V4 repositioning (April 2026) sharpened the product from "PRD-to-skill-pack generator" to "validation + governance pre-flight layer." This narrowed V1 scope, made the audit-trail discipline central rather than a feature, and aligned the output (validated artifact bundle) with emerging AI governance frameworks.

## Get notified when V1 ships publicly

Watch this repo, or DM me on [LinkedIn](https://www.linkedin.com/in/santosh-achanta-ds/).
