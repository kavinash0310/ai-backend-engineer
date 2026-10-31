# AI backend engineering glossary

## Backend and networking

- **API:** A contract that lets software systems request data or actions from each other.
- **Backend:** Server-side software that validates requests, applies business rules, stores data, and calls dependencies.
- **Client:** A program or device that sends requests to a service.
- **Process:** A running program with memory and operating-system resources.
- **Port:** A numbered network endpoint used by a process.
- **DNS:** The system that resolves domain names to network addresses.
- **TCP:** A transport protocol that provides ordered, reliable byte streams.
- **TLS:** A protocol layer that encrypts traffic and authenticates a server or peer.
- **HTTP:** A request/response protocol used by web APIs.
- **JSON:** A text serialization format for structured data.
- **REST:** A style for designing network APIs around resources and HTTP semantics.
- **Status code:** An HTTP response number that summarizes the result.
- **Idempotency:** Property where repeating an operation has the same intended effect.
- **OpenAPI:** A machine-readable description of HTTP API routes, schemas, and responses.
- **SSE:** Server-Sent Events, a one-way server-to-client event stream over HTTP.
- **WebSocket:** A persistent two-way communication channel between client and server.

## Python application design

- **ASGI:** An async-capable Python interface between web servers and applications.
- **Dependency injection:** A way to supply resources or policies to a function without hardcoding construction.
- **Event loop:** Scheduler that runs asynchronous tasks while they wait for I/O.
- **Coroutine:** An async function that can suspend and resume.
- **Blocking I/O:** Waiting operation that prevents the current execution thread from doing other work.
- **Timeout:** Maximum time allowed for an operation.
- **Connection pool:** Reusable set of database or network connections.
- **Worker:** A process that consumes background jobs.
- **Backpressure:** Mechanism that slows or rejects producers when consumers cannot keep up.

## Data

- **Primary key:** Column or set of columns that uniquely identifies a database row.
- **Foreign key:** Constraint linking a row to a valid row in another table.
- **Transaction:** A group of database operations treated as a unit.
- **ACID:** Atomicity, consistency, isolation, durability properties of transactions.
- **Migration:** Versioned database schema change.
- **Index:** Data structure that speeds selected queries at storage and write cost.
- **Cache:** Temporary or derived storage used to avoid repeated work.
- **TTL:** Time-to-live expiration duration.
- **Queue:** Durable or managed buffer of work for asynchronous processing.
- **Dead-letter queue:** Place for jobs that exhausted retries or cannot be processed.
- **Object storage:** Storage for large files addressed by keys rather than relational columns.
- **Vector index:** Structure that retrieves vectors by similarity.
- **Embedding:** Numeric representation of text or another item used by models.
- **Tenant:** Customer, organization, or isolation boundary in a multi-tenant application.

## Identity and security

- **Authentication:** Verifying who a caller is.
- **Authorization:** Deciding what an authenticated caller may do.
- **OAuth 2.0:** Framework for delegated authorization.
- **OpenID Connect:** Identity layer built on OAuth 2.0.
- **JWT:** Signed token format; claims must be validated and are not encrypted by default.
- **Scope:** Named permission or range of access.
- **Least privilege:** Give each identity only the minimum access it needs.
- **BOLA/IDOR:** Broken object-level authorization, where a caller accesses another user's object by changing an identifier.
- **SSRF:** Server-side request forgery, where a backend is tricked into making an unintended network request.
- **Rate limit:** Bound on how many requests may be made over a period.
- **Prompt injection:** Malicious or conflicting text that attempts to manipulate model behavior.
- **Secret:** Credential such as a password, token, or API key that must be protected.
- **Threat model:** Structured description of assets, actors, trust boundaries, attacks, and controls.

## AI application systems

- **Inference:** Using a trained model to produce a prediction or output.
- **Provider adapter:** Application layer that translates internal model requests to a provider-specific API.
- **Token:** Text unit used by a language model; token count is not word count.
- **Structured output:** Model response expected to match a schema such as JSON.
- **RAG:** Retrieval-augmented generation; retrieve evidence and pass it as context for generation.
- **Chunk:** Passage indexed or retrieved independently from a larger document.
- **Reranker:** Model that reorders candidate passages for a query.
- **Citation grounding:** Linking generated claims to source passages that support them.
- **Agent:** Model-driven loop that selects tools or actions based on observations.
- **Tool call:** Structured request from a model to an application-defined function.
- **Evaluation set:** Examples and criteria used to assess model behavior.
- **Hallucination:** Plausible generated claim that is unsupported or false.
- **Model drift:** Change in the relationship between inputs, outputs, and outcomes over time.

## Operations

- **Container image:** Packaged application and runtime dependencies.
- **Container:** Running instance of an image.
- **CI/CD:** Automated build, check, and release pipeline.
- **Health check:** Endpoint or check for process liveness or readiness.
- **Log:** Timestamped record of an event.
- **Metric:** Numeric measurement over time.
- **Trace:** End-to-end record of a request across components.
- **SLI:** Service-level indicator, a measured reliability property.
- **SLO:** Service-level objective, a target for an SLI over a period.
- **p95 latency:** Latency at or below which 95% of measured requests finish.
