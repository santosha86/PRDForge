<div align="center">

# 🔨 PRDForge

### The PRD-to-Skill-Pack Meta-Framework that says **"no"** when it should.

**Validate a multi-agent idea in a day, not a month.**

[![Status](https://img.shields.io/badge/status-pre--build-orange)](#)
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)
[![Built on](https://img.shields.io/badge/built%20on-Claude%20Code-9b59b6)](https://claude.com/claude-code)
[![Model](https://img.shields.io/badge/model-Sonnet%204.6-2ecc71)](#)
[![Topology](https://img.shields.io/badge/topology-hybrid%20multi--agent-e74c3c)](#)
[![Agents](https://img.shields.io/badge/agents-11%20across%206%20stages-f39c12)](#)

</div>

---

## 🎯 The 30-Second Pitch

You have an idea for a multi-agent product. Today you have two expensive paths:

- **Build the LangGraph/CrewAI version first** → 3–8 weeks before you know if the idea has legs.
- **Hand-build a Claude Code skill pack** → faster, but every architectural mistake costs you up to **70% performance** ([Google / Anthropic research](#)).

**PRDForge reads your PRD and emits a runnable Claude Code skill pack — but only after a gate agent decides whether your problem is actually a multi-agent problem at all.** If it isn't, PRDForge refuses to generate one and tells you why.

> **Every other AI builder tool says "yes" to everything. PRDForge is the one that says "no."**
> That's the moat.

---

## 🧠 Why This Is Different

| The crowded category | PRDForge |
|---|---|
| MetaGPT outputs Python code → heavy, slow, single-topology | Outputs a Claude Code **skill pack** → fork, run, validate |
| YC's gstack ships finished software → CEO/designer/QA agents | Stops at "**testable**" → the validation checkpoint *before* you commit to building |
| Anthropic skills repo is a registry | PRDForge is a **generator** with topology-aware design |
| Every builder tool says "yes" | **PRDForge can refuse** — Stage 0 Suitability Gate rejects single-agent / CRUD / workflow-automation problems |

PRDForge's lane: the **topology-aware skill pack generator that validates multi-agent suitability first.**

---

## 🚪 The Moat: Stage 0 — Multi-Agent Suitability Gate

Before any design work happens, one agent answers one question:

> **"Is this problem actually a multi-agent problem?"**

```yaml
# Example 1 — good fit
suitability_verdict: PROCEED
reason_code: SUITABLE
confidence: 0.92
reasoning: >
  The PRD describes evaluating CVs across 11 independent dimensions
  (ATS score, Google presence, bucket match, channel fit, etc.).
  Each dimension requires specialized reasoning with no dependencies.
  Textbook blind-panel evaluation — exactly what parallel fan-out
  multi-agent solves well.

# Example 2 — gets rejected (this is the magic)
suitability_verdict: REJECT
reason_code: SINGLE_AGENT_SUFFICIENT
confidence: 0.85
reasoning: >
  The PRD describes "summarize this meeting transcript into action
  items." Single reasoning task, well-defined input/output. A single
  well-prompted LLM call will outperform any multi-agent decomposition.
  Multi-agent would add latency and cost with zero quality benefit.
alternative_recommendation: >
  Write a single Claude skill file with a structured output prompt.
  No multi-agent system needed.
```

The gate has six possible verdicts:

| Verdict | When PRDForge refuses to build |
|---|---|
| `SUITABLE` | ✅ Proceed — genuine multi-agent decomposition |
| `SINGLE_AGENT_SUFFICIENT` | ❌ One well-prompted call beats a fleet |
| `NOT_AI_PROBLEM` | ❌ Deterministic / regular code |
| `WORKFLOW_AUTOMATION` | ❌ n8n / Zapier / Make territory |
| `SCOPE_TOO_VAGUE` | ❌ PRD too thin — returns clarifying questions |
| `SCALE_MISMATCH` | ❌ Too small (one API call) or too big (use gstack) |

**Why this matters:** Research shows multi-agent reduces performance by **39–70% on sequential reasoning** when misapplied. Saying "no" is the highest-value feature in the tool.

---

## 🏗️ Architecture — A Hybrid Multi-Agent System That Demonstrates Itself

PRDForge **is** the kind of system it generates. Six stages, four topologies, eleven agents.

```
PRD.md input
     │
     ▼
┌────────────────────────────────────────────────────┐
│ STAGE 0 — Suitability Gate          (THE MOAT)     │
│   @multi-agent-suitability-checker                 │
└─────────────────┬──────────────────────────────────┘
          REJECT  │  PROCEED
            │     │
            ▼     ▼
     Explain   STAGE 1 — Understanding         (Sequential)
     & exit   ┌────────────────────────────────────┐
              │  @prd-parser                        │
              │  @requirement-analyzer              │
              │  @domain-researcher                 │
              └─────────────────┬───────────────────┘
                                ▼
              STAGE 2 — Architecture            (Hierarchical)
              ┌────────────────────────────────────┐
              │  @architecture-planner (supervisor)│
              │      └─→ @topology-selector        │ ← KEYSTONE
              └─────────────────┬───────────────────┘
                                ▼
              STAGE 3 — Design                  (Parallel Fan-Out)
              ┌────────────────────────────────────┐
              │  @agent-architect                  │
              │  @schema-designer                  │
              │  @orchestration-aggregator-designer│
              └─────────────────┬───────────────────┘
                                ▼
              STAGE 4 — Generation              (Parallel)
              ┌────────────────────────────────────┐
              │  @skill-file-generator             │
              │  @doc-test-generator               │
              └─────────────────┬───────────────────┘
                                ▼
              STAGE 5 — Review                  (Sequential, Fail-Closed)
              ┌────────────────────────────────────┐
              │  @architecture-reviewer            │
              │  routes failures back to Stage 2/3 │
              └─────────────────┬───────────────────┘
                                ▼
                   ✅ Complete skill pack
                   at ./out/<slug>/
```

### The Keystone: `@topology-selector`

The single most consequential agent. The wrong topology choice tanks performance by 70%. So one specialist does nothing else:

```
IF tasks are independent AND evaluation is the goal
    → Parallel fan-out (blind panel)         e.g. GhostCheck

IF tasks have strict linear dependencies
    → Sequential pipeline                    e.g. BidForge

IF specialists need a coordinator
    → Hierarchical (manager + specialists)   e.g. ComplianceAuditor

IF iteration improves output
    → Circular / Maker-Checker               e.g. ContentWriter

IF adversarial reasoning helps
    → Debate                                 e.g. RiskAssessor

IF multiple patterns apply
    → Hybrid (most common in production)
```

Every choice is defended with: PRD evidence, expected token cost, latency profile, and rejected alternatives.

---

## ⚡ Quickstart

```bash
# Install
curl -fsSL https://prdforge.dev/install | bash

# Cheap, fast: just run the suitability gate (~10s, ~$0.05)
prdforge check --prd ideas/my-multi-agent-idea.md

# Full pipeline (only runs if gate says PROCEED, ~15min, <$3)
prdforge generate \
  --prd ideas/ghostcheck-prd.md \
  --out ./out/ghostcheck/ \
  --explain

# Validate the generated pack actually works (V1.1)
prdforge validate \
  --pack ./out/ghostcheck/ \
  --sample examples/sample-cv.md

# Walk through every architecture decision PRDForge made
prdforge explain --pack ./out/ghostcheck/
```

### CLI surface

| Command | Purpose |
|---|---|
| `prdforge check` | Stage 0 only — is this multi-agent suitable? |
| `prdforge generate` | Full pipeline if suitable, else fails with reason |
| `prdforge regen` | Re-generate a single stage after PRD updates |
| `prdforge diff` | Show what changed between two generations |
| `prdforge explain` | Walk through architecture decisions made |
| `prdforge validate` | Run the generated pack on sample input (V1.1) |
| `prdforge examples` | List reference PRDs (good-fit + bad-fit) |

---

## 🔬 What Makes the Output Defensible

Every generated skill pack passes ten quality gates enforced by `@architecture-reviewer` before it ships:

- ✅ Every agent has a narrow, one-sentence mandate
- ✅ Every agent has YAML frontmatter with capabilities
- ✅ Output schema is consistent across all agents
- ✅ Aggregator matches the chosen topology
- ✅ Router `SKILL.md` references the correct topology
- ✅ Capability manifests exist for every tool-using agent
- ✅ README has quickstart + architecture diagram
- ✅ `HARNESS_ENGINEERING.md` documents the moat
- ✅ Sample inputs exist for at least one agent
- ✅ The generated pack actually runs end-to-end on its sample input

If any gate fails → block shipment → route a specific fix hint upstream → max 2 retry cycles. **Fail-closed by design.**

---

## 📦 Repository Layout

```
prdforge/
├── .claude/skills/
│   ├── prdforge/                 # main router /prdforge generate
│   ├── stage0-gate/
│   │   └── multi-agent-suitability-checker.md   ← THE MOAT
│   ├── stage1-understanding/     # parser, analyzer, researcher
│   ├── stage2-architecture/
│   │   ├── architecture-planner.md
│   │   └── topology-selector.md                  ← THE KEYSTONE
│   ├── stage3-design/            # agent-architect, schema, orchestration
│   ├── stage4-generation/        # skill-file + doc-test generators
│   ├── stage5-review/            # architecture-reviewer (veto power)
│   └── templates/                # skill-pack template scaffolding
├── examples/                     # golden PRDs (good-fit + bad-fit)
│   ├── good-fit-ghostcheck-prd.md       # should PROCEED
│   ├── good-fit-bidforge-prd.md         # should PROCEED
│   ├── bad-fit-summarizer-prd.md        # should REJECT (single-agent)
│   ├── bad-fit-crud-app-prd.md          # should REJECT (not-AI)
│   └── bad-fit-zapier-workflow-prd.md   # should REJECT (workflow)
├── out/                          # generated skill packs land here
└── docs/
    ├── SCHEMAS.md
    ├── TOPOLOGY_RUBRIC.md
    ├── SUITABILITY_RUBRIC.md            ← the moat, documented
    ├── WHY_NOT_GSTACK.md
    ├── HARNESS_ENGINEERING.md
    └── ZERO_TRUST.md
```

---

## 🧰 Tech Stack

| Layer | Choice | Why |
|---|---|---|
| Runtime | Claude Code | Local-first, your Max subscription, zero infra |
| Model | Claude Sonnet 4.6 (extended thinking) | Best reasoning for keystone decisions |
| PRD parsing | MarkItDown | One library covers PDF / DOCX / MD / HTML |
| CLI | Bun | 30-second install, one binary, cross-platform |
| Cross-agent portability | `--host` flag | Works in Claude Code, Codex, OpenCode, Cursor |
| Schemas | JSON Schema + Pydantic-shaped models | Validated at every stage gate |
| Privacy | Local-first, BYOK, zero telemetry | MIT license — no lock-in, no leakage |

---

## 🎓 Design Principles (and the research behind them)

| Principle | Backed by |
|---|---|
| **A multi-agent system that says "no" earns trust** | 70% performance delta from misapplied multi-agent (Google/Anthropic) |
| **3–5 agents is optimal, 20+ underperforms** | Multi-agent research consensus — PRDForge holds at 11 across 6 stages |
| **Topology choice is the keystone, not an afterthought** | 81% boost on parallel tasks; 70% tank on sequential when wrong |
| **Fail closed at every stage gate** | Mediocre generations don't ship; reviewer has veto power |
| **The tool dogfoods its own output** | PRDForge demonstrates the patterns it generates |
| **Generated output is *testable*, not *finished*** | Idea-validation loop compression — we are not gstack |

---

## 🆚 Positioning vs YC's gstack (80K ⭐)

PRDForge is **complementary, not competitive**. Different stage of the pipeline:

| Dimension | gstack | PRDForge |
|---|---|---|
| **Primary user** | Technical founder shipping a product | Multi-agent builder testing an idea |
| **Input** | Natural-language idea | Structured PRD document |
| **Output** | Working SaaS / feature / code change | A testable skill pack (not a product) |
| **Scope** | Whole sprint: think → code → ship | Architecture + initial skill pack |
| **Topology awareness** | N/A — ships regular software | **Keystone** — rejects bad fit |
| **Ideal outcome** | "I shipped a v1" | "I know if my multi-agent idea has legs" |

```
IDEA → [PRDForge: is this multi-agent?]
         │
     PROCEED  │  REJECT
         │     │
         │     └→ single-agent / regular code / workflow tool
         │
         ▼
    [PRDForge: validates idea with testable skill pack]
         │
         ▼
    Test, iterate
         │
   VALIDATED │ NOT VALIDATED
         │      │
         │      └→ next idea (3 weeks saved)
         │
         ▼
    [gstack: scale into full SaaS]
```

PRDForge sits **between** idea and gstack. It's the checkpoint that decides which ideas deserve gstack's full treatment.

---

## 🚫 What PRDForge Explicitly Does **Not** Do

Ruthless scope discipline. Every "could it also..." gets redirected:

- ❌ Generate a skill pack for a single-agent problem (Stage 0 rejects it)
- ❌ Generate for pure CRUD or data-pipeline problems
- ❌ Ship the generated code (gstack does that)
- ❌ Browser/QA testing (gstack `/qa`)
- ❌ Design system generation (Anthropic frontend-design skill)
- ❌ Output Python framework code (MetaGPT does that)
- ❌ Multi-tenant hosting / auto-deployment
- ❌ Estimate how long the final product will take
- ❌ Recommend frameworks (LangGraph, CrewAI) — skill pack only

**The discipline:** PRDForge's job ends when the idea is validated.

---

## 🗺️ Roadmap

| Phase | Window | Highlights |
|---|---|---|
| **V1** Core Generator + Suitability Gate | Weeks 1–2 | Stage 0 moat, all 11 agents, GhostCheck regenerated end-to-end |
| **V1.1** Idea Validation Loop | Week 3 | Auto-run generated pack, idea scorecard PNG, refinement pass |
| **V1.2** Portfolio Mode | Week 4 | Multi-PRD generation, cross-pack consistency, optional DSPy/GEPA self-tuning |

### Success metrics (90-day)

| Metric | Target |
|---|---|
| GitHub stars | 1,500 |
| Skill packs generated publicly | 30 |
| Suitability verdicts delivered | 250 |
| Correctly-rejected PRDs (case studies) | 10 |
| Suitability gate accuracy on golden set | 100% |
| Generated pack works first try | ≥ 70% |

---

## 📣 The Story Behind the "No"

The launch post isn't *"I built a generator."*

It's:

> **"I built PRDForge. Fed it my own first idea. It rejected it. Here's the reasoning — and why the rejection saved me three weeks of wasted work."**

Because nobody else's AI tool tells them no. Every other tool says yes to everything. **The "no" is the story.**

---

## 📄 License & Privacy

- **License:** MIT
- **Privacy:** Local-first, BYOK, **zero telemetry**
- **Data:** Your PRDs, your generated packs, your machine. Nothing leaves.

---

## 👤 Author

**Santosh Achanta** — building tooling for the Claude Code skill ecosystem.

PRDForge is the meta-framework born from the lived experience of hand-building [GhostCheck](#) (a blind-panel CV evaluator) — compounding that expertise into a tool that decides whether your *next* multi-agent idea even deserves the build.

---

<div align="center">

### *"Validate the idea in a day, not a month."*

⭐ **Star the repo** if a tool that says "no" sounds like the missing piece.

</div>
