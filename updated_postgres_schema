erDiagram
  CONTACTS {
    uuid contact_id PK
    uuid account_id FK
    text primary_email
    text first_name
    text last_name
    text phone
    text country_code
    text crm_contact_ref
    boolean is_active
    timestamptz created_at
    timestamptz updated_at
  }

  ACCOUNTS {
    uuid account_id PK
    text account_name
    text crm_account_ref
    text country_code
    boolean is_active
    timestamptz created_at
    timestamptz updated_at
  }

  SALES_REPS {
    uuid sales_rep_id PK
    text name
    text email
    text crm_user_ref
    boolean is_active
    timestamptz created_at
  }

  ACCOUNT_OWNERSHIP {
    uuid account_id FK
    uuid sales_rep_id FK
    text role
    date effective_from
    date effective_to
  }

  CAMPAIGNS {
    uuid campaign_id PK
    text name
    text purpose
    boolean is_active
    timestamptz created_at
  }

  CAMPAIGN_RUNS {
    uuid campaign_run_id PK
    uuid campaign_id FK
    date run_date
    int week_of_month
    uuid model_run_id FK
    uuid template_version_id FK
    uuid sending_policy_id FK
    text run_label
    text status
    timestamptz created_at
  }

  MODEL_RUNS {
    uuid model_run_id PK
    date scoring_week
    text model_version
    text s3_uri
    text checksum
    int predicted_count
    timestamptz created_at
  }

  AUDIENCE_MEMBERS {
    uuid audience_member_id PK
    uuid model_run_id FK
    uuid account_id FK
    uuid contact_id FK
    numeric churn_score
    text segment_label
    timestamptz created_at
  }

  TEMPLATES {
    uuid template_id PK
    text name
    text description
    boolean is_active
    timestamptz created_at
  }

  TEMPLATE_VERSIONS {
    uuid template_version_id PK
    uuid template_id FK
    text subject_template
    text html_template
    text text_template
    int version_number
    timestamptz created_at
  }

  CAMPAIGN_RUN_CONTEXT {
    uuid campaign_run_id PK
    jsonb context_json
    text source
    text checksum
    timestamptz fetched_at
  }

  SENDING_POLICIES {
    uuid sending_policy_id PK
    text country_code
    text legal_entity_name
    boolean is_active
    text from_email
    text reply_to_email
    timestamptz created_at
    timestamptz updated_at
  }

  MESSAGES {
    uuid message_id PK
    uuid campaign_run_id FK
    uuid contact_id FK
    uuid account_id FK
    text to_email
    text rendered_subject
    text current_status
    timestamptz queued_at
    timestamptz sent_at
    timestamptz last_event_at
  }

  SEND_ATTEMPTS {
    uuid attempt_id PK
    uuid message_id FK
    text idempotency_key
    text ses_message_id
    text provider
    text request_payload_hash
    text attempt_status
    timestamptz attempted_at
  }

  MESSAGE_EVENTS {
    uuid event_id PK
    uuid message_id FK
    text ses_message_id
    text event_type
    timestamptz event_ts
    jsonb raw_payload
    timestamptz ingested_at
  }

  SUPPRESSIONS {
    uuid suppression_id PK
    text scope_type
    text scope_value
    text reason
    text source
    timestamptz starts_at
    timestamptz expires_at
    timestamptz created_at
  }

  SUBSCRIPTION_PREFERENCES {
    uuid preference_id PK
    uuid contact_id FK
    text category
    boolean is_subscribed
    text source
    timestamptz updated_at
  }

  %% Relationships
  ACCOUNTS ||--o{ CONTACTS : has
  SALES_REPS ||--o{ ACCOUNT_OWNERSHIP : assigned
  ACCOUNTS ||--o{ ACCOUNT_OWNERSHIP : owned_by

  CAMPAIGNS ||--o{ CAMPAIGN_RUNS : has
  MODEL_RUNS ||--o{ AUDIENCE_MEMBERS : produces
  CONTACTS ||--o{ AUDIENCE_MEMBERS : selected
  ACCOUNTS ||--o{ AUDIENCE_MEMBERS : selected

  TEMPLATES ||--o{ TEMPLATE_VERSIONS : versions
  CAMPAIGN_RUNS ||--|| CAMPAIGN_RUN_CONTEXT : uses
  TEMPLATE_VERSIONS ||--o{ CAMPAIGN_RUNS : applied_in
  SENDING_POLICIES ||--o{ CAMPAIGN_RUNS : governs

  CAMPAIGN_RUNS ||--o{ MESSAGES : creates
  CONTACTS ||--o{ MESSAGES : receives
  ACCOUNTS ||--o{ MESSAGES : billed_to

  MESSAGES ||--o{ SEND_ATTEMPTS : retried_as
  MESSAGES ||--o{ MESSAGE_EVENTS : generates

  CONTACTS ||--o{ SUBSCRIPTION_PREFERENCES : preferences
