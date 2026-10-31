# 10. Senior system design, capstone, and job readiness

A senior AI backend engineer owns a system, not just a model call. They turn an ambiguous need into explicit quality, latency, cost, privacy, and availability constraints; make trade-offs visible; and help a team operate and improve the feature.

## 10.1 Capstone: permission-aware knowledge assistant

Build a backend that ingests authorized documents and answers questions with citations. The goal is an end-to-end system with measurable behavior, not an elaborate agent.

### Functional requirements

- Authenticated users upload/sync only documents they may manage.
- Ingestion runs as a durable job with visible status and safe errors.
- Users query only accessible documents.
- Responses cite source IDs/sections or abstain if evidence is insufficient.
- Deletion, replacement, and permission changes affect derived chunks under a documented policy.

### Non-functional requirements

Choose measurable goals for interactive p95 latency, availability, retrieval recall@k, groundedness, token budgets, tenant quotas, cost per successful answer, and deletion propagation time. Values depend on product; justify yours.

### Suggested architecture

~~~mermaid
flowchart TD
  Client --> GW[Gateway and identity]
  GW --> API[Spring Boot API]
  API --> APP[Application use cases]
  APP --> PG[(PostgreSQL: tenants, jobs, versions)]
  APP --> OBJ[(Object storage: source files)]
  APP --> OB[Transactional outbox]
  OB --> Q[Managed queue]
  Q --> W[Spring worker]
  W --> PARSE[Safe parser]
  PARSE --> EMB[Spring AI embedding adapter]
  EMB --> VEC[(Postgres vector index or vector store)]
  API --> RET[Authorized retrieval]
  RET --> VEC
  RET --> CHAT[Spring AI chat adapter]
  CHAT --> RET
  RET --> RESP[Validated answer and citations]
  API --> OBS[Metrics, traces, redacted logs]
  W --> OBS
  CHAT --> OBS
~~~

### Milestones

1. Contract: API examples, identity assumptions, classification, SLOs.
2. Thin slice: authenticated request, fake model, persistence, stable errors.
3. Security first: tenant checks and negative tests before real documents.
4. Provider: budgets, timeout, schema, safe failure, cost metadata.
5. Ingestion: validation, durable job, parser, version metadata, deletion.
6. Retrieval: vector + keyword candidates, filters, citations, bounded context.
7. Evaluation: versioned questions and retrieval/generation report.
8. Production: queue, retries, trace propagation, rate limits, deployment.
9. Failure drill: provider outage, duplicate message, revoked access, slow DB, bad migration.
10. Review: trade-offs, unresolved risks, next experiments.

## 10.2 Questions to answer before coding

- What user task is solved, and what may the model decide?
- Which facts come from source of truth and which are generated?
- What are latency, quality, availability, privacy, and cost targets?
- Can it be synchronous; otherwise what is the durable job state machine?
- Which actions are idempotent and how do retries avoid duplicate effects?
- How are users, tenants, documents, and tools authorized?
- What if the model is wrong, slow, unavailable, or changed?
- How are retrieval and answer quality measured separately?
- How does a user delete data, and when do copies/embeddings disappear?
- What evidence justifies provider, model, vector store, and queue choices?

## 10.3 Capacity estimation

State assumptions and measure rather than inventing precision. Estimate peak requests/second, token distributions, concurrency, provider rate limits, queue arrival/processing, storage growth, and DB connections. Little's Law gives a rough relation: concurrent work is arrival rate multiplied by average time in system. It is a starting point, not a load test; tail latency matters.

Example: 8 requests per second taking 4 seconds on average in the provider implies roughly 32 concurrent calls before burst/headroom. If quota is lower, queue, shed, or limit traffic. Increasing web threads does not add provider capacity.

## 10.4 Senior trade-off practice

Be ready to defend Spring MVC with bounded blocking calls vs reactive composition; synchronous response vs durable job; hosted vs self-hosted inference; PostgreSQL vector extension vs dedicated vector DB; single model vs routing/fallback; retry vs fail-fast/circuit breaker; cache vs freshness/privacy/invalidation; strict schema vs flexible text; and one service vs separate API/worker.

There is no universal answer. State constraints, alternatives, risks, operating cost, and evidence that would change your decision.

## 10.5 Portfolio and interview evidence

Prepare five-minute and thirty-minute walkthroughs. Show architecture/trust boundaries, API/schema, model/retrieval evaluation, p50/p95 latency, errors/concurrency/cost assumptions, security tests, a diagnosed failure, a measured trade-off, and limitations.

Do not claim “production ready” because it runs locally. Say exactly what was load-tested, deployed, monitored, and reviewed.

## Questions and answers

**What makes this senior-level instead of a demo?**  
Operational evidence: realistic identity/data boundaries, failure handling, measured quality/cost, deployment/recovery, security review, trade-offs, and clear communication.

**Do I need to train a model?**  
No. Many AI backend roles integrate models. You need enough literacy to select and evaluate one. Training/fine-tuning is role-specific and belongs in the theory companion.

**Should I use an agent framework?**  
Only if the task needs multi-step model-directed tool choice. Start with explicit workflows and allowlisted tools; frameworks do not remove authorization or idempotency.

**Can this repository make me senior?**  
It provides structure and practice artifacts. Seniority also requires ownership, judgment, collaboration, and learning from production incidents.

## Final readiness gate

Ask another engineer to review the capstone without narration. They should find trust boundaries, run it, understand the evaluation, see operational telemetry, trigger a safe failure, and locate rollback/runbook steps. Missing artifacts identify your next task.