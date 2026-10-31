# 7. Testing Spring AI applications

AI systems need ordinary software tests and separate evaluation of uncertain model behavior. Passing a unit test cannot prove an answer is correct; a few good answers cannot prove the API is secure or reliable.

## 7.1 The test pyramid

~~~mermaid
flowchart TD
  U[Unit: policy, mapping, retries, limits] --> I[Integration: Spring, DB, HTTP adapters]
  I --> C[Contract: provider and public API assumptions]
  C --> E[AI evaluation: retrieval and answer quality]
  E --> L[Load, resilience, and security exercises]
~~~

Keep fast CI deterministic. Do not make every build call a paid model API. Live-provider checks can be controlled smoke tests with explicit spend, credentials, and data policies.

## 7.2 Unit tests

Use JUnit 5 and a fake application port/model adapter. Cover prompt limits and truncation; tenant/role authorization; provider outcome mapping; retry count and end-to-end deadline; output parsing and semantic validation; idempotency and state transitions; fallback or abstention.

Test behavior, not private method counts, unless the interaction itself is required. Inject a clock and ID generator where deterministic behavior helps.

## 7.3 Spring and integration tests

Use Spring Boot testing for wiring, HTTP behavior, validation, filters, serialization, errors, and configuration. Use Testcontainers or disposable dependencies when actual PostgreSQL or queue semantics matter. In-memory databases may differ in SQL, locking, extensions, collation, and transactions.

For external HTTP providers, use a local stub to simulate normal response, timeout, 429, 5xx, invalid JSON, connection reset, and slow streaming. Assert timeout and retry budgets. Test startup behavior with missing secrets so local and production policies are intentional.

## 7.4 Contract and provider tests

A provider adapter contract suite runs against fakes and, if approved, a sandbox provider. Verify request fields, authentication, timeout, response mapping, usage fields, and normalized errors. Rerun when changing provider SDK/API versions.

Version the public API schema. Validate model output against an explicit contract. Schema compatibility does not prove semantic quality.

## 7.5 AI evaluation dataset

Version cases containing input, expected behavior, relevant source IDs, authorization context, expected abstention/policy outcome, and risk/difficulty tags.

Measure separate stages:
1. **Retrieval:** recall@k, precision@k, ranking quality, permission leakage.
2. **Generation:** correctness, groundedness, citation support, completeness, unsafe behavior, refusal/abstention quality.
3. **System:** latency, token use, provider errors, cost, task success.

Use deterministic checks where possible: citation IDs exist, numbers parse, required fields validate, forbidden tools do not execute. Use transparent rubrics and human review for subjective quality. An LLM judge is another model output; record its version/prompt and inspect samples with people.

## 7.6 Regression and release gates

Record dataset, model, prompt, retrieval settings, parser/embedding, and evaluator versions. Compare candidates to a baseline. Inspect critical slices: unanswerable questions, protected tenants, long documents, and high-risk actions. Avoid one aggregate score hiding severe failures.

Set minimum gates for privacy/safety errors and acceptable quality/latency/cost ranges. A quality improvement that doubles cost is a product trade-off requiring measured discussion.

## Practice

Create CI with unit tests, PostgreSQL integration, stubbed provider behavior, and a small versioned evaluation set. Include an unanswerable question and cross-tenant attack. Publish baseline/candidate results.

## Questions and answers

**If the model is mocked, is AI tested?**  
The mock tests your handling. It does not test retrieval or real model quality. You need deterministic integration behavior and representative evaluation.

**Should unit tests call a live model?**  
Usually no. Live calls are slow, costly, flaky, and hard to reproduce. Use a separately controlled evaluation/smoke suite.

**Can an LLM judge prove correctness?**  
No. It may share biases and make mistakes. Use objective checks and human review for important examples.

**How do I test RAG tenant safety?**  
Seed separate tenant documents, query under both identities, test direct IDs and semantic retrieval, and assert no foreign text or citations escape.

## Mastery checkpoint

Show CI tests plus a versioned evaluation report. Explain what the code tests establish and what the model evaluation does not prove.