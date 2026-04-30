# BUNDLE_FORMAT.md

The on-disk contract for a PRDForge **validated artifact bundle**. Every `prdforge clarify` run that passes Stage 0 produces a bundle in this format. Every `prdforge trace` run reads it.

The bundle is the V1 product output. It is plain markdown + JSON, framework-agnostic, and designed to be auditable by external reviewers without running PRDForge itself.

---

## Directory layout

```
out/<slug>/
├─ PRD.md                  # bit-for-bit copy of the source PRD (immutable)
├─ CLARIFICATIONS.md       # audit log of resolved ambiguities (may be empty)
├─ REQUIREMENTS.md         # numbered requirements R-01…R-NN with acceptance criteria
├─ TRACEABILITY.md         # coverage matrix; written only after `prdforge trace`
└─ _metadata.json          # machine-readable summary
```

The `<slug>` is derived from the PRD's `document_id` field (lowercased, hyphenated). If absent, derived from the PRD title.

---

## File-by-file contract

### `PRD.md` (immutable)

A **bit-for-bit copy** of the input PRD. PRDForge's core discipline is that the source spec is never modified; the only way the file in the bundle differs from the user's input is that PRDForge made a copy. The bytes are identical.

**Verification:** `sha256sum out/<slug>/PRD.md` should equal `sha256sum <input-prd>`.

### `CLARIFICATIONS.md` (append-only during run)

YAML frontmatter + markdown body. The frontmatter exposes the auditor's verdict and entry count for machine readers; the body renders each entry in human-readable form.

When the auditor returned `NO_CLARIFICATIONS_NEEDED`, the body contains a single "✅ No clarifications needed" section and the `entries` array in frontmatter is empty.

Format defined in `.claude/skills/templates/clarifications-template.md`.

### `REQUIREMENTS.md`

YAML frontmatter (with the `R-*` ID list) followed by a markdown rendering of each requirement, including:

- ID, title, priority
- Description
- Acceptance criteria (testable conditions)
- Source PRD section (or `null` if synthesized)

If `CLARIFICATIONS.md` contains entries with `resolution_kind: NEW_REQUIREMENT`, those new IDs (`R-12`, `R-13`, …) appear in `REQUIREMENTS.md` after the analyzer is re-run. The `source_section` for these is `"CLARIFICATION:C-XX"` (e.g. `"CLARIFICATION:C-04"`).

Format defined in `.claude/skills/templates/requirements-template.md`.

### `TRACEABILITY.md`

Written only after `prdforge trace --bundle <path> --delivery <path>` runs successfully. YAML frontmatter (with verdict and per-status counts) followed by a markdown table with one row per requirement.

Each row has:
- `R-*` ID
- Requirement title
- Source (`PRD` or `CLARIFICATION`)
- Coverage status (`FULL` | `PARTIAL` | `NONE`)
- Covered-by (file paths or agent names from the delivery)
- Notes (required for `PARTIAL` and `NONE`)

`TRACEABILITY.md` is **replaceable per run.** Re-running `prdforge trace` overwrites it. The history is in git.

Format defined in `.claude/skills/templates/traceability-template.md`.

### `_metadata.json`

```json
{
  "bundle_version": "1.0",
  "prdforge_version": "0.3",
  "generated_at": "2026-04-30T14:32:00Z",
  "source_prd_path": "examples/ambiguous-prd.md",
  "source_prd_sha256": "a3f5...",
  "suitability_verdict": {
    "verdict": "PROCEED",
    "reason_code": "SUITABLE",
    "confidence": 0.88
  },
  "auditor_verdict": "CLARIFICATIONS_RESOLVED",
  "requirement_count": 12,
  "clarification_count": 4,
  "skipped_clarifications": 0,
  "traceability_status": "PASS",
  "traceability_run_at": "2026-04-30T15:08:00Z"
}
```

This file enables CI consumption — a downstream tool can read `_metadata.json` to decide whether to proceed without parsing the markdown.

`traceability_status` and `traceability_run_at` are absent until `prdforge trace` has been run.

---

## Lifecycle of a bundle

```
1. prdforge check --prd X.md
   → emits SuitabilityVerdict to stdout
   → if REJECT: stops here. NO BUNDLE PRODUCED.
   → if PROCEED: caller decides whether to run clarify next.

2. prdforge clarify --prd X.md --out ./out/myslug/
   → re-runs Stage 0 internally (must PROCEED)
   → runs Stage 1 (parser → analyzer → auditor → researcher)
   → writes:
       out/myslug/PRD.md
       out/myslug/CLARIFICATIONS.md
       out/myslug/REQUIREMENTS.md
       out/myslug/_metadata.json
   → if user skipped clarifications: bundle still ships, but
     _metadata.json has skipped_clarifications > 0.

3. (User builds the downstream system using the bundle as input.)

4. prdforge trace --bundle ./out/myslug/ --delivery ./path/to/built/system/
   → reads bundle's REQUIREMENTS.md
   → scans delivery for evidence of each R-*
   → writes:
       out/myslug/TRACEABILITY.md
   → updates _metadata.json with traceability_status and traceability_run_at
   → fails closed (non-zero exit) if any requirement is NONE.

5. prdforge bundle --show ./out/myslug/
   → prints a concise summary from _metadata.json + the markdown files
```

Every step is **idempotent** at the bundle level — re-running `clarify` overwrites `CLARIFICATIONS.md`, `REQUIREMENTS.md`, and `_metadata.json` (but not `PRD.md` if its sha256 is unchanged). Re-running `trace` overwrites `TRACEABILITY.md`.

---

## What the bundle is *not*

- **It is not generated code.** PRDForge V1 and V2 do not generate runnable systems. The bundle is purely the validation/audit artifact.
- **It is not a contract with a specific framework.** The user is free to build the downstream system in any tool, in any language, on any platform.
- **It is not editable.** Users should not hand-edit `CLARIFICATIONS.md` or `REQUIREMENTS.md` after they're written; if requirements change, edit the source PRD and re-run `prdforge clarify`. The bundle is regenerated, not patched.

---

## Why this shape

Three reasons for the immutable-PRD-plus-additive-log shape:

1. **Audit-grade.** Regulators (EU AI Act, NIST AI RMF, ISO 42001) ask for provable spec→delivery traceability. An immutable source plus an append-only clarifications log plus a coverage matrix is exactly what they ask for.
2. **Auditable without PRDForge.** Anyone with a markdown reader can review the bundle. There is no proprietary format or special tooling required for the audit.
3. **Framework-agnostic.** The bundle is the validation step; the user picks any execution path downstream. PRDForge is the governance layer, not the build tool.

---

**Format version:** 1.0 (V1 release)
**Last reviewed:** April 2026
