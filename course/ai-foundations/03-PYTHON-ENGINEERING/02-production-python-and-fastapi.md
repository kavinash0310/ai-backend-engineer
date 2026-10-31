# Production Python, APIs and async boundaries

## Move from notebook to package

A notebook is useful for exploration, but production code needs modules with clear responsibilities, validated configuration, tests, logging, reproducible dependencies and a deployment boundary. Separate API schemas, domain/service logic, storage adapters, model clients and configuration. Keep core behavior callable without HTTP so jobs and tests can reuse it.

## Types and boundary validation

Type hints help reviewers, editors and static checkers; Python does not enforce them at runtime by default. Validate untrusted data at boundaries such as HTTP, files and queue messages. Keep these checks distinct:
- type validation: is this the right kind of value?
- domain validation: is this value allowed for this task?
- authorization: may this identity do it?
- output validation: does generated data fit an allowed schema and business rule?

Pydantic can validate request shape and constraints; it cannot decide whether the authenticated caller has access to a document.

## HTTP API with FastAPI

FastAPI maps requests to Python handlers; Pydantic models commonly express schemas. Keep handlers thin and delegate policy/orchestration to application services.

~~~python
from pydantic import BaseModel, Field
from fastapi import FastAPI

app = FastAPI()

class QuestionRequest(BaseModel):
    question: str = Field(min_length=1, max_length=8000)
    max_answer_tokens: int = Field(default=500, ge=1, le=1500)

class AnswerResponse(BaseModel):
    answer: str
    trace_id: str
    status: str

@app.post("/v1/answer", response_model=AnswerResponse)
async def answer(request: QuestionRequest) -> AnswerResponse:
    result = await application_service.answer(request)
    return AnswerResponse(**result)
~~~

This only illustrates the shape. Authentication, trusted tenant scope, quotas, timeout, cancellation, safe errors and lifecycle-managed clients still need implementation. Do not accept tenant identity from an arbitrary request field.

## Async I/O versus model computation

Async I/O helps one event loop manage many operations waiting on network or database. It does not make CPU-heavy Python or GPU inference nonblocking. If a long native call runs on the event-loop thread, other requests stall.

Options include an async provider client, bounded thread/process pools, or a dedicated model-serving runtime/service. Bound worker concurrency and queue size: an unbounded executor turns overload into memory pressure. Cancellation may not interrupt a native computation immediately, so track abandoned work and deadlines.

## Configuration, errors and observability

Validate required settings at startup. Separate model endpoint/name/token caps from code. Keep a safe local configuration distinct from production. Store credentials in an approved secret manager and never commit keys, private data or weights by accident.

Use structured logs with trace ID, service/version, stage, status and safe timing fields. Avoid raw prompts, retrieved text, credentials and personal data by default. Map internal exceptions to stable safe categories such as validation, unauthorized, quota, timeout, dependency unavailable, malformed output and unexpected failure. Do not return stack traces.

## Health and deployment

Separate liveness (process is running) from readiness (service can accept traffic). Initialize long-lived clients in application lifespan and close them on shutdown. Build reproducible containers with pinned/locked dependencies, non-root execution where feasible, vulnerability scanning and no embedded secrets.

## Testable boundaries

Inject model and retrieval adapters so domain logic can use controlled fakes. Test validation, authorization, malformed output and error mapping deterministically. Keep separate evaluation runs for real model behavior because generation can be nondeterministic.

## Questions and answers

**Does async make inference faster?** Not inherently. It helps concurrent waiting on I/O; computation still consumes CPU/GPU and needs a bounded serving runtime.

**Is Pydantic authorization?** No. It validates data shape; trusted identity and policy decide access.

**Should the API return provider errors directly?** No. Map them to a stable application error and keep diagnostics in restricted telemetry.

## Practice

1. Convert notebook logic into a package service with explicit request and result types.
2. Add byte/token limits and safe error categories to a model endpoint.
3. Simulate a blocking call and compare event-loop responsiveness before/after bounded offloading.
4. Define local versus production endpoint configuration and secret handling.
5. Design health, readiness, startup and shutdown behavior for a model worker.

## Completion check

You can package Python logic, validate untrusted input, explain async limitations, bound inference work, and operate an API without exposing raw user content or secrets.