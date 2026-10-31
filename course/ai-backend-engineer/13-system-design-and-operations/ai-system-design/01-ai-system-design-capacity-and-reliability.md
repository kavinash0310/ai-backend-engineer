# AI system design: capacity, latency and reliability

## Design goal

An AI architecture must meet user requirements under uncertain, expensive and probabilistic dependencies. Start with product behavior, data risk and measurable SLOs; then choose components. Do not begin by drawing a vector database or agent framework.

## 1. Clarify requirements

For an AI knowledge assistant, ask:
- Who is the user and how is identity established?
- What data may the system retrieve and where are permissions enforced?
- Does the user need an answer, a citation, a decision or a workflow action?
- What is a correct answer and what should happen with insufficient evidence?
- What are expected corpus size, update frequency, request rate and peak concurrency?
- What are TTFT, end-to-end latency, availability and freshness targets?
- What are data retention, privacy and region requirements?
- What is the cost budget and acceptable degraded behavior?

Separate functional requirements from quality attributes. A model producing text is one step, not the entire system.

## 2. Reference architecture

~~~mermaid
flowchart LR
  U[User] --> API[Gateway and Spring API]
  API --> ID[Identity, tenant policy, quotas]
  ID --> ORCH[Request orchestrator]
  ORCH --> RET[Hybrid retrieval]
  RET --> IDX[(Lexical/vector index)]
  RET --> META[(Metadata and ACL store)]
  ORCH --> LLM[Inference adapter]
  LLM --> PROVIDER[Hosted model or worker]
  API --> JOB[Ingestion job service]
  JOB --> Q[Queue]
  Q --> W[Parser/embed/index workers]
  W --> IDX
  W --> META
  API --> OBS[Traces, metrics, audit]
~~~

This is a logical architecture. A first version may combine API/orchestrator/retrieval in one deployable service and use managed model inference. Split services only when independent scaling, runtime isolation, ownership or resilience justifies the cost.

## 3. Capacity estimation

State assumptions before calculating. Illustrative formulas:

- daily model input tokens = requests/day × average input tokens;
- daily output tokens = requests/day × average output tokens;
- mean in-flight requests ≈ arrival rate × average end-to-end latency (Little's law);
- queue delay grows when sustained arrival rate approaches or exceeds service capacity;
- vector storage lower bound = vector_count × dimensions × bytes/value, before index and metadata overhead.

Example assumptions: 20,000 questions/day, 2,000 input tokens and 300 output tokens each:
- input volume ≈ 40 million tokens/day;
- output volume ≈ 6 million tokens/day.
Peak load can be much higher than the daily average, so collect a time-of-day profile.

If arrival rate is 4 requests/second and mean end-to-end time is 5 seconds, average in-flight work is about 20 requests. This does not describe burst tail or model GPU batch behavior, but helps reason about concurrency and memory.

For embeddings, estimate documents/day × chunks/document × dimensions × bytes plus metadata/index overhead. Corpus growth, re-embedding on model upgrade, backups and deletion propagation are part of capacity planning.

## 4. Latency budget

Break the end-to-end target into stages:
- gateway/authentication;
- query preprocessing;
- lexical/vector retrieval;
- reranking;
- prompt assembly;
- model prefill/TTFT;
- token decode;
- response validation and delivery.

For target p95 of 8 seconds, assigning all 8 seconds to the provider call leaves no budget for queueing or retrieval. Reserve margins and measure actual stage percentiles. Token count and model choice can change latency significantly.

Streaming can reduce time to first visible content while total completion takes longer. Product SLO should distinguish TTFT, completion and cancellation.

## 5. Availability and failure modes

Dependencies fail independently: model provider throttling/outage, vector index unavailability, stale ACL metadata, parser failure, queue backlog, database pool exhaustion, network partition or secret rotation error.

For each, define:
- detection signal and threshold;
- user-visible behavior;
- retry/timeout policy;
- fallback if safe;
- recovery/replay process;
- data consistency implications;
- owner and runbook.

Fallback can be lexical results with source links, a cached answer when freshness/permissions allow, or a clear temporary-unavailable message. Do not invent an answer when evidence/model service is unavailable.

## 6. Backpressure and overload control

Letting every request wait in an unbounded queue is not resilience. Enforce:
- maximum request bytes/tokens;
- per-tenant and global concurrency;
- bounded queue and queue-age deadline;
- model generation length;
- retrieval/reranker budgets;
- retry budget;
- cancellation;
- overload rejection with retry guidance.

Use bulkheads for dependency pools and circuit breakers for repeated upstream failure. Backpressure should preserve the most useful work and avoid one tenant exhausting capacity.

## 7. Consistency and idempotency

A document update touches source metadata, chunks, vectors and lexical indexes. Partial failure can leave stale representations. Use versioned ingestion state, idempotent job IDs, retryable stages, tombstones/deletion propagation and reconciliation.

A generation request can time out after the provider completed. Persist request state and idempotency keys where supported; do not blindly repeat side effects. Read models and indexes may be eventually consistent—document the time until updates/deletions become effective and how permissions are handled during that interval.

## 8. Cost and model routing

Cost per task includes input/output tokens, embeddings, reranking, storage, network, compute, observability and human review. Route only after defining capability and policy needs. A cheaper model may need more retries or human corrections and can cost more per successful task.

Useful levers:
- remove irrelevant prompt context;
- cache deterministic or safely reusable work with correct tenant/version keys;
- cap output and tool calls;
- batch offline work;
- use smaller model for validated simple tasks;
- avoid repeated embeddings on unchanged content;
- set per-tenant budgets and alert before exhaustion.

Any model router needs evaluation for quality differences, data handling and safe fallback.

## 9. SLOs and dashboards

Potential service indicators:
- availability and successful completion;
- p50/p95/p99 end-to-end and TTFT;
- queue depth/age and rejection rate;
- model timeout/throttling and retry rates;
- retrieval latency and index freshness;
- request/token volume and cost;
- citation validity, groundedness and abstention on a sampled evaluation set;
- security events and unauthorized retrieval attempts.

Do not use raw user text as a metric label. Quality labels may arrive later; maintain a separate evaluation pipeline.

## 10. Architecture review format

For a design interview or ADR:
1. requirements/assumptions and non-goals;
2. API and data ownership;
3. diagram and critical request sequence;
4. capacity estimate with units and peak assumptions;
5. consistency and update/delete path;
6. security and tenant authorization;
7. failure modes/fallback/rollback;
8. SLO, telemetry and evaluation;
9. trade-offs and staged implementation.

A good design names the uncertain assumptions and how to measure them.

## Questions and answers

**Q: Why use Little's law?**  
A: For a stable system, average in-flight work relates to arrival rate times average time in system. It gives a first concurrency estimate, not a complete tail-latency model.

**Q: What is the most important capacity number for LLM service?**  
A: There is no single number. Prompt/output tokens, peak arrival profile, latency target, context lengths, concurrency, model/runtime and KV cache all matter.

**Q: Should retrieval failure fall back to the language model's memory?**  
A: Only if the product explicitly permits an ungrounded answer and labels it correctly. For evidence-required tasks, report unavailable/insufficient evidence instead.

**Q: When should I make the Python model server a separate microservice?**  
A: When runtime, accelerator, scaling or ownership needs justify isolation. Include network failure, auth, contract and operations costs in the decision.

## Practice

1. Estimate token volume for your own expected daily and peak usage; show assumptions and units.
2. Draw a 10-second latency budget across five stages, leaving margin for variability.
3. Use Little's law for 3 requests/sec and 4 sec mean latency; discuss why this is not a queue limit.
4. Design a document-delete flow across database, vector/lexical index and caches.
5. Write fallback behavior for provider throttling, retrieval outage and queue saturation.

## Completion check

You can clarify requirements, estimate first-order capacity, budget latency, constrain overload, plan updates/deletion, define SLOs and defend an AI architecture trade-off.