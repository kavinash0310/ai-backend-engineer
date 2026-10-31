# 1. Modern Java for AI backends

A Java AI backend spends much of its time waiting for remote systems: model providers, databases, vector stores, object storage, and queues. The challenge is to make each boundary explicit: what data enters, how much work is allowed, which failures are expected, and what the application can safely do next.

## Learning outcomes

You will model provider requests and responses with immutable types, separate transient from permanent failures, keep blocking work bounded, choose synchronous or asynchronous execution, and build a fake adapter for tests.

## 1.1 A useful mental model

~~~mermaid
flowchart LR
  A[Application use case] --> P[Java interface / port]
  P --> X[Provider adapter]
  X --> N[Network call with deadline]
  N --> R[Typed success or failure]
  R --> A
  T[Fake adapter in tests] --> P
~~~

The application describes the capability it needs, such as “generate a summary with these limits.” The adapter translates that request into a vendor SDK or HTTP call. This keeps vendor response types from spreading into controllers, database tables, and business rules.

## 1.2 Use types to make invalid states harder to express

Immutable request/response types help keep values stable across layers. Java records work well for small data carriers, but do not automatically validate fields or make nested collections immutable. Validate at the boundary and defensively copy collections callers might mutate.

~~~java
public record GenerationRequest(
    String requestId,
    String prompt,
    int maxOutputTokens,
    Duration timeout
) {
    public GenerationRequest {
        if (requestId == null || requestId.isBlank()) {
            throw new IllegalArgumentException("requestId is required");
        }
        if (prompt == null || prompt.isBlank()) {
            throw new IllegalArgumentException("prompt is required");
        }
        if (maxOutputTokens < 1 || maxOutputTokens > 2048) {
            throw new IllegalArgumentException("output budget is outside the allowed range");
        }
        if (timeout == null || timeout.isNegative() || timeout.isZero()) {
            throw new IllegalArgumentException("timeout must be positive");
        }
    }
}
~~~

Do not reuse one type as the public HTTP request, domain command, persistence entity, and provider payload just because their current fields look similar. Their validation and compatibility requirements differ.

Use Optional mostly for a return value that may be absent. Do not wrap every field and parameter in Optional. At an API boundary, validation should produce a clear client error; an absent database row may become a domain-level not-found result.

Generics help define a reusable contract when there are multiple implementations. They do not help when they hide important domain meaning. Keep provider exceptions and response classes inside the adapter.

## 1.3 Model outcomes, not one catch-all exception

An AI call fails for different reasons. Retrying every exception can turn a bad request into repeated cost or amplify an outage.

~~~java
public sealed interface GenerationResult
    permits GenerationResult.Success, GenerationResult.Rejected, GenerationResult.Unavailable {

    record Success(String text, String providerRequestId, Usage usage)
        implements GenerationResult {}

    record Rejected(String safeReason)
        implements GenerationResult {}

    record Unavailable(String category, boolean retryable)
        implements GenerationResult {}
}
~~~

Classify outcomes deliberately:

- **Caller/input error:** malformed or oversized input; do not retry.
- **Authorization/policy denial:** do not call the model; do not retry.
- **Rate limit or temporary network failure:** retry only if safe, within the overall deadline and retry budget.
- **Provider permanent error:** invalid credentials or unsupported model; alert and correct configuration.
- **Malformed/semantically invalid output:** validate it. A bounded repair attempt may help, but never trust output just because JSON parses.
- **Application defect:** preserve internal diagnostics, return a safe external error, and do not disguise it as a provider outage.

Do not log raw prompts or provider bodies by default. Logs can contain request/job IDs, model/provider identifiers, outcome category, duration, retry count, and allowed usage metadata.

## 1.4 Timeouts and concurrency are correctness

A client timeout means “stop waiting”; it does not prove that the provider stopped computing. If a tool call changes external state, the remote action may finish after the client gives up. Use idempotency or reconciliation.

Set limits at multiple layers: inbound size/deadline, model connect/read/overall timeout, maximum in-flight calls, bounded connection pools and queues, tenant quotas, token budgets, and a total deadline covering retrieval through response writing. An unbounded request queue converts overload into memory growth and extreme latency.

## 1.5 Threads, virtual threads, and reactive clients

Choose the execution model from the whole stack.

- Spring MVC with synchronous JDBC and a synchronous HTTP client is a reasonable baseline; the request thread waits during I/O.
- Virtual threads can make blocking I/O code cheaper to scale in supported runtimes. They do not create provider capacity, database connections, memory, or quota. Bound concurrent calls to scarce dependencies.
- WebFlux/Reactor can fit non-blocking systems, especially streaming or reactive composition. Calling blocking JDBC/SDK code on an event-loop thread can stall many requests.
- Parsing, embedding, reranking, and local inference may be CPU-bound. Virtual threads do not add CPU; use measured, bounded compute capacity.

Measure concurrency, downstream quotas, pools, and cancellation before changing architecture.

## Exercise

Create a TextGenerationPort interface and fake implementation. Test success, timeout, invalid request, rate limit, malformed response, and cancellation. The application service should depend on the interface rather than a vendor SDK. Add an overall deadline and prove retry attempts cannot exceed it.

## Questions and answers

**Why not call the provider SDK directly from a controller?**  
It couples the public HTTP contract to vendor details, mixes transport and policy, and makes isolated tests difficult. The controller should validate HTTP input and delegate to a use case.

**Should every provider error be retried?**  
No. Retry only classified transient errors, only if the operation is safe, and only within a deadline and attempt budget.

**Do virtual threads solve slow model APIs?**  
They can reduce the cost of waiting threads. They do not reduce provider latency or increase provider quota. You still need limits and an overload policy.

**Should prompt text be inside an exception message?**  
No. Exceptions may reach logs. Keep sensitive content out of messages; use a correlation ID and redacted diagnostics.

## Mastery checkpoint

Draw the flow from use case through provider adapter and back. Explain retryability, deadline enforcement, what a timeout means for remote side effects, and how a test replaces the real provider.