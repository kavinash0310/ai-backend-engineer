# Roadmap: become an AI backend engineer

This path teaches how to build backend services around AI models. It starts at programming and web fundamentals, then adds data, security, model integration, and operations. The paired [AI From Scratch course](https://github.com/kavinash0310/ai-from-scratch) supplies deeper AI/ML theory; this repo focuses on the services that make models usable and reliable.

## Before you begin

You do not need prior AI experience. You do need regular practice. If Python is new, follow the [Python foundation path](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/python) alongside Stage 1. Learn SQL and Git as you reach them. A computer that can run Python, PostgreSQL, and Docker is enough for most learning projects; use hosted model APIs or small local models when hardware is limited.

## Path and milestones

| Stage | Focus | Evidence before moving on |
| --- | --- | --- |
| 0 | Programming, Git, terminal, and AI map | Explain a Python program, use Git, and link the model concepts you need |
| 1 | Linux process and network basics | Run a local server and inspect a request |
| 2 | HTTP and API design | Design and document a validated REST endpoint |
| 3 | FastAPI service architecture | Build a small service with error handling and dependency injection |
| 4 | SQL and PostgreSQL | Model data, write joins, use transactions and migrations |
| 5 | Authentication and API security | Enforce user and object permissions; pass a threat-model review |
| 6 | Async work, caches, and queues | Move slow jobs off the request path and explain retry/idempotency |
| 7 | Model-provider integration | Call a model with limits, timeout, retries, structured output, and cost tracking |
| 8 | RAG backend and tool workflows | Ingest documents, retrieve with permissions, cite evidence, and test failures |
| 9 | Testing and evaluation | Demonstrate unit, integration, contract, load, and AI quality checks |
| 10 | Docker and deployment | Build a reproducible image and deploy with health checks and secret configuration |
| 11 | Observability and reliability | Trace one request end-to-end and diagnose a simulated failure |
| 12 | Capstone and portfolio | Ship a documented service with a demo, evaluation report, and known limitations |

At 6–10 focused hours per week, a learner with Python basics might spend 6–10 months on the path. A complete beginner may need longer. The schedule is a planning aid, not a race.

## Suggested weekly rhythm

1. Read one lesson and write a small glossary in your own words.
2. Reproduce the example locally.
3. Change one assumption and observe the effect.
4. Add tests for success, invalid input, and failure.
5. Write a short note: what failed, why, and how you know the fix works.

Keep the first projects small. You will learn more from a well-tested API using SQLite or PostgreSQL and a hosted model than from adding Kubernetes, several databases, and a complex agent before understanding request flow.

## How this path connects to AI From Scratch

Use the other repo for Python, linear algebra, probability, ML evaluation, embeddings, transformer attention, LLM limitations, and responsible AI. Relevant starting points are its [textbook index](https://github.com/kavinash0310/ai-from-scratch/blob/main/course/TEXTBOOK.md), [LLM subject volume](https://github.com/kavinash0310/ai-from-scratch/blob/main/course/llm/TEXTBOOK.md), [RAG and tool lessons](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/generative-ai), and [AI engineering subject folder](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/engineering).

## Portfolio standards

For each project, publish a README that states the user problem, architecture, setup steps, API examples, data handling, model choice, evaluation results, cost and latency assumptions, security controls, failure behavior, and limitations. Never put private keys, personal data, or unauthorized datasets in a public repository.

A backend engineer is not measured by whether a model returns a fluent answer. Show that the system validates inputs, restricts access, handles timeouts, preserves data correctly, records useful telemetry without leaking sensitive content, and can recover from failure.
