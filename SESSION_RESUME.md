# SESSION_RESUME.md

> **Purpose:** Picking-up-tomorrow context for any future Claude Code (or other agent host) session working in this repo. Read this first before any task. CLAUDE.md links here from its top section.
>
> **Last session:** 2026-05-01. Latest commit: `5d3f115` (`Add SESSION_RESUME.md`). Snapshot last refreshed just before a manual `/compact`.

---

## 🔔 Most recent in-flight context (uncommitted — survives via this file only)

The chat that just got compacted included two threads after commit `5d3f115`:

### Thread A — Token-control / hook design (deferred, not implemented)
- Discussed how to keep `SESSION_RESUME.md` fresh automatically.
- **Verified via `claude-code-guide` agent that `PreCompact` is a real Claude Code hook event** (PascalCase, fires on both auto-compact and manual `/compact`, receives matcher `"manual"` or `"auto"`). Authoritative docs: [code.claude.com/docs/en/hooks.md](https://code.claude.com/docs/en/hooks.md), [hooks-guide.md](https://code.claude.com/docs/en/hooks-guide.md).
- **Important constraint:** hooks run shell commands in isolation — they cannot ask Claude to do things, only execute scripts whose stdout is appended to context.
- **Agreed design (deferred):** PreCompact-only hook (no Stop hook) → small shell script `.claude/hooks/refresh-session-resume.sh` → updates a marker block (`<!-- AUTO-SNAPSHOT-START / END -->`) in `SESSION_RESUME.md` from `git log` + `git status`. Curated handoff content above the marker stays untouched. No auto-commit.
- **Status: deferred by user** — said "we will see this later." Pick this back up only if the user asks; otherwise stay focused on V1.0 release work.

### Thread B — V1.0 smoke test (about to start when compact was requested)
- The next planned action was running `/prdforge check` against all five golden-set PRDs and comparing each verdict to the expected outcome documented in the PRD's frontmatter.
- Caveat noted: when Claude "invokes" the Stage 0 agent in the same session, it's the executor reading the skill prompt — not a fresh isolated session. Full isolated-session re-tests are recommended before the public V1.0 push, but in-session smoke test catches the major issues.
- **This is the suggested first action when resuming.** See bottom of this file.

---

## TL;DR — where we are

**V1.0 is implementation-complete.** Full V1 pipeline (Stages 0 + 1 + 5) is runnable end-to-end. Six agents shipped. Bundle format spec'd. Five golden-set PRDs cover every Stage 0 outcome.

**Remaining work before public V1.0 release tag:** smoke testing (the immediate next step), prompt tightening, launch posts, outreach. Not new agents.

**Next planned phase:** V2 = advisory architecture brief (6 more agents, Q3 2026).

**Deferred / parked:**
- PreCompact hook to auto-refresh this file (design agreed, not yet implemented — see Thread A above).

---

## Read these first (in order)

1. **[`07_PRDForge_PRD_V4.md`](./07_PRDForge_PRD_V4.md)** — the authoritative spec. V4 supersedes V3 and V2 (preserved as audit history).
2. **[`STATUS.md`](./STATUS.md)** — live build log, current phase, what's next.
3. **[`CLAUDE.md`](./CLAUDE.md)** — operating rules for working in this repo.
4. **[`docs/BUNDLE_FORMAT.md`](./docs/BUNDLE_FORMAT.md)** — on-disk contract for the V1 product output.
5. **`.claude/projects/-Users-santosh-work-Development-PRDForge/memory/MEMORY.md`** — project memory index (persistent across sessions).

If short on time, the V4 PRD §1 (Executive Summary) and §3 (Scope) are the highest-leverage pages.

---

## What this product is

**PRDForge is the validation + governance pre-flight layer for spec-driven AI development.**

It reads a PRD and produces a *validated artifact bundle* — verdict + clarifications log + requirement traceability matrix. **It does not generate code.** Output is markdown + JSON, framework-agnostic, designed to feed whatever downstream execution tool the user prefers.

Two non-negotiable disciplines (the moats):

1. **Moat 1 — Runtime Suitability Refusal (Stage 0).** PRDForge is the only tool in the category that *refuses* to proceed for problems that should be single-agent, deterministic code, or workflow automation.

2. **Moat 2 — Audit-grade traceability (Stages 1 + 5).** Source PRD is immutable. Clarifications go to an append-only `CLARIFICATIONS.md` log. Stage 5 fails closed if any requirement in `(PRD ∪ CLARIFICATIONS)` lacks coverage in downstream delivery.

---

## What's done — version-by-version log

### V0.1 (commit `9095427`) — Scaffolding + Stage 0 moat agent
- Repo structure per V3 PRD §4.5 (later updated by V4)
- `CLAUDE.md`, `docs/SCHEMAS.md`, `docs/SUITABILITY_RUBRIC.md`, `docs/CLARIFICATION_RUBRIC.md`, `docs/TOPOLOGY_RUBRIC.md`
- `@multi-agent-suitability-checker` (the first moat)
- Router `SKILL.md` (initial)
- 3 golden-set PRDs (good-fit + 2 bad-fit)

### V0.2 (commit `5fd4609`) — Stage 1 Understanding (the second moat is live)
- `@prd-parser` — PRD → ProductSpec
- `@requirement-analyzer` — R-01…R-NN with acceptance criteria
- `@clarification-auditor` — **the second moat**, writes CLARIFICATIONS.md
- `@domain-researcher` — web-search enrichment
- `clarifications-template.md`
- `examples/ambiguous-prd.md` (4–6 seeded ambiguities for testing)

### V4 repositioning (commit `e206a83`) — Validation + governance pre-flight layer
- `07_PRDForge_PRD_V4.md` (new spec, supersedes V3)
- README rewritten — new positioning, no competitor names
- STATUS.md, CLAUDE.md, router SKILL.md updated
- V2 + V3 PRDs preserved as audit history (immutable, eat your own dogfood)
- New persona priority: regulated-industry teams under EU AI Act / NIST AI RMF / ISO 42001
- Code generation explicitly out of scope at every version

### V0.3 (commit `8443c79`) — V1.0 ready
- `@architecture-reviewer` (lightweight V1 traceability gate)
- `docs/BUNDLE_FORMAT.md` — on-disk contract
- `requirements-template.md`, `traceability-template.md`
- Router `SKILL.md` with full V1 orchestration
- `examples/bad-fit-crud-app-prd.md` (NOT_AI_PROBLEM)
- Five golden-set PRDs total covering every Stage 0 outcome

---

## V1 commands — runnable today

```bash
# Stage 0 only — fast, cheap suitability verdict (~10s, ~$0.05)
/prdforge check --prd <path>

# Stages 0 + 1 — produces validated artifact bundle
/prdforge clarify --prd <path> --out <dir>
# → bundle written: PRD.md (immutable copy) + REQUIREMENTS.md +
#   CLARIFICATIONS.md + _metadata.json

# Stage 5 — validates downstream coverage, fail-closed
/prdforge trace --bundle <path> --delivery <built-system-path>
# → TRACEABILITY.md emitted; verdict PASS or FAIL with fix hints

# Inspect a bundle at a glance
/prdforge bundle --show <path>

# Placeholder for V2
/prdforge advise --bundle <path>   # not yet implemented
```

---

## What's next (V1.0 → V1.1 polish, no new agents)

These are the items left before the public V1.0 release tag:

- [ ] Smoke-test the full pipeline manually on all five golden-set PRDs (verify each Stage 0 verdict matches the expected outcome documented in the example PRD's frontmatter)
- [ ] Tighten any agent prompts that produce schema-invalid output during the smoke test
- [ ] Cut the V1.0 release tag (`v1.0.0`)
- [ ] Write the launch posts:
  - Track A — "the no" story (developer audience)
  - Track B — compliance/governance angle (regulated-industry audience)
- [ ] Outreach to 3 regulated-industry contacts for early-adopter feedback

If the user says "continue" or "proceed" tomorrow without further direction, the natural next step is the smoke test of all five golden-set PRDs.

---

## What's after V1.0 (V2 — Q3 2026)

V2 adds 6 agents producing a single markdown advisory document. **Still no code generation.**

- `@architecture-planner` — packaging, scale model, memory strategy
- `@topology-selector` — picks parallel / sequential / hierarchical / hybrid (the keystone)
- `@agent-architect` — proposes 5–11 agents with mandates
- `@schema-designer` — proposes the AgentVerdict contract
- `@orchestration-aggregator-designer` — router + aggregator pattern
- `@architecture-brief-writer` — consolidates the above into one markdown brief
- `prdforge advise` command

V3 = optional reference skill-pack implementation. Deferred indefinitely.

---

## Hard rules — do not violate

1. **Source PRD is immutable.** No agent modifies it.
2. **Stage 0 can refuse.** The pipeline must respect a `REJECT` verdict.
3. **Stage 5 fails closed.** A bundle with any unmapped requirement does not pass.
4. **Tight PRDs produce empty `CLARIFICATIONS.md`.** No tax for clarity.
5. **Output is markdown + JSON only.** Framework-agnostic. **No code generation in V1 or V2.**
6. **Agent count capped at 12 across all 6 stages.** V1 uses 6, V2 adds 6 more.
7. **Product docs never name competitor tools.** Positioning stands on its own merits. Comparative analysis is internal/strategy material only. (See memory file `feedback_competitive_landscape_first.md` for context — the user gave explicit feedback on this.)
8. **The V4 PRD is the source of truth.** When CLAUDE.md, the README, or a SKILL file disagrees with the PRD, the PRD wins.
9. **V2 and V3 PRDs are preserved untouched** as audit history of the product's evolution. This is the same immutability discipline V4 codifies for users — eat your own dogfood.

---

## User context — preferences and feedback that shaped this product

These are durable; they should guide future sessions in this repo.

### Process feedback

- **Always check the competitive landscape before scoping any new idea.** Run targeted web searches (`X comparison 2026`, `X open source alternatives`, `X github`) before agreeing to scope or design any feature. Surface the closest 3–5 competitors. Identify the gap. Then frame the user's idea as starting from that gap, not from a blank page.
- **No defensiveness when challenged.** When the user pushes back on a design choice, lead with honest assessment of the gap, not justification for past decisions.
- **Be explicit about what's been verified vs. what's being recommended from memory.** Verify before claiming a tool/feature exists.

### Product positioning preferences

- **Product stands on its own merits.** No competitor names in any user-facing artifact (README, PRDs from V4 onwards, SKILL files, STATUS.md, CLAUDE.md).
- **Independence over comparison.** Pitch what PRDForge does, not what it does better than X.
- **The V4 framing is final.** Do not pivot product positioning again without explicit user direction.

### Strategic decisions made (binding for V1)

- **Option A** chosen from a three-option strategic review: position PRDForge as the *validation + governance pre-flight layer* that fits in front of any downstream execution tool. (Alternative options were "sharpen and ship as-is" and "pivot to single-purpose Suitability Gate." Both were considered and rejected.)
- **EU AI Act timing** is the forcing function. The Act's high-risk AI provisions take effect 2026-08-02. PRDForge's outputs map to several of its Articles (esp. Articles 11, 12, 17). Frame for that timeline.
- **Code generation** is permanently out of scope. V1 and V2 outputs are documents and structured artifacts, never runnable code. V3 (optional reference skill-pack) is deferred indefinitely.
- **Agent count discipline:** cap at 12 across all 6 stages. V1 is at 6. V2 will use the remaining 6.

---

## Outstanding open questions

From V4 PRD §13:

| # | Question | Decision by |
|---|---|---|
| Q1 | Should V1 ship without `@architecture-reviewer` (Stage 5) or wait for it? | **Resolved 2026-04-30** — Stage 5 lightweight is in V1, shipped in V0.3. |
| Q2 | Should the bundle output include a JSON-only manifest for CI consumption? | **Resolved 2026-04-30** — `_metadata.json` is part of the bundle, spec'd in `docs/BUNDLE_FORMAT.md`. |
| Q3 | Should `prdforge check` accept a confidence-threshold flag for tunable strictness? | V1.0 release |
| Q4 | When/whether to ship V3 (reference skill-pack implementation) | After V2 reception |
| Q5 | Should there be a public, opt-in registry of rejected PRDs (with reasons) as a teaching corpus? | V1.1 |

---

## Files Claude should know about by name

```
07_PRDForge_PRD_V4.md            ← authoritative spec (V4)
06_PRDForge_PRD_V3.md            ← preserved (audit history)
05_PRDForge_PRD_V2.md            ← preserved (audit history)
05_PRDForge_PRD_V2.pdf           ← preserved
README.md                        ← V4-aligned product pitch
STATUS.md                        ← live build log
CLAUDE.md                        ← repo operating rules
SESSION_RESUME.md                ← this file

docs/
  SCHEMAS.md                     ← all inter-agent data contracts
  SUITABILITY_RUBRIC.md          ← Stage 0 decision tree
  CLARIFICATION_RUBRIC.md        ← Stage 1 auditor rules
  TOPOLOGY_RUBRIC.md             ← V2 keystone rubric
  BUNDLE_FORMAT.md               ← V1 product output contract

.claude/skills/
  prdforge/SKILL.md              ← router with full orchestration
  stage0-gate/multi-agent-suitability-checker.md   ← MOAT 1
  stage1-understanding/
    prd-parser.md
    requirement-analyzer.md
    clarification-auditor.md      ← MOAT 2 part 1
    domain-researcher.md
  stage5-review/
    architecture-reviewer.md      ← MOAT 2 part 2
  templates/
    clarifications-template.md
    requirements-template.md
    traceability-template.md

examples/   (5 golden-set PRDs)
  good-fit-ghostcheck-prd.md      → expects PROCEED, empty clarifications
  bad-fit-summarizer-prd.md       → expects REJECT / SINGLE_AGENT_SUFFICIENT
  bad-fit-crud-app-prd.md         → expects REJECT / NOT_AI_PROBLEM
  bad-fit-zapier-workflow-prd.md  → expects REJECT / WORKFLOW_AUTOMATION
  ambiguous-prd.md                → expects PROCEED + 4–6 clarifications
```

---

## Where the project memory lives

Persistent across sessions automatically:
`/Users/santosh_work/.claude/projects/-Users-santosh-work-Development-PRDForge/memory/`

- `MEMORY.md` — index
- `feedback_competitive_landscape_first.md` — process feedback (research market FIRST; no competitor names in product docs)
- `project_v4_repositioning.md` — V4 strategic decision context
- `project_v1_complete.md` — V1.0 implementation snapshot (added in this update)

When picking up tomorrow, the memory system loads `MEMORY.md` automatically. SESSION_RESUME.md is the explicit, version-controlled handoff that complements it.

---

## Suggested first action tomorrow

If the user says "continue" or "let's pick up where we left off":

> "Last session closed V0.3, which made V1.0 implementation-complete. The natural next step is a smoke test on the five golden-set PRDs to verify each one produces the expected verdict before we cut the V1.0 release tag. Want me to run that now?"

If the user wants to do something else, follow their direction — but always confirm against this file's "Hard rules" section before making product or scope decisions.
