# Product Requirements Document (PRD) — V2
# PRDForge — PRD-to-Skill-Pack Meta-Framework

| Field            | Detail                                                     |
|------------------|------------------------------------------------------------|
| **Document ID**  | PF-PRD-002                                                 |
| **Version**      | 2.0 — Revised after gstack discovery                       |
| **Date**         | April 2026                                                 |
| **Author**       | Santosh Achanta                                            |
| **Status**       | Draft — Pre-build                                          |
| **Repo**         | github.com/santosh/prdforge *(to be created)*              |
| **License**      | MIT                                                        |
| **Supersedes**   | PF-PRD-001                                                 |
| **Related work** | Y Combinator's gstack (80K stars, general software factory) |

---

## Changelog V1 → V2

| Change | Why |
|---|---|
| Added §1.6 explicit positioning vs gstack | gstack exists at 80K stars; clarity prevents confusion |
| Added Stage 0: Multi-Agent Suitability Gate | The moat. Must reject unsuitable PRDs before any design |
| Collapsed agent roster from 15 to 11 | Research shows 3-5 agents optimal, 20+ underperforms |
| Narrowed product definition: "validate the idea, not ship a SaaS" | User clarification on real goal |
| Added §14.6 — Learnings Adopted from gstack | Five concrete patterns worth borrowing |
| Lowered success-metric ceilings | gstack holds the general category; PRDForge's lane is narrower |
| Strengthened §4.2 topology decision | The actual differentiator from gstack |
| Added §3.5 — What PRDForge Explicitly Does NOT Do | Ruthless scope clarity |

---

## 1. Executive Summary

### 1.1 Problem Statement

A person with an idea for a multi-agent product faces two expensive options today:

1. **Build the Python/framework version first** (LangGraph, CrewAI, AutoGen) — 3–8 weeks of engineering before they know if the idea has legs. Most ideas die here.
2. **Hand-build a Claude Code skill pack** — faster (5 days) but still requires expert architectural decisions: which topology, how many agents, what contracts, what fail-closed behavior. Getting these wrong tanks performance by up to 70% (Google / Anthropic research).

Both paths waste time because the **idea validation loop is buried under architecture decisions.** You can't know if your multi-agent concept works until you've already built half a product.

### 1.2 Product Vision

PRDForge reads a PRD and produces a **testable skill pack**. Not a finished product. Not a SaaS. A runnable Claude Code skill pack that lets you validate the multi-agent concept in one day instead of one month.

The skill pack PRDForge generates is deliberately minimal: enough to run end-to-end, produce real output, and tell you whether the idea is worth pursuing further. If yes, scale it up with your own engineering. If no, you saved four weeks.

Critical design choice: **PRDForge is itself a hybrid multi-agent system, demonstrating the exact patterns it generates.** Sequential understanding → hierarchical architecture → parallel design → parallel generation → sequential review.

### 1.3 The Unfair Differentiator: Multi-Agent Suitability Gate (Stage 0)

Before PRDForge designs anything, a gate agent answers one question:

> **"Is this problem actually a multi-agent problem?"**

If the answer is no, PRDForge refuses to generate a skill pack and explains why. This is the moat. Most "AI builder" tools happily produce multi-agent bloat for problems that should be single-agent or simple scripts. PRDForge is the one that says no.

This matters because:
- Research shows **multi-agent reduces performance by 39–70% on sequential reasoning tasks** when misapplied
- Generating a multi-agent skill pack for a non-multi-agent problem produces slower, more expensive, worse output than a single-agent version
- Users who hear "no, use a single agent" trust PRDForge more than users who get a mediocre skill pack

### 1.4 Why This Exists

Four gaps in the April-2026 landscape:

1. **Y Combinator's gstack (80K stars) is general software factory** — CEO, designer, eng manager, QA. It generates and ships working software. It does NOT design multi-agent architectures.
2. **MetaGPT outputs Python code** — not Claude Code skill packs. Heavy, slow, assumes single sequential topology.
3. **Anthropic's official skills repo** is a registry of skills, not a generator of skill packs.
4. **No existing tool rejects PRDs that shouldn't be multi-agent** — everyone defaults to "more agents = better."

PRDForge fills the intersection: **topology-aware skill pack generator that validates multi-agent suitability first.**

### 1.5 The Killer Feature

> **"Validate the idea in a day, not a month."**

You give PRDForge a PRD on Monday morning. Monday evening, you have a runnable skill pack that produces real output on real input. Tuesday you decide whether the idea is worth scaling up.

That's the value proposition, stripped of jargon. Not "AI builds software for you." Not "one-click multi-agent systems." **Idea validation loop compression.**

### 1.6 Positioning vs gstack (Read This Carefully)

| Dimension | gstack | PRDForge |
|---|---|---|
| Primary user | Technical founder shipping a product | Multi-agent builder testing an idea |
| Input | Natural-language idea ("I want to build X") | Structured PRD document |
| Output | Working SaaS / feature / code change | A testable skill pack (not a product) |
| Scope | Whole software sprint: think → code → ship | Architecture + initial skill pack |
| Topology awareness | N/A — it ships regular software | Keystone decision — rejects bad fit |
| Ideal user outcome | "I shipped a v1" | "I know if my multi-agent idea has legs" |
| When to use which | If you know you want to build X and need help shipping | If you're not sure multi-agent is even right for your problem |

**They are complementary, not competitive.** After PRDForge validates your idea, you might use gstack to scale it into a product. That's the right stack.

### 1.7 Success Definition

- Product: PRDForge generates GhostCheck from its PRD with ≥ 90% fidelity to the hand-built version.
- Product: Stage 0 correctly rejects 3 "not actually multi-agent" test PRDs.
- Signal: 1,500 GitHub stars within 90 days (narrower lane than gstack).
- Career: At least one Principal-level conversation explicitly triggered by PRDForge's existence.

---

## 2. Target Users

| Persona                          | Pain                                              | Why PRDForge Helps                                                |
|----------------------------------|---------------------------------------------------|--------------------------------------------------------------------|
| Solo multi-agent builder         | Spends weeks designing before shipping            | PRD in → testable skill pack out in ~15 minutes                   |
| ML/AI engineer exploring agents  | Unsure if a problem is even suitable for agents   | Stage 0 gate gives an honest verdict before any build             |
| Enterprise AI lead evaluating ideas | Budget doesn't support multi-week spikes       | Spin up testable skill pack in a day, decide fast                 |
| Teacher / trainer of multi-agent | Junior engineers over-engineer with LangGraph     | PRDForge demonstrates correct architecture including "no, don't"  |
| Claude Code skill pack author    | Doesn't know where to start on new product        | Fork PRDForge, run on their PRD, get working scaffolding          |

Explicit **non-users** — PRDForge is NOT for:

- Founders shipping SaaS products (use gstack)
- Teams that already know their architecture (just write the skill pack)
- Problems that aren't multi-agent in nature (PRDForge will reject them)

---

## 3. Scope

### 3.1 V1 — Core Generator with Suitability Gate (Week 1–2)

| ID      | Feature                        | Description                                                                |
|---------|--------------------------------|----------------------------------------------------------------------------|
| V1-01   | PRD Parser                     | PDF/DOCX/MD → structured `ProductSpec` JSON                               |
| V1-02   | **Multi-Agent Suitability Gate** (Stage 0) | Reject-or-proceed decision with rationale           |
| V1-03   | Requirement Analyzer           | Extract functional requirements, user stories, success criteria           |
| V1-04   | Domain Researcher              | Identify industry, failure modes, existing competitors (via web search)   |
| V1-05   | Architecture Planner           | Decide packaging, scale model, memory strategy                            |
| V1-06   | Topology Selector              | Choose parallel / sequential / hierarchical / hybrid — with evidence      |
| V1-07   | Agent Architect                | Propose 5–11 agents (not 15+) with mandates, tiers, tool needs           |
| V1-08   | Schema Designer                | Design the agent output contract for this product                         |
| V1-09   | Orchestration + Aggregator Designer (combined) | Router SKILL.md + combination logic                |
| V1-10   | Skill File Generator           | Write actual .md files for every proposed agent                           |
| V1-11   | Doc + Test Generator (combined)| README, ROADMAP, SCHEMAS.md + sample inputs                               |
| V1-12   | Architecture Reviewer          | Fail-closed validator with specific fix hints                             |
| V1-13   | CLI                            | `prdforge generate --prd X.md --out ./ghostcheck/`                         |

**Note on agent count:** V1 PRDForge has **11 agents** (down from 15 in V1 draft). Research says 3-5 is optimal, 20+ underperforms. 11 is a reasonable middle ground for a meta-framework that needs coverage across 5 stages.

### 3.2 V1.1 — Idea Validation Loop (Week 3)

| ID        | Feature                    | Description                                                           |
|-----------|----------------------------|-----------------------------------------------------------------------|
| V1.1-01   | Auto-run generated skill pack | Execute the generated pack on sample input immediately after generation |
| V1.1-02   | Validation Report          | Is the output sensible? Does the topology chosen actually fit?         |
| V1.1-03   | Idea Scorecard             | Shareable PNG: "Multi-agent fit = 8/10, Topology = parallel fan-out, First output sample" |
| V1.1-04   | Refinement Pass            | User feedback → regenerate specific stages                             |

### 3.3 V1.2 — Portfolio & Cross-Pack Consistency (Week 4)

| ID        | Feature                      | Description                                                         |
|-----------|------------------------------|---------------------------------------------------------------------|
| V1.2-01   | Portfolio Mode               | Generate multiple skill packs from related PRDs                     |
| V1.2-02   | Shared Harness Extraction    | Identify common patterns across generated packs                     |
| V1.2-03   | Cross-Pack Consistency       | Enforce shared schemas and naming                                   |
| V1.2-04   | Hermes Self-Evolution Hook   | Optional DSPy + GEPA integration for auto-tuning                   |

### 3.4 Explicitly Out of Scope (The Anti-gstack Manifesto)

| Item                              | Rationale                                             |
|-----------------------------------|-------------------------------------------------------|
| Shipping the generated code        | PRDForge stops at "testable." gstack ships.          |
| QA / browser testing               | gstack does this via `/qa`. Use gstack after PRDForge |
| Design system generation           | gstack has `/design-consultation`. Not our lane.     |
| PR review on the generated pack    | gstack has `/review`. Run it post-PRDForge.          |
| CEO-level scope challenges         | gstack has `/plan-ceo-review`. We focus on architecture, not product scope. |
| Python framework output            | MetaGPT does this. We output skill packs only.       |
| Multi-tenant hosting               | Local-first. V3 concern if ever.                     |
| Auto-deployment                    | User deploys the skill pack themselves.              |

**The discipline:** every feature request that creeps toward "build the whole product" gets redirected to gstack. PRDForge's job ends when the idea is validated.

### 3.5 What PRDForge Does NOT Do (Anti-Features)

To prevent scope creep, here are the things PRDForge explicitly refuses:

- **Will not generate** a skill pack for a single-agent problem (Stage 0 rejects it)
- **Will not generate** a skill pack for a pure CRUD or data pipeline problem
- **Will not generate** prompts without evidence from the PRD (fails closed)
- **Will not estimate** how long the final product will take — not our job
- **Will not recommend** frameworks (LangGraph, CrewAI) — skill pack only
- **Will not create** API schemas, database schemas, or frontend mockups
- **Will not optimize** prompts for cost (V1.2 optional via Hermes)

---

## 4. Architecture

PRDForge is a hybrid multi-agent system demonstrating the patterns it generates.

### 4.1 High-Level Flow (Revised)

```
PRD.md input
     │
     ▼
┌────────────────────────────────────────┐
│ STAGE 0 — Suitability Gate             │
│   @multi-agent-suitability-checker     │ ← GATE (can reject)
│   Output: VERDICT + reasoning          │
└─────────────────┬──────────────────────┘
          REJECT  │  PROCEED
            │     │
            ▼     ▼
     Explain   STAGE 1 — Understanding (Sequential)
     & exit   ┌────────────────────────────────────┐
              │  @prd-parser                        │
              │  @requirement-analyzer              │
              │  @domain-researcher                 │
              └─────────────────┬───────────────────┘
                                │
                                ▼
              STAGE 2 — Architecture (Hierarchical)
              ┌────────────────────────────────────┐
              │  @architecture-planner (supervisor)│
              │      │                             │
              │      ├─→ @topology-selector        │ ← KEYSTONE
              │      └─→ @scale-notes              │
              └─────────────────┬───────────────────┘
                                │
                                ▼
              STAGE 3 — Design (Parallel Fan-Out)
              ┌────────────────────────────────────┐
              │  @agent-architect                  │
              │  @schema-designer                  │
              │  @orchestration-aggregator-designer│
              └─────────────────┬───────────────────┘
                                │
                                ▼
              STAGE 4 — Generation (Parallel)
              ┌────────────────────────────────────┐
              │  @skill-file-generator (fires once │
              │    per agent in roster)            │
              │  @doc-test-generator               │
              └─────────────────┬───────────────────┘
                                │
                                ▼
              STAGE 5 — Review (Sequential, fail-closed)
              ┌────────────────────────────────────┐
              │  @architecture-reviewer            │
              │  (on FAIL, routes to Stage 2 or 3) │
              └─────────────────┬───────────────────┘
                                │
                                ▼
                   Complete skill pack
                   at ./out/<slug>/
```

### 4.2 The Topology Decision — The Keystone (Unchanged from V1)

This is the single most consequential agent. Research shows 81% performance boost on parallel tasks, 70% tank on sequential tasks if chosen wrong.

**Decision rubric encoded in `@topology-selector`:**

```
Given ProductSpec, determine topology:

IF tasks are independent AND evaluation is the goal
    → Parallel fan-out (blind panel)
    Examples: GhostCheck, EvalForge, LayoffRadar

IF tasks have strict linear dependencies AND order matters
    → Sequential pipeline
    Examples: BidForge (parse → extract → draft → polish)

IF task spans multiple domains requiring specialist coordination
    → Hierarchical (manager + specialists)
    Examples: ComplianceAuditor, FullStackReviewer

IF multiple patterns apply to different stages
    → Hybrid (most common in production)

IF iteration improves output
    → Circular / Maker-Checker
    Examples: ContentWriter, CodeRefactorer

IF adversarial reasoning helps
    → Debate
    Examples: RiskAssessor, ArchitectureDeciderPro
```

Output fields:
- Primary topology choice
- Secondary topologies for sub-workflows
- Rationale citing PRD evidence
- Expected token cost vs alternatives
- Expected latency profile
- Alternatives considered and rejected

### 4.3 Stage 0 — The Suitability Gate (NEW, The Moat)

This is the most important addition in V2. Before any design work, this agent answers: is the problem multi-agent at all?

**Decision tree for `@multi-agent-suitability-checker`:**

```
PROCEED TO STAGE 1 if ALL of:
  ✓ Problem has naturally decomposable sub-tasks
  ✓ Each sub-task benefits from specialized reasoning
  ✓ Either specialization or parallelization offers real value
  ✓ Coordination cost is acceptable for the use case

REJECT with reason "SINGLE_AGENT_SUFFICIENT" if:
  ✗ Problem can be solved by one well-designed prompt
  ✗ All logic is a single reasoning chain
  ✗ No independent subtask structure

REJECT with reason "NOT_AI_PROBLEM" if:
  ✗ Problem is deterministic (use regular code)
  ✗ Problem is a data pipeline (use ETL tools)
  ✗ Problem is pure CRUD

REJECT with reason "WORKFLOW_AUTOMATION" if:
  ✗ Better solved by n8n / Zapier / Make
  ✗ No reasoning, just integration glue

REJECT with reason "SCOPE_TOO_VAGUE" if:
  ✗ PRD is not specific enough to determine suitability
  (Suggest: refine PRD, come back)

REJECT with reason "SCALE_MISMATCH" if:
  ✗ Problem is too small (single API call)
  ✗ Problem is too big (needs entire SaaS, use gstack)
```

**Output of Stage 0:**

```yaml
suitability_verdict: PROCEED | REJECT
reason_code: SUITABLE | SINGLE_AGENT_SUFFICIENT | NOT_AI_PROBLEM |
             WORKFLOW_AUTOMATION | SCOPE_TOO_VAGUE | SCALE_MISMATCH
confidence: 0.0-1.0
reasoning: one paragraph, citing PRD evidence
alternative_recommendation: str  # if rejected
expected_multi_agent_benefit: str  # if proceeding
```

**Example outputs:**

```yaml
# Example 1 — good fit
suitability_verdict: PROCEED
reason_code: SUITABLE
confidence: 0.92
reasoning: >
  The PRD describes evaluating CVs across 11 independent dimensions
  (ATS score, Google presence, bucket match, channel fit, etc.).
  Each dimension requires specialized reasoning with no dependencies
  between dimensions. This is a textbook blind-panel evaluation
  problem — exactly what multi-agent parallel fan-out solves well.

# Example 2 — should reject
suitability_verdict: REJECT
reason_code: SINGLE_AGENT_SUFFICIENT
confidence: 0.85
reasoning: >
  The PRD describes "summarize this meeting transcript into action
  items." This is a single reasoning task with a well-defined input
  and output. A single well-prompted LLM call will outperform any
  multi-agent decomposition. Multi-agent would add latency and cost
  with zero quality benefit.
alternative_recommendation: >
  Write a single Claude skill file with a structured output prompt.
  No multi-agent system needed.
```

This gate is PRDForge's moat. Nothing else in the ecosystem does it.

### 4.4 Tech Stack

| Layer              | Tech                                                              |
|--------------------|-------------------------------------------------------------------|
| Runtime            | Claude Code (user's Max subscription for local dev)               |
| LLM                | Claude Sonnet 4.6 (default), extended thinking enabled where needed |
| Parsing            | MarkItDown for PRD parsing (PDF/DOCX/MD/HTML)                    |
| Web research       | Claude Code web search tool                                       |
| File ops           | Claude Code native file tools                                     |
| CLI packaging      | Bun (learning from gstack — fast install, one binary)             |
| Schema validation  | JSON Schema defs in docs; LLM-based validation at gates           |
| Output rendering   | Markdown + YAML frontmatter (same as what it generates)           |
| Shareable cards    | Anthropic frontend-design skill (V1.1)                            |

### 4.5 Repository Layout

```
prdforge/
  .claude/
    skills/
      prdforge/
        SKILL.md                    # main router /prdforge generate
      stage0-gate/
        multi-agent-suitability-checker.md    # THE MOAT
      stage1-understanding/
        prd-parser.md
        requirement-analyzer.md
        domain-researcher.md
      stage2-architecture/
        architecture-planner.md
        topology-selector.md        # THE KEYSTONE
      stage3-design/
        agent-architect.md
        schema-designer.md
        orchestration-aggregator-designer.md
      stage4-generation/
        skill-file-generator.md
        doc-test-generator.md
      stage5-review/
        architecture-reviewer.md
      templates/
        skill-pack-template/
        agent-skill-template.md
        schemas-template.md
        harness-engineering-template.md
        readme-template.md
  examples/                         # golden test PRDs
    good-fit-ghostcheck-prd.md      # should PROCEED
    good-fit-bidforge-prd.md        # should PROCEED
    bad-fit-summarizer-prd.md       # should REJECT as single-agent
    bad-fit-crud-app-prd.md         # should REJECT as not-AI
    bad-fit-zapier-workflow-prd.md  # should REJECT as workflow-automation
  out/                              # where generated skill packs go
    .gitkeep
  docs/
    SCHEMAS.md
    TOPOLOGY_RUBRIC.md              # keystone decision logic
    SUITABILITY_RUBRIC.md           # NEW — the moat doc
    WHY_NOT_GSTACK.md               # explicit positioning
    HARNESS_ENGINEERING.md
    ROADMAP.md
    ZERO_TRUST.md
  CLAUDE.md
  README.md
  LICENSE
```

### 4.6 Key Design Choices

| Choice                                         | Rationale                                                                    |
|------------------------------------------------|------------------------------------------------------------------------------|
| PRDForge is itself a hybrid multi-agent system | Dogfooding + demonstration                                                   |
| Stage 0 can reject PRDs                        | The moat. gstack would try to build anything.                               |
| Topology selector is dedicated agent           | Most consequential decision; 70% performance delta                           |
| Architecture reviewer has veto power           | Fail-closed: bad generations don't ship                                      |
| 11 agents (not 15)                             | Research: 3-5 optimal, 20+ underperforms. 11 is covered-but-not-bloated      |
| Templates are skill files, not Python          | Users fork and modify generation without Python                              |
| Stage 3+4 parallel fan-out                     | Design and generation are independent once architecture fixed                |
| Stage 5 routes failures back upstream          | Self-correcting loop (max 2 cycles to prevent infinite loop)                 |
| Generated output is testable, not finished     | PRDForge validates ideas, doesn't ship products                             |
| Bun-based CLI (learned from gstack)            | Fast install, cross-platform, one binary                                    |
| `--host` flag for cross-agent portability (learned from gstack) | Works with Claude Code, Codex, OpenCode, Cursor            |

---

## 5. Agent Catalog (11 agents across 6 stages)

All agents follow the same `AgentVerdict`-style output contract (see §6).

### 5.1 Stage 0 — Suitability Gate (1 agent)

#### 0.1 `@multi-agent-suitability-checker`
**Mandate:** Determine whether the PRD describes a genuinely multi-agent problem.
**Input:** Raw PRD (pre-parse).
**Output:** `SuitabilityVerdict` (see §4.3 output format).
**Tools:** None.
**Temperature:** 0.1 (this decision must be consistent).
**Extended thinking:** Enabled.
**Failure handling:** If uncertain (confidence < 0.6), returns `REJECT: SCOPE_TOO_VAGUE` with a list of clarifying questions.

### 5.2 Stage 1 — Understanding (3 agents, Sequential)

#### 1.1 `@prd-parser`
Convert unstructured PRD to `ProductSpec`. No tools. Fails closed on malformed input.

#### 1.2 `@requirement-analyzer`
Extract discrete functional requirements with acceptance criteria. No tools.

#### 1.3 `@domain-researcher`
Enrich with external context — competitors, failure modes, industry norms. Tools: Web search.

### 5.3 Stage 2 — Architecture (2 agents, Hierarchical)

#### 2.1 `@architecture-planner` (supervisor)
Makes keystone decisions. Delegates to `@topology-selector`. Synthesizes `ArchitecturePlan`. Extended thinking enabled.

#### 2.2 `@topology-selector` (specialist)
Pick topology with defended rationale. The keystone. Extended thinking enabled. Output: `TopologyChoice`.

**Note:** V1 PRDForge V1 merges the tier-designer, tool-classifier, and scale-analyst into `@architecture-planner` via explicit sub-decisions in its prompt. Agent count discipline (11 not 15).

### 5.4 Stage 3 — Design (3 agents, Parallel Fan-Out)

#### 3.1 `@agent-architect`
Propose 5-11 specific agents with mandates, tiers, inputs, outputs. Output: `AgentRoster`.

#### 3.2 `@schema-designer`
Design the AgentVerdict contract specific to this product. Output: `SchemaDefinitions`.

#### 3.3 `@orchestration-aggregator-designer`
Generate router SKILL.md skeleton + aggregator logic (determined by topology choice). Output: `OrchestrationSpec`.

### 5.5 Stage 4 — Generation (2 agents, Parallel Fan-Out)

#### 4.1 `@skill-file-generator`
For every agent in `AgentRoster`, write the actual skill .md file with:
- YAML frontmatter (capabilities, inputs, outputs)
- Clear prompt with examples
- Test inputs and expected outputs
**Parallelized across the roster** — one invocation per target agent.

#### 4.2 `@doc-test-generator`
Produce README, ROADMAP, SCHEMAS.md, HARNESS_ENGINEERING.md, and golden-set test fixtures.

### 5.6 Stage 5 — Review (1 agent, Sequential, Fail-Closed)

#### 5.1 `@architecture-reviewer`
Validate generated skill pack against quality gates (see §9.2). On fail, routes back to Stage 2 or 3 with specific fix hints. Maximum 2 retry cycles before final fail.

---

## 6. Data Schemas

All documented in `SCHEMAS.md`, Pydantic-ready.

### 6.1 `SuitabilityVerdict` (NEW — the gate schema)

```python
class SuitabilityVerdict(BaseModel):
    verdict: Literal["PROCEED", "REJECT"]
    reason_code: Literal[
        "SUITABLE",
        "SINGLE_AGENT_SUFFICIENT",
        "NOT_AI_PROBLEM",
        "WORKFLOW_AUTOMATION",
        "SCOPE_TOO_VAGUE",
        "SCALE_MISMATCH",
    ]
    confidence: float  # 0.0 – 1.0
    reasoning: str
    alternative_recommendation: str | None  # if rejected
    expected_multi_agent_benefit: str | None  # if proceeding
    clarifying_questions: list[str] | None  # if SCOPE_TOO_VAGUE
```

### 6.2 `ProductSpec`
(same as V1)

### 6.3 `TopologyChoice`
(same as V1)

### 6.4 `ArchitecturePlan`
(simplified from V1 — absorbs tier/tool/scale specialists into this one schema)

### 6.5 `AgentRoster`
(same as V1, but count capped at 11)

### 6.6 `ReviewReport`
(same as V1, with `retry_count` field added)

---

## 7. CLI Specification

```
prdforge check         Run Stage 0 only — is this multi-agent suitable?
prdforge generate      Full pipeline if suitable, else fails with reason
prdforge regen         Re-generate one stage after PRD updates
prdforge diff          Show what changed between two generations
prdforge explain       Walk through the architecture decisions made
prdforge validate      Run the generated pack on sample input (V1.1)
prdforge examples      List reference PRDs (good-fit + bad-fit)
```

### 7.1 Key Commands

**Check suitability only (fast, cheap):**
```bash
prdforge check --prd examples/my-idea-prd.md
# Output: PROCEED or REJECT with reasoning
# Cost: ~$0.05, runtime: ~10 seconds
# Use this before generate to save time/cost on bad-fit ideas
```

**Full generate:**
```bash
prdforge generate \
    --prd examples/ghostcheck-prd.md \
    --out ./out/ghostcheck/ \
    --explain
# Runs Stage 0 first; aborts if REJECT
```

**Validate the generated pack runs (V1.1):**
```bash
prdforge validate --pack ./out/ghostcheck/ --sample examples/sample-cv.md
# Runs the generated pack end-to-end on a sample input
# Outputs: did it produce sensible output?
```

---

## 8. Build Plan

### 8.1 V1 — Weeks 1–2

**Week 1 — Suitability gate + understanding + architecture**
| Day | Work                                                                 |
|-----|----------------------------------------------------------------------|
| 1   | Repo scaffold, CLAUDE.md, SCHEMAS.md, SUITABILITY_RUBRIC.md, TOPOLOGY_RUBRIC.md |
| 2   | **Stage 0: @multi-agent-suitability-checker** (the moat — most important) |
| 3   | Stage 1 agents: @prd-parser, @requirement-analyzer, @domain-researcher |
| 4   | Stage 2: @topology-selector (the keystone)                           |
| 5   | Stage 2: @architecture-planner + integration test. Run Stage 0→1→2 on GhostCheck PRD end-to-end |

**Week 2 — Design, generation, review**
| Day | Work                                                                 |
|-----|----------------------------------------------------------------------|
| 6   | Stage 3: @agent-architect + @schema-designer                         |
| 7   | Stage 3: @orchestration-aggregator-designer                          |
| 8   | Stage 4: @skill-file-generator                                       |
| 9   | Stage 4: @doc-test-generator                                         |
| 10  | Stage 5: @architecture-reviewer. End-to-end run generating GhostCheck from PRD. Compare to hand-built. |

### 8.2 V1.1 — Week 3 — The Validation Loop

Auto-run generated skill pack, idea scorecard, refinement pass.

### 8.3 V1.2 — Week 4 — Portfolio Mode

Generate BidForge, CodeReviewAgent, EvalForge. Public launch.

---

## 9. Non-Functional Requirements

### 9.1 Performance

| Metric                                         | Target                           |
|------------------------------------------------|----------------------------------|
| Suitability check (Stage 0 only)               | < 10 seconds                     |
| Full generation end-to-end                     | < 15 minutes                     |
| Token cost per generation                      | < $3 on Claude Sonnet 4.6       |
| Generated skill pack works first try           | ≥ 70% of the time (honest)       |
| Suitability gate accuracy on golden set        | 100% (must reject all bad-fit PRDs) |

### 9.2 Quality Gates (enforced by @architecture-reviewer)

A generated skill pack must pass ALL of:

1. Every agent has a narrow, one-sentence mandate
2. Every agent has YAML frontmatter with capabilities
3. Output schema is consistent across all agents
4. Aggregator matches topology choice
5. Router SKILL.md references the correct topology
6. Capability manifests exist for every tool-using agent
7. README has quickstart + architecture diagram
8. HARNESS_ENGINEERING.md documents the moat
9. Sample inputs exist for at least one agent
10. **The generated pack actually runs end-to-end on its sample input (V1.1)**

If any fail → block shipment, route specific fix hint back to stage. Max 2 retry cycles.

### 9.3 Privacy & Compliance

Same as GhostCheck: local-first, BYOK, zero telemetry, MIT.

---

## 10. Risks and Mitigations

| Risk                                              | Mitigation                                                       |
|---------------------------------------------------|------------------------------------------------------------------|
| Suitability gate has false positives (rejects good PRDs) | Extensive golden-set testing; confidence threshold tunable       |
| Suitability gate has false negatives (accepts bad PRDs)  | Architecture reviewer catches these in Stage 5                   |
| Generated skill packs are mediocre                 | Ship with 4 hand-curated example PRDs + golden outputs          |
| Users complain PRDForge is "too restrictive"       | The moat is the restriction — frame it as a feature             |
| gstack adds multi-agent architecture features      | PRDForge's narrower focus + topology gate keeps differentiation |
| Extended thinking costs too much                   | Cache architecture decisions; only rethink when PRD changes     |
| 70% first-try success is too low                   | V1.1 auto-run + refinement loop closes the gap                  |
| Users expect "one-click perfect product"           | README clearly frames as "validate ideas, not ship products"    |

---

## 11. Success Metrics (Revised for Narrower Lane)

### 11.1 Product Metrics (90-day)

| Metric                                          | Week 2 | Week 4 | Week 8 | Week 12 |
|-------------------------------------------------|--------|--------|--------|---------|
| GitHub stars                                    | 100    | 500    | 1,000  | 1,500   |
| Skill packs generated publicly (trackable forks)| 1      | 5      | 15     | 30      |
| Suitability verdicts delivered (including rejections) | 5 | 25   | 100    | 250     |
| Correctly-rejected PRDs (public case studies)   | 1      | 3      | 5      | 10      |

**Note:** Lower ceilings than V1 draft. gstack holds general category; PRDForge is narrower. 1,500 stars in a focused category is a strong outcome.

### 11.2 Personal Metrics

| Metric                                                      | 90-day target |
|-------------------------------------------------------------|---------------|
| Principal-level recruiter conversations triggered by PRDForge | ≥ 2         |
| Speaking/podcast invitations                                 | ≥ 1           |
| Public "how I built this" write-up                           | Yes          |
| Generated skill packs in personal portfolio                  | ≥ 4          |

### 11.3 Narrative Metrics

- At least one "PRDForge rejected my idea, and it was right" testimonial (this is the moat validated).
- At least one generated skill pack that someone else deployed publicly.
- At least one architecture decision PRDForge made that hand-building would have missed.

---

## 12. Launch Strategy

### 12.1 The Key Post (Week 3)

Not "I built a generator." That's been done.

**The key post is: "PRDForge rejected my own idea. Here's why it was right."**

Show a real PRD you wrote that you thought needed multi-agent. Show the Stage 0 rejection with its reasoning. Show that you agreed. That's the viral hook — a tool that tells its users "no."

Nobody else's AI tool tells them no. Every other tool says yes to everything. The "no" is the story.

### 12.2 Launch Sequence

```
Week 3:  "I built PRDForge. Fed it my first idea. It rejected it. 
         Here's the reasoning — and why the rejection saved me 
         3 weeks of wasted work."
         + Repo with GhostCheck regeneration demo

Week 4:  "PRDForge generated GhostCheck from its PRD. Side-by-side 
         with the hand-built version. ~90% match."

Week 5:  "PRDForge's topology selector picked HIERARCHICAL for my 
         new idea when I had assumed PARALLEL. The rationale was 
         better than mine."

Week 6:  "25 PRDs tested. Suitability gate accuracy: 92%. The 2 
         it got wrong, both my fault — ambiguous PRDs."

Week 8:  "PRDForge vs gstack — when to use which. The clearest 
         framing yet."
```

### 12.3 The Deep Technical Post (Week 4-5)

*"Why Every AI Builder Tool Should Have a 'Reject' Verdict"*

Cite the Google/Anthropic research (70% performance delta). Show the SUITABILITY_RUBRIC. Demonstrate with 3 good-fit and 3 bad-fit PRDs. This is the post that lands on Hacker News.

---

## 13. Open Questions

| #  | Question                                                              | Decision By |
|----|-----------------------------------------------------------------------|-------------|
| Q1 | Final name — PRDForge, SkillForge, SpecCraft, ArchBuilder?            | Day 1       |
| Q2 | Suitability gate: should users be able to override it with a flag?    | Week 1      |
| Q3 | How strict should quality gates be — ship loose, or fail-closed hard? | Week 1      |
| Q4 | Release Suitability Rubric as standalone doc for industry discussion? | Week 2      |
| Q5 | Should PRDForge itself generate a companion gstack-style sprint companion? | V1.2 |

---

## 14. Appendix

### 14.1 Why PRDForge Is the Right Move Now

Three convergent reasons:

1. **Claude Code skill ecosystem just crossed critical mass** — Anthropic registry live, career-ops 37K, OpenClaw 145K+, gstack 80K. Skill pack demand proven; production still artisanal.

2. **Multi-agent architecture mistakes are quantified** — Research (Google, Anthropic) shows 70% performance swings. Defensible whitespace.

3. **You have lived the full hand-build cycle once** — GhostCheck is almost complete. Building PRDForge compounds that expertise into a tool.

### 14.2 Why PRDForge Is Narrower Than gstack

gstack ships software. PRDForge validates ideas. Different stages of the pipeline, different users, different outputs. Not competitive — complementary.

### 14.3 Why the Suitability Gate Is the Real Moat

Every AI tool says yes to everything. The one that says no earns trust. And in a category where misuse tanks performance by 70%, saying no is the highest-value feature the tool has.

### 14.4 The Week 4 Narrative

*"I built a framework that generates skill packs, and the first thing it did was reject one of my own ideas. That rejection saved me three weeks. Here's the framework."*

This is a career-transforming signal that no hand-built portfolio can match.

### 14.5 What Sits Upstream and Downstream

```
IDEA → [PRDForge: is this multi-agent?]
         │
     PROCEED  │  REJECT
         │     │
         │     └→ use single-agent / regular code / workflow tool
         │
         ▼
    [PRDForge: validates idea with testable skill pack]
         │
         ▼
    Test the skill pack, iterate
         │
         │
     VALIDATED │ NOT VALIDATED
         │      │
         │      └→ move on to next idea (3 weeks saved)
         │
         ▼
    [gstack: scale into full SaaS]
```

PRDForge sits between idea and gstack. It's the checkpoint that decides which ideas deserve gstack's full treatment.

### 14.6 Learnings Adopted from gstack

Without copying, we're adopting these 5 patterns from gstack:

| gstack pattern | How PRDForge uses it |
|---|---|
| **Slash-command UX** (`/office-hours`, `/review`) | PRDForge uses same pattern: `/prdforge check`, `/prdforge generate` |
| **Bun-based installer** (fast, cross-platform) | PRDForge CLI built with Bun for 30-second install |
| **Cross-agent portability** (`--host codex`, `--host opencode`) | PRDForge supports `--host` flag; works in Claude Code, Codex, OpenCode |
| **Opt-in anonymous telemetry with Supabase** | Same model: off by default, local analytics always available |
| **Chain-of-specialists workflow** (think → plan → build → review) | PRDForge's 6-stage pipeline follows the same arc, narrower in scope |

What we are NOT copying:
- gstack's 23 skills (too broad; we stay focused on multi-agent generation)
- gstack's browser automation (not our lane)
- gstack's ship/deploy skills (PRDForge stops at testable)
- gstack's design system generation (Anthropic frontend-design skill handles this)

### 14.7 Honest Acknowledgments

This V2 PRD incorporates pushback from the author on V1:
1. gstack was missed in V1's prior-art research
2. Agent count reduced (15 → 11) based on multi-agent research
3. Stage 0 suitability gate added as the moat
4. Scope tightened to "validate ideas, not ship SaaS"
5. Star-count targets lowered to realistic levels given gstack dominates general category

---

*End of Document V2*
