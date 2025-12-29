# Redshift Schema – Email Management System (Analytics)

This document defines the **analytics warehouse schema** used for reporting,
trend analysis, and dashboards. Redshift stores **historical, append-only**
event data.

## Entity Relationship Diagram (Star Schema)

```mermaid
erDiagram
  DIM_CUSTOMER {
    bigint customer_key PK "identity/surrogate key"
    uuid customer_id "from Postgres"
    text email_or_hash
    text company
    text country
  }

  DIM_CAMPAIGN {
    bigint campaign_key PK "identity/surrogate key"
    uuid campaign_id "from Postgres"
    text campaign_name
    int week_of_month
    date generated_date
    text template_name
  }

  DIM_TIME {
    int time_key PK "YYYYMMDD"
    date the_date
    int year
    int month
    int week
    int day_of_week
  }

  FACT_EMAIL_EVENT {
    bigint event_key PK "identity"
    int time_key FK
    bigint customer_key FK
    bigint campaign_key FK
    text event_type "delivery|bounce|complaint|open|click|unsubscribe"
    text ses_message_id
    text bounce_type
    text complaint_type
    text raw_payload "json string / SUPER"
  }

  DIM_TIME ||--o{ FACT_EMAIL_EVENT : "time_key"
  DIM_CUSTOMER ||--o{ FACT_EMAIL_EVENT : "customer_key"
  DIM_CAMPAIGN ||--o{ FACT_EMAIL_EVENT : "campaign_key"
```

## Why Redshift uses Facts and Dimensions

- **fact_email_event** is the append-only table of “things that happened.”
- **dim_customer / dim_campaign / dim_time** provide context for grouping,
  filtering, and trend reporting.
- This design is optimized for queries like:
  - delivery rate by week
  - bounce rate by domain
  - complaints by campaign
  - opens/clicks by customer segment

## Data Loading Notes (Typical)

### From Lambda / Event Pipeline
- Insert a row into **fact_email_event** for every SNS event
  (delivery/bounce/complaint, and later open/click).
- Maintain dimensions by:
  - Upserting dim tables from Postgres (batch ETL), or
  - Creating dimension rows when new IDs appear (streaming approach)

### Source of Truth
- Postgres remains the **operational source of truth**
  (suppression, send status, current state).
- Redshift remains the **historical analytics store**.
