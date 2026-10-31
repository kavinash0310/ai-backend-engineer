# Senior AI backend engineer readiness assessment

Use this rubric to review evidence from a Java/Spring project. It is a learning tool, not a certification or job guarantee. Score what another engineer can verify in code, tests, traces, deployment, and your explanation.

## Readiness levels

### Level 1 — AI-aware backend engineer
You explain inference, tokens, context, embeddings, uncertainty, and why model output requires validation. You can build a conventional Spring API and persist data.

### Level 2 — AI integration engineer
You integrate a model through Spring AI or a controlled adapter, bound time/cost, validate structured responses, test provider failures, and explain model selection.

### Level 3 — Production AI backend engineer
You ship tenant-safe RAG or another AI workflow with durable processing, evaluation, security controls, tracing, and deployment/recovery evidence.

### Level 4 — Senior-level evidence
You lead design from requirements through trade-offs, capacity estimates, evaluation gates, production operation, incident response, and reviewable documentation. Actual seniority also depends on ownership and experience.

## 100-point evidence rubric

| Area | Points | Evidence |
| --- | ---: | --- |
| Java/Spring architecture and API | 10 | Clear boundaries, validation, stable errors, versioned contract |
| Data modeling and consistency | 10 | Schema, migrations, constraints, transaction choices, indexes, recovery |
| Identity, tenant isolation, security | 15 | OAuth2/JWT, object checks, threat model, negative tests |
| Model integration and failure policy | 10 | Spring AI/provider seam, timeout, bounded retry, output validation, quotas |
| Retrieval/workflow quality | 10 | Evaluation; permission-aware RAG/citations or workflow correctness |
| Testing and evaluation | 10 | Unit, integration, contract, security, failure, load, quality tests |
| Privacy and responsible operation | 10 | Classification, redaction, retention, provider policy, abuse controls |
| Reliability and observability | 10 | Traces/metrics/logs, SLO, runbook, failure drill, rollback |
| Deployment and reproducibility | 5 | Container, config/secrets, automated build, cloud setup |
| Senior communication and trade-offs | 10 | Architecture decisions, measured alternatives, demo, limitations |

A model response in a browser proves almost none of these areas. Claims need concrete artifacts.

## Review questions

1. Can a user cross tenant boundaries through direct IDs, semantic search, citations, or tools?
2. What is the deadline if the provider takes 60 seconds?
3. Can retries duplicate a database or external side effect?
4. Which content is sent, stored, logged, retained, or deleted?
5. Can you measure p95 latency and cost per successful task?
6. Does evaluation distinguish retrieval failure from generation failure?
7. What happens to jobs during restart, provider outage, or queue redelivery?
8. Can you demonstrate restore/rollback rather than only describe it?
9. Which assumptions have not been tested under realistic load?
10. What result would change your technology choice?

Record evidence in [PROGRESS.md](../PROGRESS.md).