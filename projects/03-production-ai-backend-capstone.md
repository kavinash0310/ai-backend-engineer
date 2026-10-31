# Project 3: production AI backend capstone

## Goal

Ship a bounded multi-user AI application with persistent data, background processing, measurable model quality, and operating evidence. Choose a realistic problem such as internal policy search, support-ticket drafting, or document extraction. Implement the backend in Java/Spring Boot.

## Required architecture

- Spring Boot API with authentication, validation, versioned routes, and stable errors.
- PostgreSQL persistence for tenants, source metadata, jobs, and outputs.
- Object storage for uploaded source files.
- Durable queue and worker for ingestion or long model tasks.
- Spring AI/provider adapter with time/token/concurrency limits and output validation.
- Permission-aware retrieval or allowlisted tool workflow.
- Dockerized local environment and one cloud deployment.
- Unit, integration, contract, security, failure, load, and AI evaluation evidence.
- Redacted logs, metrics, traces, quota controls, health behavior, runbook, and rollback plan.

## Build sequence

1. **Requirements:** name the user, task, data sensitivity, latency/quality/cost goals, and what the model may decide.
2. **Architecture:** diagram services, stores, trust boundaries, request/job flows, and external dependencies.
3. **Vertical slice:** authenticated Spring endpoint, fake provider, stable errors, and persistence.
4. **Data/security:** migrations, tenant isolation, secret handling, retention, threat model, negative tests.
5. **AI integration:** provider, prompt/model version, deadlines, structured response, token and cost metadata.
6. **Durable work:** outbox or equivalent consistency design, queue, idempotency, retries, DLQ/manual replay.
7. **RAG/tool workflow:** filtered retrieval/citations or strictly allowlisted and separately authorized tools.
8. **Evaluation:** versioned cases for normal, adversarial, unanswerable, stale, malformed, and permission-denied behavior.
9. **Operations:** container, cloud identity/secrets, traces, dashboard, alerts, SLO, backup/restore, deployment/rollback.
10. **Failure drill:** simulate provider outage, queue duplicate, slow database, unauthorized access, and worker termination.
11. **Review:** report measurements, trade-offs, limitations, and next experiments.

## Deliverables

1. Architecture and request-sequence diagrams.
2. API documentation, local setup, and reproducible deployment instructions.
3. Threat model and data-retention/deletion note.
4. Test report and versioned AI evaluation results.
5. Latency, throughput, error, resource, and cost measurements with workload assumptions.
6. Dashboard screenshots or query definitions, alerts, runbook, and rollback procedure.
7. Demo and a list of known limitations and untested assumptions.

## Final design defense

Explain why the schema fits the data; how tenant isolation applies to direct reads and semantic search; how duplicate jobs are handled; what happens when the provider is unavailable; how you measure quality and cost; which metrics page the on-call engineer; how deletion propagates; and what evidence would make you pause rollout.

## Portfolio quality

State exactly which components are deployed and load-tested. Never claim that RAG guarantees correctness or that a local demo is production-ready. Show what you observed, what you measured, what remains unknown, and which trade-off you would revisit first.