# Project 3: production AI backend capstone

## Goal

Ship a small multi-user AI application with a persistent backend, a background workflow, and operational evidence. Choose a bounded use case such as internal policy search, support-ticket drafting, or document extraction.

## Required architecture

- API service with authentication, validation, versioned routes, and stable errors.
- PostgreSQL persistence for users/tenants, source metadata, jobs, and outputs.
- Object storage for uploaded source files.
- Queue and worker for ingestion or long model tasks.
- Model adapter with limits, timeouts, retries, and output validation.
- Permission-aware retrieval or tool workflow.
- Dockerized local environment and deployment instructions.
- Unit, integration, API contract, load, AI evaluation, and security tests.
- Logs, metrics, and traces with sensitive data redacted.
- Rate/token quotas, health checks, failure responses, and rollback plan.

## Deliverables

1. Architecture diagram and one request sequence diagram.
2. API documentation and setup instructions.
3. Threat model and data-retention note.
4. Evaluation set with answerable, unanswerable, malformed, and adversarial cases.
5. Test report and latency/cost measurements.
6. Demo and list of known limitations.

## Final defense

Explain why the database schema fits the data, how tenant isolation works, how duplicate jobs are handled, what happens when the model provider is unavailable, which system metrics you monitor, what evidence supports quality, and what would make you pause deployment.
