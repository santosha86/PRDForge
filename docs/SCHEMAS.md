# SCHEMAS.md

All data contracts produced and consumed inside the PRDForge pipeline. Pydantic-shaped — agents emit JSON/YAML matching these structures, and downstream agents read them as their input.

This document is the **single source of truth for inter-agent contracts**. If a schema and an agent prompt disagree, the schema wins.

---

## 1. `SuitabilityVerdict`

Emitted by `@multi-agent-suitability-checker` (Stage 0). The first gate.

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
    confidence: float                       # 0.0 – 1.0
    reasoning: str                          # one paragraph, must cite PRD evidence
    alternative_recommendation: str | None  # required if verdict == REJECT
    expected_multi_agent_benefit: str | None # required if verdict == PROCEED
    clarifying_questions: list[str] | None  # required if reason_code == SCOPE_TOO_VAGUE
```

**Validation rules:**
- `verdict == "PROCEED"` ⇒ `reason_code == "SUITABLE"` AND `expected_multi_agent_benefit` is non-null.
- `verdict == "REJECT"` ⇒ `reason_code != "SUITABLE"` AND `alternative_recommendation` is non-null.
- `reason_code == "SCOPE_TOO_VAGUE"` ⇒ `clarifying_questions` is non-null and non-empty.
- `confidence < 0.6` ⇒ verdict must be `REJECT` with `reason_code == "SCOPE_TOO_VAGUE"`.

---

## 2. `ProductSpec`

Emitted by `@prd-parser` (Stage 1). Structured representation of the PRD.

```python
class ProductSpec(BaseModel):
    document_id: str                        # e.g. "PF-PRD-003"
    title: str
    version: str
    objective: str                          # one-paragraph product vision
    target_users: list[Persona]
    in_scope: list[str]                     # short bullet phrases
    out_of_scope: list[str]
    constraints: list[str]                  # tech, compliance, perf
    success_definition: list[str]           # measurable outcomes
    raw_sections: dict[str, str]            # section title → markdown content

class Persona(BaseModel):
    name: str
    pain: str
    why_it_helps: str
```

---

## 3. `RequirementSet`

Emitted by `@requirement-analyzer` (Stage 1). Numbered requirements with acceptance criteria.

```python
class RequirementSet(BaseModel):
    requirements: list[Requirement]

class Requirement(BaseModel):
    id: str                                 # "R-01", "R-02", ...
    title: str                              # short label
    description: str                        # what it does
    acceptance_criteria: list[str]          # testable conditions
    priority: Literal["MUST", "SHOULD", "COULD"]
    source_section: str | None              # which PRD section it came from
```

**Stability:** IDs are stable for the duration of a single `prdforge` run. Do not renumber across stages.

---

## 4. `ClarificationLog`

Emitted by `@clarification-auditor` (Stage 1, NEW IN V3). Audit log of how PRD ambiguities were resolved. Written to `CLARIFICATIONS.md` alongside the generated pack.

```python
class ClarificationLog(BaseModel):
    prd_path: str
    generated_at: datetime                  # ISO 8601
    auditor_verdict: Literal[
        "NO_CLARIFICATIONS_NEEDED",
        "CLARIFICATIONS_RESOLVED",
        "CLARIFICATIONS_PARTIALLY_SKIPPED",
    ]
    entries: list[ClarificationEntry]

class ClarificationEntry(BaseModel):
    clarification_id: str                   # "C-01", "C-02", ...
    raised_by: str                          # "@clarification-auditor"
    timestamp: datetime
    relates_to: str                         # "R-07" or "OBJECTIVE"
    ambiguity_type: Literal[
        "MISSING_DETAIL",                   # e.g. "fast" with no latency budget
        "CONTRADICTION",                    # two requirements conflict
        "UNDEFINED_TERM",                   # key term not defined
        "GAP",                              # behavior implied not specified
    ]
    question: str                           # what the auditor asked
    user_answer: str | None                 # what the user replied
    user_skipped: bool                      # true if user declined to answer
    resolution_kind: Literal[
        "DISAMBIGUATION",                   # clarifies existing requirement
        "EXTENSION",                        # extends scope of existing req
        "NEW_REQUIREMENT",                  # surfaces a new requirement
    ] | None
    new_requirement_id: str | None          # if resolution_kind == NEW_REQUIREMENT
```

**Validation rules:**
- `auditor_verdict == "NO_CLARIFICATIONS_NEEDED"` ⇒ `entries == []`.
- `user_skipped == True` ⇒ `user_answer == None` AND `resolution_kind == None`.
- `resolution_kind == "NEW_REQUIREMENT"` ⇒ `new_requirement_id` is non-null and matches a `R-*` ID in the analyzer's output (the analyzer is re-run after clarifications are resolved).
- The source PRD file at `prd_path` is **never modified** as a side-effect of this log.

---

## 5. `TopologyChoice`

Emitted by `@topology-selector` (Stage 2). The keystone decision.

```python
class TopologyChoice(BaseModel):
    primary_topology: Literal[
        "PARALLEL_FAN_OUT",
        "SEQUENTIAL_PIPELINE",
        "HIERARCHICAL",
        "HYBRID",
        "CIRCULAR_MAKER_CHECKER",
        "DEBATE",
    ]
    secondary_topologies: list[str]         # for sub-workflows, if any
    rationale: str                          # must cite PRD evidence
    expected_token_cost: str                # qualitative ("low" | "medium" | "high") + reasoning
    expected_latency_profile: str           # qualitative + reasoning
    alternatives_considered: list[AlternativeTopology]

class AlternativeTopology(BaseModel):
    topology: str
    why_rejected: str
```

---

## 6. `ArchitecturePlan`

Emitted by `@architecture-planner` (Stage 2). Synthesizes the architecture decisions.

```python
class ArchitecturePlan(BaseModel):
    topology: TopologyChoice
    packaging: Literal["SKILL_PACK", "PYTHON", "BOTH"]   # V1: always SKILL_PACK
    scale_model: str                        # qualitative description
    memory_strategy: str                    # stateless | session | persistent
    tier_strategy: str                      # which agents need extended thinking
    tool_strategy: str                      # which agents need tools (web, file ops)
```

---

## 7. `AgentRoster`

Emitted by `@agent-architect` (Stage 3). The cast of agents the generated skill pack will include.

```python
class AgentRoster(BaseModel):
    agents: list[ProposedAgent]

class ProposedAgent(BaseModel):
    name: str                               # "@cv-ats-scorer"
    mandate: str                            # one sentence
    tier: Literal["SONNET", "OPUS", "HAIKU"]
    inputs: list[str]                       # field names from the agent's input schema
    outputs: list[str]                      # field names from AgentVerdict
    tools: list[str]                        # ["web_search"] | ["file_read"] | []
    extended_thinking: bool
    rationale: str                          # why this agent is needed
```

**Constraints:**
- `len(agents)` must be in `[5, 11]` per V3 §3.1.
- Every agent's mandate must be a single sentence.

---

## 8. `SchemaDefinitions`

Emitted by `@schema-designer` (Stage 3). The AgentVerdict contract for the *generated* skill pack.

```python
class SchemaDefinitions(BaseModel):
    agent_verdict_fields: list[VerdictField]
    aggregator_input_shape: str             # how aggregator consumes verdicts
    aggregator_output_shape: str            # final user-facing output

class VerdictField(BaseModel):
    name: str
    type: str                               # "str" | "float" | "list[str]" | ...
    required: bool
    description: str
```

---

## 9. `OrchestrationSpec`

Emitted by `@orchestration-aggregator-designer` (Stage 3). Router + aggregator behavior, derived from `TopologyChoice`.

```python
class OrchestrationSpec(BaseModel):
    router_skill_md: str                    # full text of router SKILL.md
    aggregator_logic: str                   # pseudo-code or markdown describing combination
    failure_modes: list[str]                # what happens when an agent fails
```

---

## 10. `ReviewReport`

Emitted by `@architecture-reviewer` (Stage 5). The fail-closed verdict.

```python
class ReviewReport(BaseModel):
    verdict: Literal["PASS", "FAIL"]
    failed_gates: list[str]                 # references to gate IDs in §9.2 of PRD
    fix_hints: list[str]                    # specific actions per failed gate
    retry_count: int                        # 0..2 (max 2 retries before final fail)
    traceability_matrix: list[RequirementCoverage]   # NEW IN V3

class RequirementCoverage(BaseModel):
    requirement_id: str                     # "R-07"
    requirement_text: str
    source: Literal["PRD", "CLARIFICATION"]
    covered_by: list[str]                   # agent names or feature paths
    coverage_status: Literal["FULL", "PARTIAL", "NONE"]
    notes: str | None
```

**Fail conditions (any of these → `verdict == "FAIL"`):**
- Any `coverage_status == "NONE"` in the traceability matrix.
- Any of the 11 quality gates in PRD §9.2 fails.
- `retry_count >= 2` AND any failed gates remain.

---

## 11. Type aliases used across schemas

```python
class Tier(str, Enum):
    SONNET = "SONNET"   # default
    OPUS = "OPUS"       # extended thinking, keystone decisions
    HAIKU = "HAIKU"     # cheap deterministic tasks

class CoverageStatus(str, Enum):
    FULL = "FULL"
    PARTIAL = "PARTIAL"
    NONE = "NONE"
```

---

## Versioning

- This document is versioned alongside the PRD. V3 adds `ClarificationLog` and `RequirementCoverage`.
- Schema changes that break consumer agents are **breaking changes** and require a PRD bump.
- Adding optional fields is non-breaking.
