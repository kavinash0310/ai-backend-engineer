# Project 1: Spring Boot model inference API

## Goal

Build a small Java HTTP service that accepts a bounded user request and returns a model response through a server-side provider adapter. This project teaches controlled model integration; it does not train a model.

## Recommended stack

Java and Spring Boot, Spring MVC, Spring AI or a provider-specific adapter behind an application interface, Bean Validation, PostgreSQL only if you persist requests, JUnit 5, and a local HTTP stub for provider tests. Select dependency versions using the current Spring AI compatibility matrix and BOM.

## Request path

~~~mermaid
sequenceDiagram
  participant C as Client
  participant S as Spring Security
  participant API as Controller
  participant U as Use case
  participant M as Model adapter
  participant P as Provider
  C->>S: HTTPS + access token
  S->>API: Authenticated principal
  API->>API: Validate size and options
  API->>U: Command + trusted actor
  U->>M: Prompt with explicit budgets
  M->>P: Provider API call with deadline
  P-->>M: Output and usage
  M-->>U: Typed outcome
  U-->>API: Safe result/error
  API-->>C: Stable JSON
~~~

## Build in milestones

1. Define a versioned request/response contract. Do not let callers set system prompts, credentials, arbitrary provider URLs, or unrestricted model names.
2. Create an application interface for text generation and a fake implementation. Keep provider-specific SDK types inside the adapter.
3. Add request validation and limits for input length, allowed capability/model, and maximum output tokens.
4. Integrate a provider with secret injection, connection and overall timeouts, bounded transient retries, and normalized error categories.
5. Validate generated output. If you request structured JSON, validate both schema and business meaning.
6. Add caller authentication and per-user or per-tenant rate/token quotas.
7. Record request ID, provider/model ID, duration, outcome, retry count, token usage, and estimated cost where allowed.
8. Add tests for normal result, invalid input, provider timeout, 429, 5xx, malformed output, missing credentials, and authorization.
9. Decide whether requests are ephemeral or persisted. If persisted, define retention, deletion, and access policy.

## API sketch

~~~http
POST /v1/generations
Authorization: Bearer ...
Idempotency-Key: 4c6f...

{
  "input": "Summarize this short text.",
  "maxOutputTokens": 300
}
~~~

Return a safe result with a request ID. For long work, return 202 Accepted and a job resource; do not hold a web request open indefinitely.

## Acceptance checklist

- [ ] Controller has no provider-specific parsing or retry loops.
- [ ] Input and output budgets are enforced server-side.
- [ ] Secrets are never returned, logged, or committed.
- [ ] Timeout, rate-limit, provider outage, and invalid output produce stable errors.
- [ ] Retrying a client request does not create duplicate side effects.
- [ ] Tests use a fake/stub provider; CI does not make uncontrolled paid calls.
- [ ] The README states exactly what is stored, sent to the provider, and retained.

## Questions for review

What limits prevent cost abuse? Can a caller select privileged behavior through a field? What happens if the provider finishes after the client timeout? Which logs are safe? How does the system prove the caller is allowed to use this feature?