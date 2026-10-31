# Project 1: model inference API

## Goal

Build a small HTTP service that accepts a user prompt and returns a model response through a server-side provider adapter. The goal is to demonstrate backend fundamentals, not to train a model.

## Requirements

- FastAPI endpoint with request and response schemas.
- Input validation and a maximum request size.
- Provider credentials loaded at runtime, never returned to clients or committed.
- Explicit connect and total timeouts.
- Bounded retry policy for transient failures.
- Stable handling of provider rate limits, invalid output, and outages.
- Request ID, token usage, latency, and estimated cost metadata.
- Unit tests using a fake provider and API tests for success and failure.
- Documentation describing what is and is not stored.

## Suggested endpoints

POST /v1/generations creates a request. GET /v1/generations/{id} can return the result if you persist it. If no persistence is needed initially, return the result directly but document that a client retry may call the provider again.

## Acceptance questions

What limits prevent expensive abuse? How do you ensure a client cannot choose a privileged model or system instruction? What does the client receive when the provider times out? Which fields are safe to log? Show a test for malformed structured output and another for rate limiting.
