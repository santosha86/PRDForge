---
name: prdforge
description: >
  Router for PRDForge — the validation + governance pre-flight layer for
  spec-driven AI development. Coordinates the V1 pipeline (Stages 0, 1, 5)
  and exposes user commands: check (Stage 0 only), clarify (Stages 0+1
  with bundle output), trace (Stage 5 traceability matrix), bundle
  (inspect a generated bundle). V2 will add advise (architecture brief).
commands:
  - /prdforge check
  - /prdforge clarify
  - /prdforge trace
  - /prdforge bundle
  - /prdforge advise   # (V2)
authoritative_spec: 07_PRDForge_PRD_V4.md
authoritative_bundle_format: docs/BUNDLE_FORMAT.md
---

# /prdforge — Router

PRDForge reads a PRD and produces a validated artifact bundle: a suitability verdict, an audit log of resolved ambiguities, and a requirement traceability matrix. **It does not generate code.** Output is markdown + JSON, framework-agnostic. The bundle feeds whatever downstream execution tool the user prefers.

PRDForge enforces two disciplines:

1. **Runtime Suitability Refusal (Moat 1, Stage 0)** — refuses to proceed for problems that should be single-agent, deterministic code, or workflow automation.
2. **Audit-grade traceability (Moat 2, Stages 1 + 5)** — produces an immutable PRD copy + appendable `CLARIFICATIONS.md` log + requirement coverage matrix.

The PRD V4 is the source of truth: [`07_PRDForge_PRD_V4.md`](../../../07_PRDForge_PRD_V4.md). The bundle on-disk contract is in [`docs/BUNDLE_FORMAT.md`](../../../docs/BUNDLE_FORMAT.md).

---

## How to invoke this router

When a user types `/prdforge <command> <args>`, parse the command and dispatch to the orchestration block below. Every command's orchestration is explicit — read the steps and execute them in order.

### `/prdforge check --prd <path>`

**Stage 0 only.** No bundle produced.

```
1. Read <path> (the PRD file). Verify it exists and is readable.
2. Invoke @multi-agent-suitability-checker with the PRD's raw markdown as input.
3. Receive a SuitabilityVerdict JSON object.
4. Validate it against docs/SCHEMAS.md §1:
     - confidence < 0.6 ⇒ verdict must be REJECT, reason_code SCOPE_TOO_VAGUE
     - if REJECT: alternative_recommendation must be non-null
     - if PROCEED: expected_multi_agent_benefit must be non-null
5. Render the verdict to the user in a human-readable format:
     - PROCEED: green checkmark, reasoning, expected benefit
     - REJECT: red X, reason code, reasoning, alternative recommendation
     - SCOPE_TOO_VAGUE: also list clarifying_questions with "next steps"
6. Stop. Do not invoke any Stage 1 agents.
```

### `/prdforge clarify --prd <path> --out <dir>`

**Stages 0 + 1.** Produces the validated artifact bundle.

```
1. Run Stage 0 internally (steps 1-4 of `check` above).
   - If REJECT: print verdict, exit. No bundle.
   - If PROCEED: continue.

2. Compute bundle slug:
     slug = lowercased, hyphenated form of ProductSpec.document_id
            (or PRD title if document_id absent)
   bundle_dir = <out>/<slug>/  (or <out>/ if user explicitly passed a slug)

3. Create bundle_dir if it does not exist.

4. Copy the source PRD bit-for-bit:
     cp <path> bundle_dir/PRD.md
     # Verify with sha256sum that the copy is identical to the source.

5. Invoke @prd-parser with the PRD markdown.
     → ProductSpec JSON.

6. Invoke @requirement-analyzer with ProductSpec + PRD markdown.
     → RequirementSet JSON (R-01...R-NN).

7. Render REQUIREMENTS.md using .claude/skills/templates/requirements-template.md
   with the RequirementSet data. Write to bundle_dir/REQUIREMENTS.md.

8. Invoke @clarification-auditor with ProductSpec + RequirementSet + PRD markdown.
     → ClarificationLog JSON.

9. If ClarificationLog has entries:
     a. Surface them to the user in one batch (numbered list, with relates_to,
        ambiguity_type, and proposed multi-choice options where applicable).
     b. Collect the user's answers (or skip-flags) for each.
     c. Update the ClarificationLog with user_answer / user_skipped /
        resolution_kind for each entry.
     d. If any entry has resolution_kind: NEW_REQUIREMENT:
        - Re-invoke @requirement-analyzer with the resolved clarifications
          appended to ProductSpec.raw_sections to register the new R-* IDs.
        - Re-render REQUIREMENTS.md.

10. Render CLARIFICATIONS.md using clarifications-template.md
    with the (now resolved) ClarificationLog. Write to bundle_dir/CLARIFICATIONS.md.

11. Invoke @domain-researcher with ProductSpec + RequirementSet + ClarificationLog.
     → DomainContext JSON.
   (DomainContext is held in memory for V1; not written to bundle. V2 will use it
    for the architecture brief.)

12. Write _metadata.json to bundle_dir with:
     - bundle_version: 1.0
     - prdforge_version: <current>
     - generated_at: ISO 8601 timestamp
     - source_prd_path: <path>
     - source_prd_sha256: <sha>
     - suitability_verdict: { verdict, reason_code, confidence }
     - auditor_verdict: <ClarificationLog.auditor_verdict>
     - requirement_count: <RequirementSet.requirements.length>
     - clarification_count: <ClarificationLog.entries.length>
     - skipped_clarifications: <count of entries with user_skipped: true>
     - traceability_status: null  (filled in by `prdforge trace`)
     - traceability_run_at: null

13. Print bundle summary to user:
     ✅ Bundle written to bundle_dir/
        - PRD.md (immutable copy)
        - REQUIREMENTS.md (N requirements)
        - CLARIFICATIONS.md (M entries, K skipped)
        - _metadata.json
     Next: build your downstream system, then run /prdforge trace to validate coverage.
```

### `/prdforge trace --bundle <path> --delivery <path>`

**Stage 5 traceability gate.** Validates downstream coverage. Fail-closed.

```
1. Read bundle's REQUIREMENTS.md, CLARIFICATIONS.md, and _metadata.json.
2. Verify <delivery> directory exists and is readable.
3. Invoke @architecture-reviewer with bundle_path and delivery_path.
     → ReviewReport JSON with traceability_matrix.
4. Render TRACEABILITY.md using traceability-template.md with the matrix.
   Write to <bundle>/TRACEABILITY.md.
5. Update <bundle>/_metadata.json:
     - traceability_status: "PASS" or "FAIL"
     - traceability_run_at: ISO 8601 timestamp
6. Print verdict to user:
     - PASS: green check, requirement counts by status, any PARTIAL notes
     - FAIL: red X, list of NONE requirements, fix_hints from ReviewReport
7. Exit non-zero on FAIL (so CI integrations can detect the gate).
```

### `/prdforge bundle --show <path>`

Inspect an existing bundle.

```
1. Read <path>/_metadata.json.
2. Print:
     - source PRD path + sha256 (truncated)
     - suitability verdict (verdict, reason_code, confidence)
     - auditor verdict, requirement count, clarification count
     - skipped clarifications (if any)
     - traceability status (if `prdforge trace` has run)
3. List files in the bundle directory with sizes.
4. If TRACEABILITY.md exists, print the count summary table from its frontmatter.
```

### `/prdforge advise --bundle <path>` *(V2 — placeholder)*

Will produce an advisory architecture brief (topology recommendation, agent roster proposal, schema design) as a single markdown report. **Not a code generator.** When invoked today, return:

> "/prdforge advise is a V2 feature. V1 closes with the validated bundle + traceability gate. Track progress in STATUS.md."

---

## V1 pipeline overview

```
[Stage 0] @multi-agent-suitability-checker          (sequential, gate)
    │ PROCEED
    ▼
[Stage 1] @prd-parser
          @requirement-analyzer                      (sequential)
          @clarification-auditor      ← writes CLARIFICATIONS.md
          @domain-researcher
    ▼
        Bundle written: PRD.md + REQUIREMENTS.md
        + CLARIFICATIONS.md + _metadata.json
    │
    │ (user builds downstream system using bundle as input)
    │
    ▼
[Stage 5] @architecture-reviewer (lightweight)       (sequential, fail-closed)
          → emits TRACEABILITY.md
    ▼
        Validated, audit-ready bundle
```

---

## Implementation status

This is **V0.3** — V1.0-ready as of this commit.

- ✅ `Stage 0 — @multi-agent-suitability-checker` (Moat 1: runtime refusal)
- ✅ `Stage 1 — @prd-parser`
- ✅ `Stage 1 — @requirement-analyzer` (emits stable R-* IDs)
- ✅ `Stage 1 — @clarification-auditor` (Moat 2 part 1: writes CLARIFICATIONS.md)
- ✅ `Stage 1 — @domain-researcher` (web-search enrichment)
- ✅ `Stage 5 — @architecture-reviewer` (lightweight V1 traceability gate, Moat 2 part 2)
- ✅ Bundle format spec (`docs/BUNDLE_FORMAT.md`)
- ✅ All output templates (`clarifications-template.md`, `requirements-template.md`, `traceability-template.md`)

All four V1 commands (`check`, `clarify`, `trace`, `bundle`) are runnable end-to-end today.

V2 work (Stages 2–4 architecture brief) is planned for Q3 2026.

---

## Design constraints

These are non-negotiable and enforced via [CLAUDE.md](../../../CLAUDE.md) and the rubrics in [`docs/`](../../../docs/):

- The source PRD is **immutable**. No agent modifies it.
- Stage 0 can refuse. The pipeline must respect a `REJECT` verdict.
- Stage 5 fails closed. A bundle with any unmapped requirement does not pass.
- Tight PRDs produce empty `CLARIFICATIONS.md`. No tax for clarity.
- Output is markdown + JSON only. Framework-agnostic. No code generation in V1 or V2.
- Agent count is capped at 12 across all 6 stages (V1 uses 6, V2 adds 6 more).
- Product docs never name competitor tools. Positioning stands on its own merits.
