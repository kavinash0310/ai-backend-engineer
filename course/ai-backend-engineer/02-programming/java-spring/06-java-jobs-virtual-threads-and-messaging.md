# 6. Long-running AI work: Java jobs, virtual threads, and messaging

Model calls can take seconds; document ingestion and batch processing can take minutes. When work must outlive a web request or restart, represent it as durable workflow state rather than keeping it inside a request thread.

## 6.1 Synchronous call or durable job?

Keep short, bounded work synchronous when the user needs an immediate answer and the total deadline fits the product SLO. Use a job for ingestion, bulk classification, reports, or workflows that must survive disconnects and restarts.

A job API defines states, ownership, cancellation, retention, and retries. Example state machine: accepted -> queued -> running -> succeeded, failed, or cancelled. Store attempt number and safe error category, not prompts or secrets by default.

## 6.2 Queue is delivery; database is business state

Most queues offer at-least-once delivery: a message may be delivered again after acknowledgement loss or worker crash. Assume handlers run more than once. Exactly-once business effects require idempotency or deduplication, not just a broker setting.

~~~mermaid
sequenceDiagram
  participant API as Spring API
  participant DB as PostgreSQL
  participant O as Outbox publisher
  participant Q as Queue
  participant W as Worker
  API->>DB: Transaction: create job and outbox row
  O->>DB: Claim unsent outbox row
  O->>Q: Publish job ID
  O->>DB: Mark published
  Q->>W: Deliver job ID, possibly again
  W->>DB: Claim job if eligible
  W->>W: Process idempotently
  W->>DB: Save result and completed state
  W->>Q: Acknowledge
~~~

Without an outbox, an API can commit a job then crash before publishing, or publish then roll back its row. An outbox records the business change and event intent in one SQL transaction; a publisher retries delivery.

## 6.3 Idempotency and side effects

Use an idempotency key or unique business key to detect duplicate submissions. Make state transitions conditional on expected status/version. For external tool calls, pass an idempotency key when supported and persist action results for reconciliation.

A worker should:
1. Claim a job with a lease or lock.
2. Verify it is current and authorized.
3. Execute a bounded attempt.
4. Persist progress/result durably.
5. Record retryable or terminal failure.
6. Acknowledge only after state is saved.

A lease allows recovery after a worker dies. But an old worker may continue after its lease expires. Use fencing/version tokens or idempotent effects to prevent concurrent duplicates.

## 6.4 Retry, dead-letter, and replay

Classify failures. Invalid content and authorization denial are terminal. Provider rate limit or transient network failure may be retryable. A malformed tool instruction may need review rather than an identical repeat.

Use bounded exponential backoff with jitter, maximum attempts, and maximum age. A dead-letter queue or failed-job table supports inspection, alerting, and controlled replay. Replay must apply current authorization and record whether prompt/model/index version has changed.

## 6.5 Virtual threads and reactive messaging

Virtual threads can simplify blocking worker code when work is I/O-heavy and the runtime supports them. They do not replace concurrency limits, JDBC pool sizing, provider quotas, or backpressure. Many cheap threads can still exhaust database connections or downstream capacity.

Reactive messaging can fit a service already designed around non-blocking streams. Avoid blocking JDBC or SDK calls on event loops without deliberate scheduling. Choose one understandable execution model and measure it.

## Questions and answers

**Does Kafka guarantee once-only processing?**  
Broker semantics do not make effects across application databases and external systems exactly once. Use durable state and idempotency.

**Should retry repeat the whole workflow?**  
Not always. Persist step state for expensive or side-effecting work. Retry the failed idempotent step or resume from a checkpoint under a documented version policy.

**Can a database transaction stay open during a model call?**  
Avoid it. Calls are slow and unpredictable; long transactions retain locks and connections. Save state, release the transaction, call the provider, then write the result with version checks.

**When is a queue unnecessary?**  
When work is short/bounded and losing or retrying it after process failure is acceptable. Queues add monitoring, schema evolution, replay, and consistency work.

## Practice and mastery checkpoint

Build an embedding job with an outbox. Kill the worker after intermediate progress, redeliver the message twice, and prove no duplicate business effect. Document what cancellation means after the provider request has started.