# LeadSync — HubSpot to Mailchimp Lead Forwarder

| Field | Detail |
|---|---|
| **Document ID** | LS-PRD-001 |
| **Version** | 1.0 |
| **Date** | March 2026 |
| **Status** | Reference example for PRDForge — should REJECT as `WORKFLOW_AUTOMATION` |

> **Expected Stage 0 verdict:** `REJECT` with `reason_code: WORKFLOW_AUTOMATION`, confidence ≥ 0.85.

---

## 1. Objective

LeadSync watches HubSpot for newly added leads. When a new lead appears, it sends a templated welcome email and adds the lead to a designated Mailchimp list.

## 2. Functional Requirements

- **R-01:** Subscribe to HubSpot's "new contact" webhook.
- **R-02:** On webhook fire, extract the contact's email and first name.
- **R-03:** Send a templated welcome email via SendGrid using a fixed template ID.
- **R-04:** Add the contact to a Mailchimp audience list using the Mailchimp API.
- **R-05:** On any step failure, retry up to 3 times with exponential backoff, then log the failure.

## 3. Non-Goals

- No content personalization beyond first-name substitution.
- No segmentation logic — every new contact goes to the same list.
- No analytics or attribution.

## 4. Success Criteria

- 99.5% delivery rate end-to-end.
- Median latency under 30 seconds from HubSpot event to Mailchimp confirmation.
- Zero data persistence between events (stateless).

## 5. Why this exists

Sales ops needs a reliable bridge between HubSpot (where leads arrive) and Mailchimp (where they nurture). Today this is a manual copy/paste a few times per day, and leads slip through.
