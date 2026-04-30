# InventoryDB — Internal Inventory CRUD App

| Field | Detail |
|---|---|
| **Document ID** | INV-PRD-001 |
| **Version** | 1.0 |
| **Date** | March 2026 |
| **Status** | Reference example for PRDForge — should REJECT as `NOT_AI_PROBLEM` |

> **Expected Stage 0 verdict:** `REJECT` with `reason_code: NOT_AI_PROBLEM`, confidence ≥ 0.85.

---

## 1. Objective

InventoryDB is an internal web application for tracking warehouse inventory. Staff add, edit, list, and delete items. Items have a SKU, a name, a quantity, a location, a supplier reference, and a last-updated timestamp.

## 2. Functional Requirements

- **R-01:** Authenticate users against the company SSO (SAML).
- **R-02:** Provide a list view with pagination (50 items per page) and column-based sorting.
- **R-03:** Provide a detail view per SKU showing all fields and the audit history (who changed what, when).
- **R-04:** Allow authorized roles to add new items, edit existing items, and soft-delete items. Hard delete is admin-only.
- **R-05:** Persist data in PostgreSQL. Use database migrations for schema changes.
- **R-06:** Export the current inventory as CSV on demand.
- **R-07:** Search by SKU substring or name keyword (full-text index acceptable).
- **R-08:** All write actions emit a domain event to a Kafka topic for downstream warehouse-management systems.
- **R-09:** Backups run nightly. Recovery point objective is 24 hours.

## 3. Non-Goals

- No public-facing user interface (internal tool only).
- No machine learning or recommendations.
- No image upload (text-only fields).

## 4. Success Criteria

- p95 page load under 500ms for the list view.
- Zero data loss across the audit history.
- Full audit history visible for any SKU's last 12 months of changes.

## 5. Tech preferences

- Backend: Django or Flask (Python).
- Frontend: Server-rendered (Django templates) or a thin React layer.
- Hosting: Internal Kubernetes cluster.

## 6. Why this is straightforward CRUD

There's no reasoning component here — every requirement is deterministic, every action has a closed-form data operation, and the value is in the audit-log discipline and the operational reliability, not in any AI inference. This is exactly the kind of system a small team can build with a regular framework in 2–3 weeks. An LLM in the loop would add latency, cost, and non-determinism without solving anything.
