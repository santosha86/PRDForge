# MeetingSum — Meeting Transcript Summarizer

| Field | Detail |
|---|---|
| **Document ID** | MS-PRD-001 |
| **Version** | 1.0 |
| **Date** | March 2026 |
| **Status** | Reference example for PRDForge — should REJECT as `SINGLE_AGENT_SUFFICIENT` |

> **Expected Stage 0 verdict:** `REJECT` with `reason_code: SINGLE_AGENT_SUFFICIENT`, confidence ≥ 0.80.

---

## 1. Objective

MeetingSum reads a raw meeting transcript and emits a structured list of action items, each with an owner and a due date. That's it.

## 2. Functional Requirements

- **R-01:** Accept a transcript as plain text or markdown (max 30,000 tokens).
- **R-02:** Output a list of action items. Each item has:
  - `description`: what needs to be done
  - `owner`: who is responsible (the speaker who took the action, or someone explicitly assigned)
  - `due_date`: ISO 8601 date if explicitly mentioned, else null
- **R-03:** Skip items that are not actionable (general discussion, status updates, banter).
- **R-04:** Preserve the speaker's original phrasing where possible — don't rewrite or interpret.

## 3. Non-Goals

- No speaker diarization (assume the transcript is already labeled).
- No follow-up generation.
- No integration with task trackers (the user copies the output where they want it).

## 4. Success Criteria

- 90% precision on the action-item extraction (a senior PM agrees the item is real).
- 85% recall (catches most genuine action items).
- Median runtime under 5 seconds for a 5,000-word transcript.

## 5. Why this is one task, not many

Reading a transcript and pulling out action items is a single reasoning task. There's no decomposition that helps — you can't usefully split "extract the verb" from "extract the owner" from "extract the date." The reasoning is interleaved. One well-prompted call with a structured output schema beats any multi-agent decomposition on quality, latency, and cost.
