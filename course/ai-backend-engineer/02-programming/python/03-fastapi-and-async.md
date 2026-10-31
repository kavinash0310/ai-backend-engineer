# FastAPI and async Python

## From route to application

FastAPI is a Python framework for building HTTP APIs. A route binds a method and path to a Python function. Request models describe and validate incoming JSON; response models describe public output. Type annotations support validation and documentation. A route function should coordinate the request, not contain every database and business rule.

A service can be separated into layers:
- API layer: parse HTTP, validate, authenticate, and format response.
- application/service layer: execute a use case such as answering a question.
- data-access layer: query or update persistence.
- integration layer: call model providers or external APIs.

This separation allows a service function to be tested without starting an HTTP server and lets the same logic be used by a background worker.

## Dependency injection

A dependency is a value or resource a route needs, such as a database session, current user, or settings. FastAPI's dependency system provides reusable construction and cleanup. Dependencies are useful for authentication and request-scoped resources, but authorization still needs explicit checks for the specific resource being accessed.

A database session should have a well-defined request or job lifetime and be closed reliably. Do not store per-user request data in global mutable variables. Application lifespan hooks can initialize shared resources such as connection pools and close them during shutdown.

## Async and blocking work

An async route can await asynchronous network or database I/O. While it waits, the event loop may handle other requests. Calling synchronous blocking work inside an async route can stall the loop. A synchronous route may be appropriate for synchronous libraries; framework behavior and deployment worker model must be understood.

A model API call is network I/O and often benefits from async clients, connection reuse, and explicit timeout. Local model inference may consume CPU/GPU for a long time and is not made nonblocking merely by adding async to the function. Use a separate worker or inference server for expensive computation.

## Errors and lifecycle

Translate expected failures into stable HTTP errors: invalid input, missing resource, permission denial, provider timeout, quota exceeded. Log technical context with a request ID, but return a safe client message. An unhandled exception should not leak credentials or internal prompts.

Read configuration at startup and validate required settings before accepting traffic. Do not hardcode API keys. Separate local, test, staging, and production configuration. Use an explicit dependency lock so deployments can reproduce the environment.

## Small route example

~~~python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class AskRequest(BaseModel):
    question: str = Field(min_length=1, max_length=4000)

class AskResponse(BaseModel):
    answer: str
    request_id: str

@app.post("/v1/ask", response_model=AskResponse)
async def ask(body: AskRequest):
    # Real application logic belongs in a tested service function.
    return AskResponse(answer="Replace this with your service.", request_id="example")
~~~

The example validates size and publishes a response schema. It intentionally does not call an AI provider directly from the route; a real route would authenticate the user, verify access, call a service layer with timeouts, and handle failures.

Questions: What does dependency injection improve? It makes shared resources and policies reusable and testable. Does async make CPU work fast? No. Why define a response model? It makes output shape explicit and helps detect accidental data exposure.

Practice: build a notes API with create/read/list routes, request and response models, pagination, validation, a database dependency, and tests. Add a service function and test it without HTTP.

Further reading: [FastAPI tutorial](https://fastapi.tiangolo.com/tutorial/) and [FastAPI dependencies](https://fastapi.tiangolo.com/tutorial/dependencies/).

## Resource ownership and cancellation

Create shared HTTP clients and database pools once during application startup, reuse them, and close them on shutdown. Recreating clients on each request wastes connection setup and may exhaust sockets. A request-scoped database session should be cleaned up whether the route succeeds or raises.

The event loop switches among coroutines when they await. A blocking HTTP client or CPU-heavy inference inside an async handler can stall unrelated requests. Use async clients for network waits; move CPU/GPU-heavy work to a worker or dedicated inference service. Async is concurrency for waiting, not automatic parallel computation.

When a browser disconnects during generation, decide whether to cancel provider work, continue and persist the result, or mark a job cancelled. Bound concurrency so many simultaneous requests cannot exceed database connections or provider quotas. Keep the route thin: parse HTTP, call a service use case, and translate the result into an HTTP response.

Practice: create a service function callable from both a route and a worker. Test it without an HTTP server. Simulate timeout and cancellation and record the job's terminal state.
