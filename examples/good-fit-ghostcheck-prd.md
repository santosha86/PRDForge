# GhostCheck — Blind-Panel CV Evaluator

| Field | Detail |
|---|---|
| **Document ID** | GC-PRD-001 |
| **Version** | 1.0 |
| **Date** | March 2026 |
| **Status** | Reference example for PRDForge — should PROCEED at Stage 0 |

> **Expected Stage 0 verdict:** `PROCEED` with `reason_code: SUITABLE`, confidence ≥ 0.85.

---

## 1. Objective

GhostCheck is a blind-panel evaluator for CVs and resumes. A user submits a candidate's CV; the system returns an aggregated verdict assembled from 11 specialist evaluators that each score the CV on an independent dimension, with defended rationales.

## 2. The 11 Evaluation Dimensions

Each dimension is scored 0–100 by a dedicated specialist agent. Dimensions are independent of each other — no specialist sees another's score.

| # | Dimension | What it measures |
|---|---|---|
| 1 | ATS Compliance | Will an applicant tracking system parse this CV cleanly? |
| 2 | Google Presence | Does the candidate's online footprint match the CV claims? |
| 3 | Bucket Fit | Does the candidate's experience map to the target role bucket (IC vs lead vs principal)? |
| 4 | Channel Match | Is the CV pitched at the right hiring channel (referral, recruiter, cold)? |
| 5 | Salary Alignment | Are the candidate's stated/inferred expectations within the role's band? |
| 6 | Domain Depth | How deep is their expertise in the role's primary domain? |
| 7 | Recency | Is their relevant experience recent enough to still be sharp? |
| 8 | Trajectory | Does their career progression suggest a fit for the seniority asked? |
| 9 | Communication Quality | Is the CV itself well-written, well-organized, error-free? |
| 10 | Red Flag Audit | Are there gaps, contradictions, or claims that warrant follow-up? |
| 11 | Cultural Signals | Do their stated interests/causes align with the team's published culture? |

## 3. Functional Requirements

- **R-01:** Accept a CV as PDF, DOCX, or markdown.
- **R-02:** Run all 11 specialists in parallel; total wall time must be under 60 seconds for typical CVs.
- **R-03:** Each specialist must return both a 0–100 score AND a 2–4 sentence defended rationale.
- **R-04:** Aggregate the 11 scores via a configurable weighted sum into a final verdict (PASS / REVIEW / REJECT).
- **R-05:** Final output must include all 11 individual scores and rationales — the user sees the full panel, not just the verdict.
- **R-06:** Local-first execution; no candidate data leaves the user's machine.
- **R-07:** BYOK — user supplies their own Anthropic API key.

## 4. Non-Goals

- Not a candidate-side tool (the candidate never sees the scores).
- Not a hiring-decision system — the human recruiter still makes the call.
- Not a resume-builder.

## 5. Success Criteria

- Inter-rater agreement with a senior recruiter on PASS/REJECT decisions ≥ 85% on a 50-CV golden set.
- Median end-to-end runtime under 45 seconds.
- Zero data leaves the user's machine in a default-config run.

## 6. Why this is multi-agent

The 11 dimensions require fundamentally different reasoning. ATS parsing is a structural analysis. Google presence is a web-research task. Salary alignment is a market-data lookup plus inference. Cultural signals is a soft, interpretive read. A single prompt asked to do all 11 in one pass loses depth on every one. Specialization wins; parallel execution compresses wall time ~10x vs. a sequential loop.
