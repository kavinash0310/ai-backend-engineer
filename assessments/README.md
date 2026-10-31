# Assessment and backend-engineer readiness

This assessment checks whether you can build and explain an AI-backed service. It is not a certification or job guarantee. Score evidence you can show in code, tests, diagrams, and project documentation.

## Readiness levels

### Level 1 — Foundations

You can write Python functions, use Git, run a local process, explain a basic HTTP request, and write simple SQL.

### Level 2 — Backend builder

You can build a FastAPI service with validation, documented responses, tests, PostgreSQL persistence, migrations, and authentication/authorization.

### Level 3 — AI integration engineer

You can call a model through a controlled adapter, handle timeouts and rate limits, validate structured output, track token/cost budgets, and build a permission-aware RAG flow.

### Level 4 — Production-ready project

You can containerize and deploy the system, trace requests, monitor latency and failures, enforce quotas, test security boundaries, document recovery, and explain limitations.

## 100-point evidence rubric

| Area | Points | Evidence |
| --- | ---: | --- |
| API design and validation | 10 | OpenAPI contract, bounded inputs, stable errors |
| Data model and persistence | 10 | Schema, constraints, transactions, migrations |
| Identity and authorization | 15 | Authentication plus per-object and per-tenant checks |
| AI integration | 10 | Provider adapter, timeout, retries, output validation |
| RAG or AI workflow quality | 10 | Permission-aware retrieval, citations, evaluation cases |
| Testing | 10 | Unit, integration, contract, AI evaluation, failure tests |
| Security and privacy | 10 | Threat model, secret handling, retention, limits |
| Reliability and operations | 10 | Logs, metrics, traces, health, rollback, SLO |
| Deployment and reproducibility | 5 | Container, configuration, clear setup |
| Communication and limitations | 10 | Architecture, decisions, demo, honest failure analysis |

A high score requires working evidence across all areas. A project that only returns a model response cannot earn production-readiness points for access control, reliability, or evaluation.

## Self-review prompts

1. Can a user read another tenant's document by changing an ID?
2. What happens if the model provider takes 60 seconds or returns invalid data?
3. Can a duplicate queue delivery create duplicate side effects?
4. Which prompts and outputs are logged, and why?
5. Can you measure p95 latency and cost per successful task?
6. Does the RAG test distinguish retrieval failure from generation failure?
7. How does a deploy roll back without corrupting the database?
8. What evidence supports your quality claim, and what remains unknown?

Record the answers in each project README and in [PROGRESS.md](../PROGRESS.md).
