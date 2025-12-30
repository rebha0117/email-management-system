# email-management-system

# Email Management System

This project documents the architecture and data model for an
email management system used to send weekly sailing announcements, freight rates etc.
to customers (~70k+ emails/week).

## Goals
- Reliable bulk email sending via AWS SES
- Compliance (unsubscribe, bounce, complaint handling)
- Clear operational vs analytics separation
- Scalable event-driven design

## Tech Stack
- Postgres (operational DB)
- AWS SES / SNS
- AWS Lambda
- Redshift (analytics)
- CloudWatch (logging & monitoring)
