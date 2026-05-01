# SupportAI — Customer Support Triage System

| Field | Detail |
|---|---|
| **Document ID** | SA-PRD-001 |
| **Version** | 0.1 |
| **Date** | March 2026 |
| **Status** | Reference example for PRDForge — should PROCEED at Stage 0 but produce 4–6 clarifications at Stage 1 |

> **Expected behavior:**
> - Stage 0 (`@multi-agent-suitability-checker`): `PROCEED` (the problem is genuinely multi-agent — triage + classify + route across multiple specialists).
> - Stage 1 (`@clarification-auditor`): emits 4–6 clarifications covering MISSING_DETAIL, UNDEFINED_TERM, CONTRADICTION, and GAP.

---

## 1. Objective

SupportAI triages incoming customer support tickets and routes them to the right team **fast**. It uses specialist agents to classify intent, assess urgency, detect VIP status, and recommend a response template. Premium users get priority handling.

## 2. Target Users

- Customer support managers who want faster ticket resolution.
- On-call engineers who currently get paged for issues that should have gone to the success team.

## 3. Functional Requirements

- **R-01:** Accept tickets from email, chat, and the in-app help widget.
- **R-02:** Classify each ticket into one of: `bug`, `billing`, `account`, `feature_request`, `how_to`, `escalation`.
- **R-03:** Assess urgency on a 0–3 scale (0=low, 3=critical).
- **R-04:** Identify whether the customer is a premium user. Premium users get priority routing.
- **R-05:** Recommend a response template from the team's existing template library.
- **R-06:** All ticket data must remain on-device for privacy reasons.
- **R-07:** Use OpenAI GPT-4o for the urgency-classification step (fastest model available).
- **R-08:** Route the ticket to the correct team based on classification.
- **R-09:** The system should be fast.

## 4. Non-Goals

- Auto-responding to tickets (humans still write the actual reply).
- Handling phone calls.

## 5. Success Criteria

- Faster mean time to first response.
- Fewer mis-routed tickets.
- Customer satisfaction improves.

## 6. Why this is multi-agent

Each step (classify intent, assess urgency, detect VIP, recommend template) requires different reasoning over different signals. Running them as specialists in parallel gives faster wall time AND better per-step quality than a single monolithic prompt.
