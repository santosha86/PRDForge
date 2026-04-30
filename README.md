# PRDForge

**The validation + governance pre-flight for spec-driven AI development.**

A tool that reads your Product Requirements Document and answers three questions before a single line of code is generated:

1. **Should this be built at all?** — and if not, why not.
2. **Is the spec specific enough to build against?** — and if not, what's missing.
3. **What audit trail proves what was asked for, and what was delivered?**

The output is a validated artifact bundle, not generated code. PRDForge stops at "validated and audit-ready." Whatever you use to actually build the system, you bring your own.

> Status: Active development. V0.2 shipped (Stages 0 + 1 are live). V1.0 ships with V0.3 — see [STATUS.md](./STATUS.md).
> Latest spec: [PRD V4](./07_PRDForge_PRD_V4.md) · supersedes V3.

---

## Why this exists

Two failure modes dominate AI-driven software projects:

1. **Building the wrong kind of system.** Teams default to multi-agent designs for problems that should be a single well-prompted call, a script, or a workflow tool. The wrong topology choice has documented performance penalties of up to 70%.

2. **Building the right kind of system that solves the wrong problem.** When the spec is ambiguous, an AI-driven build silently picks an interpretation. By delivery, nobody can prove what was asked for, what was built, or where the two diverged. In regulated contexts, this is no longer just a quality issue — it's a compliance failure.

Most AI-development tools optimize for production speed: spec in, code out, fast. Almost none ask "should this be built at all?" or "is the spec specific enough?" before they start. **That gap is the product.**

---

## What you get

PRDForge produces three artifacts in a single run:

| Artifact | Purpose |
|---|---|
| **Suitability Verdict** | `PROCEED` or `REJECT`, with reason code and PRD-grounded rationale. If `REJECT`, an alternative recommendation. |
| **`CLARIFICATIONS.md`** (audit log) | Every ambiguity surfaced and how the user resolved it. Append-only. The source PRD is never modified. |
| **`TRACEABILITY.md`** (matrix) | Numbered requirements (`R-01…R-NN`) with acceptance criteria, ready to verify downstream output against. |

Together, these form an immutable-spec + appendable-clarifications + traceability-matrix triple — the kind of audit trail emerging AI-governance frameworks (EU AI Act, NIST AI RMF, ISO 42001) increasingly require.

---

## The two moats

These are the disciplines PRDForge enforces. They are the product, not features within it.

### Moat 1 — Runtime suitability refusal (Stage 0)

The discipline of **"single-agent until proven multi-agent"** encoded as enforceable behavior. Most decision frameworks for "should this be multi-agent?" are advisory: read a checklist and think about it. PRDForge's Stage 0 is a *runtime gate*. If the problem isn't genuinely a multi-agent problem, PRDForge refuses to proceed and returns one of five reason codes:

| Reason | When |
|---|---|
| `SINGLE_AGENT_SUFFICIENT` | One well-prompted call beats any decomposition |
| `NOT_AI_PROBLEM` | Deterministic / data-pipeline / closed-form computation |
| `WORKFLOW_AUTOMATION` | Integration glue with no real reasoning step |
| `SCOPE_TOO_VAGUE` | Spec is too thin to commit (returns clarifying questions) |
| `SCALE_MISMATCH` | Too small (one API call) or too big (full SaaS) |

### Moat 2 — Audit-grade traceability (Stages 1 + 5)

The discipline of **"don't build until you know what you're building, and prove what you built against what you asked for."**

- The source PRD is **immutable**. Never modified by any agent.
- Clarifications go to a separate, append-only `CLARIFICATIONS.md` log.
- At delivery, the Stage 5 traceability gate validates that every numbered requirement in `(PRD ∪ CLARIFICATIONS)` has explicit coverage, and emits a coverage matrix.
- **Tight PRDs produce empty `CLARIFICATIONS.md`. No tax for clarity.**

---

## Architecture

```
PRD (markdown)
   │
   ▼
┌───────────────────────────────────────────────────┐
│  Stage 0  Suitability Gate                        │ ← MOAT 1: refuses unsuitable problems
│           (@multi-agent-suitability-checker)      │
└─────────────┬─────────────────────────────────────┘
              │ PROCEED
              ▼
┌───────────────────────────────────────────────────┐
│  Stage 1  Understanding (sequential)              │
│           @prd-parser                              │
│           @requirement-analyzer (R-01…R-NN)        │
│           @clarification-auditor                   │ ← MOAT 2: writes CLARIFICATIONS.md
│           @domain-researcher                       │
└─────────────┬─────────────────────────────────────┘
              ▼
┌───────────────────────────────────────────────────┐
│  Stage 5  Traceability Gate (lightweight, V0.3)   │
│           Cross-checks coverage of every R-* in    │
│           (PRD ∪ CLARIFICATIONS). Fails closed.    │
└─────────────┬─────────────────────────────────────┘
              ▼
        Validated artifact bundle
        ./out/<slug>/
        ├─ PRD.md (immutable copy)
        ├─ CLARIFICATIONS.md
        ├─ REQUIREMENTS.md
        └─ TRACEABILITY.md
```

V2 (planned, Q3 2026) adds an **advisory architecture brief** — topology recommendation, agent roster proposal, schema design — as a single markdown report. **PRDForge does not generate runnable code at any point.** Execution stays downstream, in whatever tool the team prefers.

---

## Why now

Three convergent forces make this the right product at the right time:

- **Regulatory pressure.** The EU AI Act's high-risk AI system provisions take effect 2026-08-02. Audit-grade trails on AI-generated systems become a procurement criterion in regulated industries (medical, finance, government, automotive).
- **Spec-driven development is mainstream.** Teams treat specs as the primary artifact. But existing tooling assumes specs are correct and complete — which they rarely are without scrutiny. PRDForge fills the validation step.
- **Multi-agent over-engineering is documented.** Public research quantifies the performance cost of misapplied multi-agent designs. There is no good runtime answer in the market today; the closest things are advisory blog posts.

---

## Quickstart

```bash
# Suitability check only — fast, cheap, runs Stage 0
prdforge check --prd ideas/my-idea-prd.md
# → PROCEED or REJECT (with reason code, rationale, and alternative recommendation)

# Full pre-flight — Stages 0 + 1, produces the validated bundle
prdforge clarify --prd ideas/my-idea-prd.md --out ./out/myidea/
# → ./out/myidea/{PRD.md, CLARIFICATIONS.md, REQUIREMENTS.md}

# Traceability matrix at delivery — validates downstream output covers every R-*
prdforge trace --bundle ./out/myidea/ --delivery ./path/to/built/system/
# → TRACEABILITY.md, with per-requirement coverage status

# Inspect a generated bundle
prdforge bundle --show ./out/myidea/
```

V2 will add `prdforge advise` for the advisory architecture brief.

---

## What PRDForge does not do

- **Does not generate code.** Output is markdown + JSON. Execution is your choice.
- **Does not modify the source PRD.** All interpretations go to `CLARIFICATIONS.md`.
- **Does not guess.** Every verdict cites PRD evidence; every clarification names the requirement it relates to.
- **Does not estimate timelines.** Not the tool's mandate.
- **Does not recommend specific frameworks.** Output is framework-agnostic.

---

## The three-document audit model

| Document | Status | Mutability |
|---|---|---|
| `PRD.md` (input) | Source intent | **Immutable** — never modified |
| `CLARIFICATIONS.md` | Audit log of resolved ambiguities | Append-only during Stage 1 |
| `TRACEABILITY.md` | Coverage matrix at delivery | Replaceable per run |

This shape mirrors how mature engineering organizations already work — specs in version control, decisions in ADRs, requirements traced through delivery. PRDForge formalizes that pattern for AI-driven development specifically.

---

## Roadmap

- **V0.1** — Stage 0 suitability gate ✅
- **V0.2** — Stage 1 four agents (parser, analyzer, **clarification auditor**, researcher) ✅
- **V0.3** — Stage 5 lightweight traceability gate + CLI router → completes V1
- **V1.0** — End-to-end validated artifact bundle production
- **V2.0** — Advisory architecture brief (topology + agent roster + schema design) as a markdown report
- **V3.0** — (optional) reference skill-pack implementation, only if V1 + V2 demand it

---

## Status

See [STATUS.md](./STATUS.md) for the live build log.
Latest design: [PRD V4](./07_PRDForge_PRD_V4.md) · prior versions [V3](./06_PRDForge_PRD_V3.md), [V2](./05_PRDForge_PRD_V2.md) preserved as audit history.

---

## Author

Built by [Santosh Achanta](https://www.linkedin.com/in/santosh-achanta-ds/) — Principal AI Engineer.

---

## License

MIT (planned). To be added with V1.0 release.
