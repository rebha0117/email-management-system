erDiagram
  CONTACTS {
    string contact_id PK
    string account_id
    string primary_email
    string country_code
    boolean is_active
    datetime created_at
  }

  ACCOUNTS {
    string account_id PK
    string account_name
    string country_code
    boolean is_active
    datetime created_at
  }

  CAMPAIGNS {
    string campaign_id PK
    string name
    boolean is_active
    datetime created_at
  }

  CAMPAIGN_RUNS {
    string campaign_run_id PK
    string campaign_id
    date run_date
    int week_of_month
    string model_run_id
    string template_version_id
    string status
    datetime created_at
  }

  CAMPAIGN_RUN_CONTEXT {
    string campaign_run_id PK
    string context_checksum
    datetime fetched_at
    string source
    string context_json
  }

  MODEL_RUNS {
    string model_run_id PK
    date scoring_week
    string model_version
    string s3_uri
    datetime created_at
  }

  TEMPLATES {
    string template_id PK
    string name
    boolean is_active
    datetime created_at
  }

  TEMPLATE_VERSIONS {
    string template_version_id PK
    string template_id
    int version_number
    datetime created_at
  }

  MESSAGES {
    string message_id PK
    string campaign_run_id
    string contact_id
    string account_id
    string to_email
    string current_status
    datetime queued_at
    datetime sent_at
    datetime last_event_at
  }

  SEND_ATTEMPTS {
    string attempt_id PK
    string message_id
    string ses_message_id
    string attempt_status
    datetime attempted_at
  }

  MESSAGE_EVENTS {
    string event_id PK
    string message_id
    string ses_message_id
    string event_type
    datetime event_ts
    datetime ingested_at
  }

  SUPPRESSIONS {
    string suppression_id PK
    string scope_type
    string scope_value
    string reason
    datetime created_at
  }

  ACCOUNTS ||--o{ CONTACTS : has
  CAMPAIGNS ||--o{ CAMPAIGN_RUNS : has
  MODEL_RUNS ||--o{ CAMPAIGN_RUNS : uses
  TEMPLATES ||--o{ TEMPLATE_VERSIONS : versions
  TEMPLATE_VERSIONS ||--o{ CAMPAIGN_RUNS : applied
  CAMPAIGN_RUNS ||--o{ MESSAGES : creates
  CONTACTS ||--o{ MESSAGES : receives
  MESSAGES ||--o{ SEND_ATTEMPTS : attempts
  MESSAGES ||--o{ MESSAGE_EVENTS : events
  CAMPAIGN_RUNS ||--|| CAMPAIGN_RUN_CONTEXT : has
