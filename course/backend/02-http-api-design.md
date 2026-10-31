# HTTP and API design

## What an API contract does

An application programming interface (API) defines how one program asks another to do work. An HTTP API uses requests and responses. A contract defines paths, methods, input fields, output fields, authentication needs, errors, limits, and compatibility expectations.

A request usually includes a method such as GET or POST, a path such as /documents/42, headers, and optionally a body. The response includes a status code, headers, and often JSON. JSON is a text format for objects, arrays, strings, numbers, booleans, and null. It is not a database and does not define access permissions.

Common methods communicate intent: GET reads; POST creates or triggers work; PUT replaces a resource; PATCH changes selected fields; DELETE removes a resource. Method semantics matter for caching, retries, and client expectations. An idempotent operation has the same intended effect when repeated. GET should be safe to retry. A payment-like POST may need an idempotency key so a network retry does not create two actions.

## Status codes and errors

Use status codes to describe outcomes: 2xx for success, 4xx for client/request problems, 5xx for server failures. Return a stable machine-readable error shape, a safe message, and a correlation/request ID. Do not expose stack traces, SQL queries, prompts, or internal provider credentials to clients.

Validation should reject missing required fields, invalid types, values outside bounds, and excessive input size before expensive model calls. Return field-level details that help a legitimate client correct input without revealing internal implementation.

## Design resources and long-running work

A resource is a named thing such as a conversation, document, or job. Good routes make ownership and lifecycle visible. Pagination prevents returning an unbounded list. Filtering and sorting should have defined limits. Version an API when clients need time to migrate from a breaking contract.

An AI completion may take longer than an interactive request budget. One design is to create a job and return 202 Accepted with a job identifier; the client polls or subscribes for the result. For token-by-token output, Server-Sent Events or WebSockets may be appropriate. Streaming needs clear event types, a final event, cancellation behavior, and reconnect semantics.

## Worked contract

~~~http
POST /v1/answers
Content-Type: application/json
Authorization: Bearer <access-token>
Idempotency-Key: 81d7...

{"question":"How do I reset my password?","document_set_id":"help-center"}
~~~

The API should authenticate the caller, verify they may query that document set, validate question length, apply rate and token limits, run retrieval and generation, then return an answer with source identifiers. A 401 means identity is missing/invalid; a 403 means identity exists but the action is not allowed; a 429 means a rate or quota limit was reached. The client needs a retry policy that respects any retry-after instruction.

## Compatibility and documentation

An OpenAPI schema describes routes, request and response models, status codes, and authentication requirements. Generated documentation helps client developers but does not guarantee the implementation matches it; contract tests can check that. Additive optional fields are generally easier to introduce than changing meanings or removing fields. Do not expose database tables directly as public API schemas.

Questions: Why validate before calling a model? It saves money and prevents invalid or oversized inputs from reaching downstream systems. Why use idempotency? Networks fail after a server may have completed an operation but before a client receives the response.

Practice: design an API for uploading a document and checking its processing status. Define paths, methods, response schemas, error codes, pagination or size limits, ownership checks, and retry behavior.

Further reading: [HTTP Semantics, RFC 9110](https://www.rfc-editor.org/rfc/rfc9110) and [FastAPI tutorial](https://fastapi.tiangolo.com/tutorial/).
