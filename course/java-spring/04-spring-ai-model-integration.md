# 4. Spring AI model integration

Spring AI provides Spring-oriented abstractions and integrations for model providers, embeddings, vector stores, and related workflows. An abstraction reduces integration wiring; it does not make providers behaviorally identical or remove the need to understand model limitations.

## 4.1 Integration boundary

~~~mermaid
flowchart LR
  UC[Use case] --> POL[Policy and budgets]
  POL --> PORT[Model capability port]
  PORT --> AD[Spring AI adapter]
  AD --> PROVIDER[Provider API]
  PROVIDER --> AD
  AD --> VALID[Parse and validate output]
  VALID --> UC
  POL --> AUD[Usage, latency, outcome telemetry]
~~~

The application requests a capability such as “classify bounded text into this schema,” not a raw vendor SDK. Keep provider configuration, prompt templates, retry decisions, and response mapping in a deliberate integration layer.

## 4.2 Compatibility and a basic call

Before adding dependencies, check the current Spring AI release's Spring Boot compatibility and use its documented BOM. Confirm Java/provider versions, starter coordinates, model API, and migration notes. The example illustrates call shape; exact APIs can change with release.

~~~java
String answer = chatClient.prompt()
    .system("Use supplied material. If evidence is insufficient, say so.")
    .user(userQuestion)
    .call()
    .content();
~~~

Production code must not trust this string by default. Add request bounds, deadline, allowed model/capability, output validation, privacy decision, safe failure mapping, and telemetry. Keep secrets in external configuration.

## 4.3 Prompts are versioned behavior

A prompt template is part of application behavior and should be reviewed and versioned. It commonly combines:
- system/developer instruction: role, policy, response constraints;
- user input: the current task;
- context: retrieved or application-provided evidence;
- output contract: schema, allowed values, format.

Delimit untrusted text, but do not mistake delimiters for a security boundary. Track prompt version and model ID for reproducibility. Include only private data that is needed and permitted.

## 4.4 Structured output and validation

JSON mode or schema-constrained generation can improve shape, but parse success is not semantic correctness. Validate required fields, ranges, enumerations, cross-field rules, provenance, and authorization.

An extracted invoice amount must be a valid decimal within business limits. A category must be allowed. A proposed tool action needs independent authorization. Invalid output may fail safely or receive one bounded repair attempt; never loop without a limit.

## 4.5 Timeouts, retry, rate limits, fallback

Set one deadline across retrieval, prompt building, provider call, and response. Retry only transient failures. Respect provider retry guidance where appropriate, add backoff/jitter, and avoid nested retries that multiply attempts. A concurrency limiter or circuit breaker can prevent repeated failures from overwhelming the provider and local pools.

Fallback may mean smaller model, cached answer, partial response, queueing, or explicit unavailable response. It must preserve quality and safety requirements. Do not silently use an unevaluated model with unknown data policy.

## 4.6 Streaming

Streaming may improve perceived latency but partial output is hard to retract. Decide whether unvalidated output may reach a user. For high-risk structured tasks, buffer and validate before sending. For chat, define termination, provider error signaling, cancellation on disconnect, persistence of partial output, and moderation.

A slow client must not create unbounded buffers or keep provider work alive forever. Enforce maximum output tokens and duration.

## 4.7 Cost and quality budgets

Estimate cost from input/output tokens, model price, retries, embeddings, reranking, storage, and failures. Track provider/model, prompt version, token counts, duration, outcome, and allowed billing cohort. Avoid raw prompt telemetry and high-cardinality metric labels.

Choose a model using representative evaluation and constraints for groundedness, latency, cost, privacy, context size, tool behavior, and availability—not a leaderboard alone.

## Practice

Implement one capability using Spring AI behind an application port. Add fake responses for success and malformed output. Test timeout, rate limit, invalid schema, token budget, and cancellation. Record prompt/model versions in evaluation metadata.

## Questions and answers

**Does Spring AI remove the need for an adapter?**  
It can provide integration abstractions. You still need an application boundary for policy, feature behavior, budgets, normalized failures, and deliberate provider changes.

**If output is valid JSON, is it safe?**  
No. Validate meaning and authorization. A syntactically valid request for a protected action still requires normal business checks.

**Should each call retry three times?**  
No. Choose based on failure class, deadline, idempotency, provider limits, and latency target.

**Should every chat stream?**  
Only when the interface benefits and the system safely handles cancellation, partial output, backpressure, moderation, and telemetry.

## Mastery checkpoint

Explain which layer owns deadline, retry budget, output schema, privacy, and cost measurement along the path from application request to provider response.