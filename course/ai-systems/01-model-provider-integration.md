# Integrating model providers into a backend

## Use a provider boundary

A provider SDK is a client library for an external model service. Keep it behind an application interface, such as generate_answer(input, options), rather than calling provider methods from every route. A provider adapter translates your application's request and response to a vendor-specific format. This makes tests easier and reduces coupling, while recognizing providers do not have identical capabilities.

Separate these concepts:
- user request: question, conversation, allowed documents;
- application policy: identity, limits, permitted model, retention;
- provider request: model ID, messages, temperature, tools, token limit;
- provider response: text or structured output, usage, finish reason, provider ID.

Do not let client input choose an arbitrary provider model or change privileged system instructions.

## Timeouts and retries

Network calls can time out, fail transiently, or return rate limits. Set explicit connection and overall deadlines. Retry only errors likely to recover, use exponential backoff with jitter, respect provider retry-after signals, and bound total attempts. Do not retry a non-idempotent action blindly. A model completion may be safe to repeat from an application perspective but can incur duplicate cost.

Circuit breakers or concurrency limits can stop a provider outage from consuming all app workers. Provide a clear temporary-unavailable response or queue the request if the task can be asynchronous. Record provider failure categories without exposing internal details to the user.

## Validate output

A model can return incomplete, malformed, or unexpected content. If the application expects a schema, parse and validate it. Treat unknown enum values and missing fields explicitly. If validation fails, a limited repair attempt may be useful, but repeated repair loops need cost and time limits. Never execute arbitrary model text as SQL, shell, or trusted markup.

For streaming output, send chunks as they arrive while handling disconnects, cancellation, provider errors after partial output, and a final completion event. Once text has been sent, the API cannot pretend the response was never partially delivered.

## Cost, concurrency, and privacy

Track input and output token usage, model, latency, retries, and estimated cost per request. Enforce per-user quotas and global concurrency caps. A token is a model's text unit; token counts differ from words and vary by language and encoding. Large history or retrieved context increases cost and may reduce relevance.

Send only information needed for the task. Review provider retention, data-use, region, and contractual settings for the chosen deployment. Do not assume a service's defaults match your privacy obligations.

## Worked flow

A request arrives; the API authenticates the user and checks quota; retrieval creates a bounded context; the service calls a provider through an adapter with a timeout; the response is validated; usage is recorded; a safe result is returned with request ID and sources. Each step can fail and should have a distinct test.

Questions: Why not retry every error? Invalid requests and permission failures will not recover, and repeated calls cost money. Why use adapters? They isolate provider-specific details while keeping your application contract stable.

Practice: implement a fake provider that can return success, timeout, rate limit, malformed output, and cancellation. Test each service outcome without live credentials.

Further reading: [FastAPI async dependencies](https://fastapi.tiangolo.com/tutorial/dependencies/) and [OpenTelemetry signals](https://opentelemetry.io/docs/concepts/signals/).

## Time budgets, retries, and output validation

An end-to-end request spends time in authentication, database, retrieval, provider generation, validation, and response transfer. Give each dependency a deadline that fits inside the overall request budget. A provider call should not wait longer than the client will wait. Track connection timeout and response timeout separately.

Retry transient network errors or provider throttling only when appropriate. Use exponential backoff with jitter and a maximum attempt/time budget. Invalid credentials and malformed requests will not recover by retrying. Repeated model calls cost money; retries must be visible in telemetry.

A provider adapter can normalize request IDs, usage, error categories, and stream events while keeping provider-specific capabilities explicit. Do not hide important differences in a lowest-common-denominator interface. Record provider and model revisions because behavior may change.

Validate structured output with both a parser and application rules. Schema validity means the shape is correct; it does not mean the content is true or authorized. One repair attempt can be useful, but cap it. On failure, return a safe error or queue for review.

Practice: implement a fake provider that can succeed, time out, rate-limit, return malformed JSON, and be cancelled. Verify cost/concurrency limits and safe client errors.
