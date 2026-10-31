# 2. Spring Boot service architecture for model-backed APIs

A model call is one dependency inside a service. A good architecture keeps HTTP, business policy, data access, and provider integration understandable and testable. Do not place everything in one controller because a prototype is small.

## 2.1 Request flow

~~~mermaid
sequenceDiagram
  participant C as Client
  participant F as Security filter chain
  participant API as Controller
  participant U as Application use case
  participant DB as PostgreSQL
  participant M as Model adapter
  C->>F: HTTPS request and token
  F->>F: Validate issuer, audience, signature
  F->>API: Authenticated principal
  API->>API: Parse and validate DTO
  API->>U: Command plus actor identity
  U->>DB: Load permitted context
  U->>M: Bounded model request
  M-->>U: Typed result
  U->>DB: Persist result and usage
  U-->>API: Application response
  API-->>C: Stable JSON contract
~~~

The filter chain authenticates. The application use case authorizes and enforces business rules. The controller owns HTTP details. Do not infer tenant identity from caller-controlled request fields; derive it from validated identity and explicit policy.

## 2.2 A practical package structure

A small service can use feature-oriented packages:

~~~text
com.example.assistant
  api/                 controllers and transport DTOs
  application/         use cases, ports, orchestration
  domain/              business rules and value types
  infrastructure/
    model/             Spring AI/provider adapter
    persistence/       repositories and entity mapping
    messaging/         queue adapter and consumers
  config/              validated configuration and wiring
~~~

This is a boundary, not a requirement that every class has an interface. Add an interface where it isolates a variable or external dependency: model provider, clock, object storage, queue. Avoid abstracting trivial in-process helpers just to add files.

## 2.3 DTOs, validation, and stable errors

API request DTOs are public contracts. Validate size, required fields, allowed values, and options before expensive work. For model endpoints, make maximum prompt size, output budget, supported model capability, and file size explicit.

Return a stable error shape with machine-readable code, safe message, and correlation ID. Keep stack traces and provider details in controlled logs. Configure Spring validation and exception handling consistently; different controllers should not invent incompatible formats.

A short bounded interaction can return a response. If work may outlast the caller's useful wait, return an accepted response containing a job ID and expose status/result endpoints. Do not keep an HTTP connection open just to hide a durable workflow.

## 2.4 Configuration is part of the contract

Externalize provider URL/model, API credential, timeout, retry budget, concurrency, token limit, and feature flags. Validate configuration at startup, but do not require a live model request just to mark the application healthy.

Keep distinct signals:

- **Liveness:** process is alive and should not be restarted.
- **Readiness:** this instance can accept intended traffic.
- **Dependency health:** an external service may be degraded even while the API process is ready.

Use safe local defaults and explicit production values. Inject secrets from a secret manager or environment mechanism; never commit them or log a resolved configuration object.

## 2.5 HTTP client and blocking boundary

Spring has synchronous RestClient, non-blocking WebClient, and HTTP Service Clients. In conventional Spring MVC, RestClient is often simple for synchronous calls. WebClient fits when non-blocking composition or streaming is useful end to end. Do not select WebFlux merely because a provider is slow; the network remains slow.

Configure connection limits, connect/response timeouts, TLS, proxy behavior, and error conversion. A client timeout should fit inside the end-to-end deadline. Avoid retries both in the client and service unless their combined attempt budget is explicit.

## 2.6 Synchronous endpoint vs job resource

Use synchronous calls for short bounded work when the caller needs an immediate result. Use a job resource for batch processing, large files, long tool workflows, or work that must survive a disconnect/restart.

~~~http
POST /v1/summaries
Idempotency-Key: 2e4e...

202 Accepted
Location: /v1/jobs/job_123
~~~

Define job transitions, ownership, cancellation, result retention, and retry behavior. Cancellation may stop local waiting while an already-sent provider request still completes.

## Practice

Build an endpoint that validates a bounded prompt, authenticates a caller, passes an application command to a use case, and returns stable response/error contracts. Add a job endpoint for long work. Test malformed input, absent identity, provider timeout, and dependency degradation.

## Questions and answers

**Is controller-service-repository enough architecture?**  
It is a useful starting structure but does not automatically isolate provider concerns, tenant policy, or workflow state. Name boundaries where behavior or external dependencies vary.

**Should the controller build the prompt?**  
No. It should translate HTTP into an application command. Prompt policy, retries, access checks, and provider parsing belong in application/integration boundaries.

**Should a provider outage make every instance unready?**  
Usually no. Separate local readiness from dependency status and define controlled degradation at the use-case level.

**When should I add a queue?**  
When work needs durable execution, backpressure, independent scaling, or survival beyond the request. Queues add consistency and operations costs, so justify them with requirements.

## Mastery checkpoint

Trace a request through filter, controller, use case, database, provider adapter, and response. State what each layer knows and where timeout/authorization failures are translated.