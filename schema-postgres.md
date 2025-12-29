# Postgres Schema – Email Management System

This document defines the **operational database schema** used to
control email sending, compliance, and system state.

## Entity Relationship Diagram (ERD)

```mermaid
erDiagram
  CUSTOMERS {
    uuid customer_id PK
    text email "UNIQUE, lowercase"
    text name
    text company
    text country
    boolean is_active
    timestamptz created_at
    timestamptz updated_at
  }

  TEMPLATES {
    uuid template_id PK
    text provider "ses|internal"
    text provider_template_name
    int week_of_month "1-4"
    int version
    boolean is_active
    timestamptz created_at
  }

  CAMPAIGNS {
    uuid campaign_id PK
    text campaign_name
    int week_of_month "1-4"
    date generated_date
    uuid template_id FK
    text status "draft|queued|sending|done|paused"
    timestamptz created_at
  }

  CAMPAIGN_RECIPIENTS {
    uuid campaign_id PK, FK
    uuid customer_id PK, FK
    text email
    text recipient_status "pending|sent|failed|suppressed|bounced|complained|unsubscribed"
    text ses_message_id
    timestamptz last_attempt_at
    int attempt_count
    jsonb template_data
    timestamptz created_at
    timestamptz updated_at
  }

  SUPPRESSION_LIST {
    text email PK
    text reason "unsubscribe|bounce_hard|complaint|manual"
    text source "user|ses|ops"
    timestamptz created_at
    timestamptz updated_at
  }

  UNSUBSCRIBE_TOKENS {
    uuid token_id PK
    text email
    uuid campaign_id FK
    timestamptz expires_at
    timestamptz created_at
    timestamptz used_at
  }

  EMAIL_EVENTS_OP {
    uuid event_id PK
    text event_type "delivery|bounce|complaint|open|click|unsubscribe"
    text email
    uuid campaign_id FK
    uuid customer_id FK
    text ses_message_id
    timestamptz event_time
    jsonb payload
    timestamptz created_at
  }

  CUSTOMERS ||--o{ CAMPAIGN_RECIPIENTS : "customer_id"
  CAMPAIGNS ||--o{ CAMPAIGN_RECIPIENTS : "campaign_id"
  CAMPAIGNS ||--o{ EMAIL_EVENTS_OP : "campaign_id"
  CUSTOMERS ||--o{ EMAIL_EVENTS_OP : "customer_id"
```

## Design Notes

- **campaign_recipients** uses a composite primary key
  (`campaign_id`, `customer_id`) to prevent duplicate sends.
- **suppression_list** is the authoritative source for compliance.
- **email_events_op** stores recent operational events for
  troubleshooting and support use cases.
- Full historical analytics are stored in Redshift.
