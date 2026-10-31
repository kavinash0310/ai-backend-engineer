# 8. Spring observability and JVM performance

Observability answers: what happened, where did time go, did the request meet its objective, and what did it cost? AI systems need provider and retrieval signals alongside ordinary service telemetry, without turning private prompts into monitoring data.

## 8.1 Trace the full operation

~~~mermaid
flowchart LR
  A[HTTP span] --> B[Authorization/validation]
  B --> C[Database span]
  C --> D[Embedding/retrieval span]
  D --> E[Provider call span]
  E --> F[Output validation]
  F --> G[Persistence/response]
~~~

Use Micrometer and OpenTelemetry-compatible instrumentation. Propagate trace context across asynchronous messages so a request can be connected to a worker. A delayed job is not one long HTTP span; use job IDs or trace links.

Useful safe attributes include operation, provider/model family, prompt version, outcome category, retry count, token counts, retrieval candidates, and aggregate tenant cohort. Avoid raw user IDs as metric labels and avoid prompts, documents, keys, or personal data in spans.

## 8.2 Metrics and service objectives

Track throughput and error rates; p50/p95/p99 latency; provider timeouts/rate limits; in-flight work; queue depth/oldest age; retries/DLQ; DB pool/slow queries; tokens/cost; and evaluation quality by release slice.

An SLI is a measurement. An SLO is a target over a window. Define user-visible objectives, such as a percentage of interactive calls below a latency goal, and state whether provider outage counts as a failure. Quality objectives need a versioned evaluation, not a vague “good answer” dashboard.

## 8.3 Logs and privacy

Use structured logs with request/job/trace IDs and normalized error categories. Redact authorization, keys, prompts, chunks, and outputs by default. If content capture is needed for support, gate it, redact it, restrict access, limit retention, and document policy. Do not dump provider response bodies blindly.

## 8.4 Diagnose performance before tuning

A slow model call may dominate. Local bottlenecks include blocking calls on constrained pools, provider quota saturation, exhausted JDBC/HTTP pools, slow vector filters, PDF parsing, serialization, huge prompts, GC pressure, and queue consumers competing with interactive requests.

Measure end to end using realistic inputs and production-like limits. Tune one bottleneck at a time. A larger thread pool can hide queueing and overload the database rather than improve latency.

## 8.5 JVM and Spring signals

Understand heap/non-heap, allocation rate, GC pauses, thread count/state, class loading, CPU, and container memory limits. Distinguish a leak from an allocation spike due to parsing/buffering. Heap/thread dumps require controlled access because they may contain sensitive data.

Size Hikari or another JDBC pool against database capacity and transaction duration, not only request threads. Bound outbound HTTP pools. Add a concurrency limiter for expensive model calls if needed. Keep capacity for health checks and control operations.

## Practice

Trace a request across API, database, retrieval, provider, and output validation. Dashboard p95 latency, provider errors, queue age, and estimated cost. Simulate a slow provider and slow SQL; diagnose both without logging prompts.

## Questions and answers

**Should user ID be a metric tag?**  
Usually not. Unique labels cause high cardinality and privacy risk. Use aggregate dimensions; use protected IDs in logs/traces only where policy permits.

**Why measure provider latency separately from API latency?**  
It separates server queueing, database/retrieval, provider wait, and response handling. The user SLO still covers the whole path.

**Should provider outage make readiness fail?**  
Not automatically. Keep process readiness separate from dependency health and define degraded behavior.

**Can logs contain prompts?**  
Only under explicit data governance. Default to redacted metadata.

## Mastery checkpoint

Show an end-to-end trace and dashboard. Explain a bottleneck and one measured improvement, plus what remains unmeasured.