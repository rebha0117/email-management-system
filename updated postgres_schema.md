```mermaid
erDiagram
  CONTACTS {
    uuid contact_id PK
    uuid account_id
    text primary_email
    text country_code
    boolean is_active
    timestamptz created_at
  }

  ACCOUNTS {
    uuid account_id PK
    text account_name
    text country_code
    boolean is_active
    timestamptz created_at
  }

  CAMPAIGNS {
    uuid campaign_id PK
    text name
    boolean is_active
    timestamptz created_at
  }

  MODEL_RUNS {
    uuid model_run_id PK
    date scoring_week
    text model_version
    text s3_uri
    timestamptz created_at
  }

  TEMPLATES {
    uuid template_id PK
    text name
    boolean is_active
    timestamptz created_at
  }

  TEMPLATE_VERSIONS {
    uuid template_version_id PK
    uuid template_id
    int version_number
    text subject_template
    text html_template
    text text_template
    timestamptz created_at
  }

  CAMPAIGN_RUNS {
    uuid campaign_run_id PK
    uuid campaign_id
    date run_date
    int week_of_month
    uuid model_run_id
    uuid template_version_id
    text status
    timestamptz created_at
  }

  CAMPAIGN_RUN_CONTEXT {
    uuid campaign_run_id PK
    jsonb context_json
    text context_checksum
    text source
    timestamptz fetched_at
  }

  MESSAGES {
    uuid message_id PK
    uuid campaign_run_id
    uuid contact_id
    uuid account_id
    text to_email
    text current_status
    jsonb render_context_json
    text rendered_subject
    text rendered_html
    text rendered_text
    timestamptz queued_at
    timestamptz sent_at
    timestamptz last_event_at
  }

  SEND_ATTEMPTS {
    uuid attempt_id PK
    uuid message_id
    text idempotency_key
    text ses_message_id
    text attempt_status
    timestamptz attempted_at
  }

  MESSAGE_EVENTS {
    uuid event_id PK
    uuid message_id
    text ses_message_id
    text event_type
    jsonb raw_payload
    timestamptz event_ts
    timestamptz ingested_at
  }

  SUPPRESSIONS {
    uuid suppression_id PK
    text scope_type
    text scope_value
    text reason
    text source
    timestamptz created_at
  }

  ACCOUNTS ||--o{ CONTACTS : has
  CAMPAIGNS ||--o{ CAMPAIGN_RUNS : has
  MODEL_RUNS ||--o{ CAMPAIGN_RUNS : uses
  TEMPLATES ||--o{ TEMPLATE_VERSIONS : versions
  TEMPLATE_VERSIONS ||--o{ CAMPAIGN_RUNS : applied
  CAMPAIGN_RUNS ||--|| CAMPAIGN_RUN_CONTEXT : has
  CAMPAIGN_RUNS ||--o{ MESSAGES : creates
  CONTACTS ||--o{ MESSAGES : receives
  MESSAGES ||--o{ SEND_ATTEMPTS : attempts
  MESSAGES ||--o{ MESSAGE_EVENTS : events
