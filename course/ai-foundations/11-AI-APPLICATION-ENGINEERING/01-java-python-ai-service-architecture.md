# Java and Python architecture for AI applications

## Goal

Use the strengths you already have in Java/Spring while learning Python for data and model ecosystems. The boundary should follow ownership, latency, deployment and team needs—not a rule that every AI component must use Python or every production service must use Java.

## 1. Three common deployment shapes

### Shape A: Java application calls a hosted model API

Spring Boot owns authentication, domain logic, provider abstraction, quotas, persistence, streaming and product response. It calls an external inference service. This is often the fastest path to a useful product, but introduces provider dependency, network failure, data-handling and cost concerns.

### Shape B: Java application calls a Python model service

A Python worker owns Python-specific preprocessing, model loading and inference. Java remains the product/API service. A versioned HTTP or gRPC contract separates deployment and scaling. This is useful when the model ecosystem is Python-native or requires a separate accelerator runtime. It adds a network hop and another service to operate.

### Shape C: Single Python service

A Python API owns model and product behavior. This can be reasonable for a focused service or Python-first team. Do not choose it solely because a model library is written in Python; preserve the operational capabilities your product needs.

~~~mermaid
flowchart LR
  C[Client] --> G[Spring Boot API]
  G --> A[Auth, policy, quota]
  A --> O[Orchestration]
  O --> R[Retrieval / database]
  O --> P[Provider adapter]
  P --> H[Hosted model API]
  P --> W[Python inference service]
  W --> M[Model runtime / accelerator]
  G --> T[Telemetry and audit]
~~~

## 2. Define a versioned service contract

Model inputs and outputs change as systems evolve. Define:
- endpoint or RPC name and contract version;
- request ID and idempotency semantics;
- tenant/user identity propagated through authenticated context;
- text and token limits;
- input schema and validation rules;
- deadline and cancellation semantics;
- output schema, error categories and model version;
- privacy and retention expectations;
- compatibility and deprecation plan.

Never forward an arbitrary inbound authorization header to a model vendor. Authenticate each hop explicitly, use service identity, and pass only the minimum context needed. Avoid making a backend endpoint an unrestricted proxy to a model provider.

For a Python worker, typed request/response schemas and explicit health/readiness endpoints make the boundary easier to operate. Add compatibility tests at the contract boundary when the repository asks for tests; this handbook describes what to design, not tests run against an existing project.

## 3. Spring Boot responsibilities

Spring can own:
- user and tenant authentication/authorization;
- domain validation and database transactions;
- rate limits, quotas and cost budgets;
- provider or worker client lifecycle;
- bounded concurrency, timeout and retry policy;
- streaming adaptation to the client;
- audit metadata and tracing;
- persistence and job orchestration.

Use a provider interface to isolate vendor-specific request/response types. Keep provider selection explicit and observable. A “fallback provider” can change answer quality, retention, region, structured-output support and safety behavior. Treat it as a policy decision and test it.

Spring AI provides model-client abstractions and integrations within Spring applications; version compatibility changes. Check its official getting-started page and match the Spring AI BOM with the Spring Boot version already used by the service. Do not copy a dependency version from an old example without checking current compatibility.

## 4. Python model-worker responsibilities

Python may own:
- tokenization and preprocessing tied to a model;
- loading weights and managing device placement;
- batching and inference scheduling;
- model-specific post-processing;
- offline evaluation and experiment code;
- framework-native runtime integrations.

The worker should not become an unbounded backdoor around Java's authorization or data policy. Pass a minimal, already-authorized payload. Return model output plus version/usage/finish metadata, not secrets or hidden implementation state.

CPU/GPU inference can block the event loop if executed directly in an async request handler. Use the model server's scheduling/runtime or a bounded worker pool. Bound queue size. Define what happens under overload and when the Java caller cancels.

## 5. Timeouts, retries and duplicate work

Set an overall request deadline at the client edge and allocate smaller budgets to internal stages. Propagate remaining time, not just the original full timeout. Use bounded retries with jitter for transient network errors. Do not retry a non-idempotent action or a generation that already produced a user-visible stream without clear semantics.

Idempotency is especially important for user-visible jobs and billable provider calls. A client retry after a network timeout may arrive even though the first request succeeded upstream. Store request/job state or use provider idempotency support where available.

A circuit breaker protects a failing dependency by refusing or limiting calls temporarily. A bulkhead limits how many resources one dependency can consume. These patterns require operationally meaningful states and metrics; they are not magic annotations.

## 6. Streaming and asynchronous work

For interactive chat, server-sent events or WebSockets can stream deltas. Keep the model-provider stream behind your domain contract. Define:
- event schema and sequence IDs;
- keepalive and heartbeat behavior;
- cancel propagation;
- client disconnect behavior;
- partial output safety;
- error event shape;
- completion and usage metadata;
- reconnect behavior (resume, restart or fail).

For long-running document ingestion or batch classification, use an asynchronous job model: create job, return job ID, process through a queue, store state, expose progress/results, and make transitions idempotent. Do not hold an HTTP request open indefinitely.

## 7. Data contracts and schema evolution

Represent inputs explicitly: text, locale, context references, model/task option, output schema. Avoid giant free-form JSON objects where unsupported fields silently change behavior. Validate types, ranges, allowed enums and size before spending inference resources.

For model outputs, treat structured generation as untrusted. Parse and validate output against a schema; a model's claim that JSON is valid is not validation. If output is invalid, define one bounded repair attempt or a safe error path rather than a recursive retry loop.

## 8. Security boundary

- The client cannot choose arbitrary provider URL, model path or tool name.
- Credentials are fetched from a secret manager and scoped narrowly.
- Authorization occurs before retrieval or inference.
- Tenant IDs come from trusted authentication state, not user text.
- Tool execution has server-side allowlists and argument validation.
- Prompt and response logs use privacy minimization.
- Model output is escaped/encoded for its eventual context (HTML, SQL, shell, etc.).
- Dependencies, model files and container images have provenance checks.

Never directly execute generated Python, SQL, shell commands or browser actions because a model returned them. Convert desired actions to typed proposals, validate against policy and permissions, then execute through restricted code.

## 9. Observability across service boundaries

Propagate a trace ID and record spans for authentication, retrieval, model provider, worker queue, inference, validation and storage. Useful dimensions include request class, model version, token counts, latency phase and status. Keep high-cardinality user text out of metric labels and raw personal content out of default traces.

Separate infrastructure metrics (CPU, memory, GPU utilization, queue depth), application metrics (timeout, retry, request rate), and quality metrics (groundedness, task success, human correction). Quality may require sampled review or delayed labels. Avoid pretending technical success is semantic success.

## 10. Deployment choices and trade-offs

| Choice | Strength | Cost/risk to plan |
|---|---|---|
| Hosted inference API | Fast to start; provider runs hardware | Network, spend, retention, limits, vendor dependence |
| Self-hosted model | Control over runtime and data path | GPU capacity, upgrades, security, operations, quality trade-off |
| Separate Python worker | Isolates model runtime and scaling | Network hop, contract, deployment and incident complexity |
| In-process model | Fewer service boundaries | Runtime coupling, memory pressure, packaging and scaling constraints |

Start with the smallest architecture meeting real requirements. Add a worker when measured needs or libraries justify the boundary. Do not distribute a toy system into microservices before it needs independent scaling or ownership.

## Questions and answers

**Q: Must my AI backend be written in Python?**  
A: No. Java can orchestrate inference through provider APIs or a model service. Python is valuable for experiments and model-native code, but a polyglot boundary should solve a concrete deployment or ecosystem need.

**Q: Should I call the model directly from a controller?**  
A: Keep controllers thin. Put policy, budget, retrieval and provider orchestration behind application/domain services so those rules can be reasoned about and changed independently.

**Q: Can I use the model to decide whether the user is authorized?**  
A: Authorization must be enforced by trusted backend policy. A model may help classify intent, but it is not an authority source.

**Q: How do I support a provider switch?**  
A: Use a stable internal request/response abstraction, retain provider-specific capability metadata, run evaluations against candidate providers, and control rollout. A common interface does not make model behavior equivalent.

## Practice

1. Draw a sequence diagram for Java API → retrieval → Python worker → response, including deadlines and cancellation.
2. Define error categories for validation, authentication, quota, timeout, upstream throttling and invalid model output.
3. Design idempotency behavior for a generation call that times out after the provider may have completed.
4. Write a provider-neutral schema with a model version and token usage fields.
5. Decide whether a simple product should start with hosted inference or a Python worker, and justify using data, scaling and operational constraints.

## Completion check

You can assign responsibilities across Java and Python, define a stable contract, manage timeouts/retries/streaming, and keep model output within backend authorization and execution controls.