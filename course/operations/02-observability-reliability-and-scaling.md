# Observability, reliability, and scaling

## Logs, metrics, and traces answer different questions

A log records an event with time and context. A metric is a numerical measurement over time, such as request count or latency. A trace shows the path of one request across services and contains spans for API handling, database queries, queue wait, retrieval, and model calls. Correlate these signals with a request or trace ID.

OpenTelemetry standardizes APIs and formats for traces, metrics, and logs. You still need to choose what to record and protect it. Never put raw prompts, access tokens, or private documents into logs without a justified need, controls, and retention plan. Prefer structured metadata such as model, latency, token count, status, retry count, and error category.

## AI-specific measurements

Track request rate, error rate, p50/p95 latency, queue age, provider timeout and rate-limit rates, input/output token usage, estimated cost, retrieval results, and evaluation outcomes. Monitor user-visible success, not only whether an HTTP response was returned. An endpoint can return 200 while generating an unsupported answer.

Use sampling for high-volume traces, while retaining error traces or selected diagnostic examples where privacy policy permits. Set alert thresholds tied to user impact and an owner who can respond.

## Reliability and time budgets

A request may spend time in authentication, database access, retrieval, model generation, and response streaming. Give each dependency a timeout that fits within an overall deadline. If every layer uses a long timeout, one request can occupy resources for minutes.

Retries help with temporary failures but increase load during an outage. Use bounded attempts, exponential backoff with jitter, and concurrency limits. Circuit breakers or bulkheads can isolate a failing provider. A fallback might be a queued response, cached answer with freshness label, or clear temporary error; it should not silently claim success.

## SLOs and scaling

A service-level indicator (SLI) is a measured property, such as successful requests under 2 seconds. A service-level objective (SLO) is a target over a window. Define it around user needs. Scaling may require more workers, database pool tuning, caching, or separate inference capacity. More replicas can increase database connections and provider spend, so tune all dependent limits together.

Load-test realistic inputs, including long contexts and concurrent streaming. Check cost and p95 latency, not only average throughput. Backpressure limits accepted work when queues grow. Autoscaling cannot fix a database bottleneck or runaway token usage.

## Questions and practice

Why are traces useful? They show where one request spent time across components. Why not retry indefinitely? It can amplify an outage and consume unbounded resources. What does a queue-age alert reveal? Work is accumulating faster than workers can process it.

Practice: trace an RAG request through API, database, vector retrieval, provider call, and response. Define one latency SLO, one cost alert, one provider-outage behavior, and one privacy-safe log format.

Further reading: [OpenTelemetry signals](https://opentelemetry.io/docs/concepts/signals/) and [Google production ML monitoring](https://developers.google.com/machine-learning/crash-course/production-ml-systems/monitoring).

## Define reliability from the user's view

An SLI is a measurement such as successful answer requests completed within a latency target. An SLO is the target over a window. An error budget is the allowed failure portion; if it is consumed, the team should prioritize reliability work. Count user-visible task success, not only HTTP 200 responses.

Trace a request across API, database, queue, retrieval, and provider. Record queue wait separately from worker execution. Metrics should use bounded labels such as route, status, and model; unique user IDs create high cardinality and can overwhelm a metrics backend. Keep sensitive prompts out of logs by default.

Capacity planning must coordinate API replicas, database pool sizes, queue worker concurrency, provider quotas, GPU limits, and memory. More API instances can multiply database connections and provider calls. Backpressure should slow or reject work before dependencies collapse. Autoscaling on CPU alone may miss a growing queue or provider throttle.

Practice: define one answer-success SLO, one cost alert, and a trace plan. Simulate a slow provider and document timeout, retry, queue, and user-visible behavior.
