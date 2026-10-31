# AI Backend Engineer: from foundations to production systems

A practical self-study path for learning how to build, secure, test, deploy, and operate backend services that use machine-learning models and large language models.

## Start with the two-repository path

This repository focuses on backend engineering and production AI systems. It is designed to work alongside [AI From Scratch](https://github.com/kavinash0310/ai-from-scratch), which teaches Python, mathematics, AI, machine learning, deep learning, NLP, LLMs, generative AI, and responsible AI from beginner level.

- If you are new to programming or AI, begin with the [AI From Scratch textbook](https://github.com/kavinash0310/ai-from-scratch/blob/main/course/TEXTBOOK.md), especially [Python](https://github.com/kavinash0310/ai-from-scratch/blob/main/course/python/TEXTBOOK.md), [ML](https://github.com/kavinash0310/ai-from-scratch/blob/main/course/ml/TEXTBOOK.md), [LLMs](https://github.com/kavinash0310/ai-from-scratch/blob/main/course/llm/TEXTBOOK.md), and [generative AI systems](https://github.com/kavinash0310/ai-from-scratch/blob/main/course/generative-ai/TEXTBOOK.md).
- Use this repository for HTTP APIs, databases, authentication, queues, model-provider integration, RAG backends, testing, deployment, reliability, observability, and security.
- You do not need to repeat all AI theory here. Learn enough model behavior to make sound backend design decisions, then link back to the AI course when a concept needs deeper study.

## The learning path

~~~mermaid
flowchart LR
  A[Python and Git] --> B[Linux, HTTP, networking]
  B --> C[API design and FastAPI]
  C --> D[SQL, PostgreSQL, data modeling]
  D --> E[Auth, security, testing]
  E --> F[Async work, queues, caching]
  F --> G[Model APIs and inference]
  G --> H[RAG and tool workflows]
  H --> I[Docker and deployment]
  I --> J[Observability, scale, reliability]
  J --> K[Production capstone]
~~~

A reasonable pace is 6–10 focused hours each week over roughly 6–10 months, depending on prior coding experience. Move forward when you can build and explain the checkpoint, not just when you finish reading.

## What you will be able to build

By the end, you should be able to design a backend that accepts a request, authenticates the caller, validates input, reads and writes persistent data, calls an AI model safely, handles slow work, returns a clear response, measures quality and cost, and recovers from failures. You will also learn when a queue, cache, vector index, or agent is unnecessary.

## Course map

1. **Programming and computer foundations** — reuse the Python and Git path from AI From Scratch; learn the shell, processes, environment variables, and basic networking.
2. **HTTP and API design** — requests, responses, JSON, status codes, validation, REST conventions, errors, pagination, streaming, and versioning.
3. **Python backend services** — FastAPI, request models, dependency injection, async I/O, configuration, and service boundaries.
4. **Data systems** — PostgreSQL, SQL, transactions, indexes, migrations, connection pools, Redis, object storage, and queues.
5. **Identity and security** — authentication versus authorization, sessions and tokens, object-level permissions, input limits, secret handling, and AI-specific threats.
6. **AI integration** — model API contracts, timeouts, retries, rate limits, token and cost budgets, structured outputs, embeddings, RAG, and tool calling.
7. **Testing and evaluation** — unit, integration, contract, load, failure, and AI quality tests.
8. **Deployment and operations** — containers, configuration, health checks, logs, metrics, traces, SLOs, scaling, and incident response.
9. **Portfolio capstones** — ship increasingly complete AI backend services with documentation and evidence.

## How to study a chapter

Read the concept, trace the request or data flow, reproduce the example, answer the questions without looking, and complete the practice task. Record decisions and failure cases. A production backend is not just an endpoint that returns a model answer; it is the surrounding data, identity, safety, performance, and operations system.

## Career expectations

This repo can build a strong learning path and portfolio foundation. It cannot guarantee a job or expert status. Employers expect you to explain trade-offs, read documentation, debug real services, review security, and maintain systems whose inputs and dependencies change. Use the project rubrics and self-assessments as evidence of skills, not as a credential.

## Contents

- [Full course roadmap](ROADMAP.md)
- [Course index](course/README.md)
- [Progress tracker](PROGRESS.md)
- [Glossary](course/GLOSSARY.md)
- [Projects](projects/README.md)
- [Assessment and readiness rubric](assessments/README.md)
