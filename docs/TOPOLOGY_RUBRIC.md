# TOPOLOGY_RUBRIC.md

The decision tree `@topology-selector` (Stage 2) follows when picking the multi-agent topology for a generated skill pack.

This is **the keystone decision** — not the moat, but the highest-stakes architectural choice inside the pipeline. Research shows the wrong topology costs up to **70% on sequential tasks** and forfeits up to **81% gain** that the right topology would have delivered on parallel ones.

---

## The six topologies

```
PARALLEL_FAN_OUT          N specialists run independently, an aggregator combines results.
SEQUENTIAL_PIPELINE       Output of stage K is input of stage K+1. Linear, ordered.
HIERARCHICAL              A supervisor agent delegates to specialists, synthesizes.
HYBRID                    Different stages of the workflow use different topologies.
CIRCULAR_MAKER_CHECKER    Iterative refinement loop: generate → critique → revise.
DEBATE                    Two or more agents argue opposing positions; a judge resolves.
```

---

## The decision tree

```
IF tasks are independent AND evaluation is the goal
    → PARALLEL_FAN_OUT (blind panel)
    Examples: GhostCheck (CV evaluation), EvalForge (model eval), LayoffRadar

IF tasks have strict linear dependencies AND order matters
    → SEQUENTIAL_PIPELINE
    Examples: BidForge (parse → extract → draft → polish)

IF the task spans multiple domains requiring specialist coordination
    → HIERARCHICAL (manager + specialists)
    Examples: ComplianceAuditor, FullStackReviewer

IF iteration improves output quality
    → CIRCULAR_MAKER_CHECKER
    Examples: ContentWriter, CodeRefactorer

IF adversarial reasoning helps (one position vs. another)
    → DEBATE
    Examples: RiskAssessor, ArchitectureDeciderPro

IF multiple of the above apply to different stages of the workflow
    → HYBRID (most common in production)
```

---

## Output requirements

`@topology-selector` must emit a `TopologyChoice` (see `SCHEMAS.md` §5) with all fields populated:

- `primary_topology` — one of the six.
- `secondary_topologies` — for hybrid layouts; describe which sub-workflow uses which.
- `rationale` — must cite specific PRD evidence. No generic justifications.
- `expected_token_cost` — qualitative ("low" | "medium" | "high") with reasoning grounded in agent count and tier strategy.
- `expected_latency_profile` — qualitative with reasoning. Parallel ⇒ low wall time, sequential ⇒ high.
- `alternatives_considered` — at least two, each with a `why_rejected` paragraph.

A `TopologyChoice` without rejected alternatives is invalid. The choice must be defended against the runner-up.

---

## Common failure modes (and how to avoid them)

| Failure mode | Why it happens | How the rubric prevents it |
|---|---|---|
| Defaulting to PARALLEL because it's "modern" | Parallel is fashionable; the agent picks it without checking dependencies. | Force the agent to enumerate cross-task dependencies. If any exist, parallel is wrong. |
| Picking SEQUENTIAL because the PRD reads top-down | Authors describe products in order; that doesn't mean they execute in order. | Distinguish documentation order from runtime dependency. Ask: would the output of step 2 actually feed step 3? |
| Reflexively going HIERARCHICAL | Hierarchy "feels safe" because there's a boss agent. | Hierarchy requires *coordination* of *different domains*. If the specialists are doing the same kind of work, parallel + aggregator is simpler. |
| Picking HYBRID to avoid commitment | Hybrid hides the absence of a real decision. | Require the agent to name which topology applies to which stage. If it can't decompose, hybrid is masking confusion. |

---

## Worked examples

### Example 1 — PARALLEL_FAN_OUT

**Product:** GhostCheck — CV blind-panel evaluator across 11 dimensions.

**Choice:**
```yaml
primary_topology: PARALLEL_FAN_OUT
rationale: >
  The PRD specifies "11 independent dimensions, each scored 0-100" with no
  cross-dimension dependency. Each dimension benefits from a specialized prompt.
  Parallel execution compresses wall time ~10x vs. sequential. Aggregation is
  a simple weighted sum.
expected_token_cost: medium  # 11 agents × ~3K tokens each
expected_latency_profile: low  # parallel; gated by slowest specialist
alternatives_considered:
  - topology: SEQUENTIAL_PIPELINE
    why_rejected: >
      No dependency between dimensions. Sequential would multiply latency
      by 11x with zero quality gain.
  - topology: HIERARCHICAL
    why_rejected: >
      Specialists are doing isomorphic work (score-and-defend). A supervisor
      adds coordination cost without coordination value.
```

### Example 2 — SEQUENTIAL_PIPELINE

**Product:** BidForge — converts an RFP into a proposal draft.

**Choice:**
```yaml
primary_topology: SEQUENTIAL_PIPELINE
rationale: >
  The PRD describes "parse RFP → extract requirements → draft response →
  polish for tone." Each stage strictly depends on the prior stage's
  structured output. Parallel execution is impossible (you cannot draft
  what you have not yet extracted).
expected_token_cost: medium
expected_latency_profile: high  # sum of all stages
alternatives_considered:
  - topology: PARALLEL_FAN_OUT
    why_rejected: >
      Linear dependency chain: parsing must precede extraction must precede
      drafting. No room for parallelism at the stage level.
  - topology: CIRCULAR_MAKER_CHECKER
    why_rejected: >
      The PRD does not call for iterative refinement; the polish stage is
      single-pass.
```

### Example 3 — HYBRID

**Product:** A document review system that fans out to 5 specialists then synthesizes via supervisor.

**Choice:**
```yaml
primary_topology: HYBRID
secondary_topologies:
  - "Stage A (review): PARALLEL_FAN_OUT across 5 specialists"
  - "Stage B (synthesis): HIERARCHICAL — supervisor reads all 5 and writes the verdict"
rationale: >
  Specialists evaluate independent dimensions (parallel-suitable) but the
  final verdict requires cross-cutting synthesis a single supervisor must
  perform after all parallel verdicts return.
```

---

## When to update this rubric

This rubric directly drives the keystone decision in every generated skill pack. Updates must be done carefully:

1. Add or refine a topology rule.
2. Re-run `@topology-selector` against the golden-set example PRDs and confirm prior choices remain stable (or document the intentional delta).
3. Bump the version footer below.
4. Note observable behavior changes in the PRD changelog.

---

**Rubric version:** 1.0 (V3 release — initial)
**Last reviewed:** April 2026
**Status:** Skeleton; will be expanded as Stage 2 agents come online (V0.3).
