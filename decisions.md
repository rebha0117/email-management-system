# Architecture Decisions – Email Management System

This document records **key technical decisions**, their reasoning,
and alternatives considered. It exists to make design intent explicit
and reviewable.

---

## Decision 1: Use Postgres for Operational Data

**Status:** Accepted

**Context:**  
The system needs a reliable source of truth for:
- customer eligibility
- send status
- suppression and compliance rules

**Decision:**  
Use **Postgres** as the operational database.

**Rationale:**
- Strong transactional guarantees (ACID)
- Suitable for frequent reads/writes and status updates
- Enforces constraints to prevent duplicate sends

**Alternatives Considered:**
- Using Redshift for operational queries (rejected — analytics optimized)
- Using DynamoDB (rejected — relational integrity needed)

---

## Decision 2: Use Redshift for Analytics and Historical Events

**Status:** Accepted

**Context:**  
Email delivery, bounce, complaint, and engagement events need to be
stored long-term and analyzed over time.

**Decision:**  
Store immutable email events in **Redshift** using a fact/dimension model.

**Rationale:**
- Optimized for large scans and aggregations
- Clear separation of operational vs analytical concerns
- Enables efficient reporting and trend analysis

**Alternatives Considered:**
- Storing all events in Postgres (rejected — scaling and query cost)
- Querying SES logs directly (rejected — not query-friendly)

---

## Decision 3: Keep `email_events_op` in Postgres

**Status:** Accepted

**Context:**  
Operational teams may need fast, recent event visibility for:
- customer support
- troubleshooting delivery issues
- real-time system inspection

**Decision:**  
Maintain a lightweight **email_events_op** table in Postgres for
recent operational events.

**Rationale:**
- Fast lookup by email or campaign
- Avoids querying Redshift for operational workflows
- Acts as a short-term cache (not a reporting source)

**Constraints:**
- Retention limited (e.g., 7–30 days)
- Not used for analytics or dashboards

---

## Decision 4: Use AWS SES Templates (Not Python-Rendered Emails)

**Status:** Accepted

**Context:**  
Weekly campaigns use predefined templates (Week 1–4) with
customer-specific data.

**Decision:**  
Define and manage email templates in **AWS SES**.
Pass per-recipient data as JSON from the sender service.

**Rationale:**
- Optimized for bulk sending
- Centralized template management
- Reduced database storage
- Cleaner compliance handling (unsubscribe links)

**Alternatives Considered:**
- Python-rendered HTML templates (rejected — harder to scale and manage)

---

## Decision 5: Event-Driven Feedback Handling via SNS and Lambda

**Status:** Accepted

**Context:**  
Delivery, bounce, complaint, and engagement events must be processed
asynchronously and at scale.

**Decision:**  
Use **SNS → Lambda** for event processing.

**Rationale:**
- Event-driven and scalable
- No always-on servers required
- Natural integration with SES
- Simplifies parallel event handling

**Alternatives Considered:**
- Polling SES logs (rejected — inefficient)
- Long-running EC2 consumers (rejected — operational overhead)

---

## Decision 6: Separate Sending Logic from Feedback Processing

**Status:** Accepted

**Context:**  
Email sending and event processing have different scaling and failure
characteristics.

**Decision:**  
- Sender service handles **outbound** emails
- Lambda handles **inbound** events

**Rationale:**
- Clear separation of responsibilities
- Easier error handling and retries
- Improves system reliability

---

## Summary

This architecture prioritizes:
- Compliance and deliverability
- Clear data ownership
- Operational reliability
- Scalable analytics

Design decisions are intentionally conservative and aligned with
AWS best practices for bulk email systems.
