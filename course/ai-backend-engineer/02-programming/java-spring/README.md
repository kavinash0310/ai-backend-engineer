# Java + Spring AI backend course

This is the primary implementation track for someone with a Java/Spring Boot background. It teaches production AI service engineering without requiring a switch to a Python backend stack.

## System we are learning to build

~~~mermaid
flowchart LR
  U[Client] --> API[Spring MVC API]
  API --> APP[Application use case]
  APP --> SEC[Identity and policy]
  APP --> DB[(PostgreSQL)]
  APP --> Q[Durable job queue]
  Q --> W[Worker]
  W --> RET[Retrieval and tools]
  RET --> V[(Vector index with metadata)]
  W --> AI[Spring AI/provider adapter]
  AI --> W
  W --> DB
  API --> OBS[Metrics, traces, safe logs]
  W --> OBS
  AI --> OBS
~~~

A model call is an unreliable network dependency returning probabilistic content. Keep it behind explicit budgets, permissions, validation, and evaluation. A model is not your database, authorization layer, business-rule engine, or source of truth.

## Chapter order

| # | Chapter | Build or decide |
| --- | --- | --- |
| 1 | [Modern Java for AI backends](./01-modern-java-for-ai-backends.md) | Typed provider port, immutable messages, error taxonomy, deadline and concurrency |
| 2 | [Spring Boot service architecture](./02-spring-boot-service-architecture.md) | Layered request flow, validation, API contract, configuration and error mapping |
| 3 | [Spring Security for AI APIs](./03-spring-security-for-ai-apis.md) | OAuth2/JWT, tenant/object access, privacy and threat model |
| 4 | [Spring AI model integration](./04-spring-ai-model-integration.md) | Provider adapter, prompt/output controls, structured output, streaming and budgets |
| 5 | [Spring AI and RAG backends](./05-spring-ai-rag-backends.md) | Ingestion, chunking, embeddings, filtered retrieval, citations, evaluation |
| 6 | [Jobs, virtual threads, and messaging](./06-java-jobs-virtual-threads-and-messaging.md) | Durable work, idempotency, outbox, retries and recovery |
| 7 | [Testing Spring AI applications](./07-testing-spring-ai-applications.md) | Deterministic tests, Testcontainers, contracts and AI evaluation |
| 8 | [Spring observability and JVM performance](./08-spring-observability-and-jvm-performance.md) | Traces, metrics, redaction, SLOs, pools, GC and load diagnosis |
| 9 | [Cloud, Kubernetes, and Spring services](./09-cloud-kubernetes-spring-services.md) | Containers, secrets, probes, autoscaling, network and rollouts |
| 10 | [Senior system design and job readiness](./10-senior-system-design-and-job-readiness.md) | Capacity, capstone, design review, portfolio and interview practice |

## Prerequisites and versions

The notes assume you can read Java, use dependency injection, build REST endpoints, and work with SQL. Use a supported Java and Spring Boot line. Do not select Spring AI and Spring Boot versions independently: check the current compatibility matrix and import the matching BOM. Verify the provider API, quotas, model limits, and migration notes. Examples teach design patterns; check exact method signatures in current docs.

## Theory links to AI From Scratch

- [Textbook index](https://github.com/kavinash0310/ai-from-scratch/blob/main/course/TEXTBOOK.md) — vocabulary and learning order.
- [Mathematics](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/mathematics) and [ML](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/ml) — probability, generalization, leakage, calibration, evaluation.
- [NLP](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/nlp) and [LLMs](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/llm) — tokens, embeddings, attention, inference, context, fine-tuning.
- [Generative AI](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/generative-ai) — prompting, retrieval, tools, evaluation, safety.
- [AI engineering](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/engineering) — deployment and lifecycle concepts.

Python is useful for notebooks and data scripts, but optional for the backend unless a model-serving component requires it.

## Prerequisite and interview references from your repositories

These are optional refreshers, not extra required curricula:

- [Java Backend Interview Mastery](https://github.com/kavinash0310/java-backend-interview-mastery): use its [Core Java](https://github.com/kavinash0310/java-backend-interview-mastery/tree/main/core-java), [concurrency](https://github.com/kavinash0310/java-backend-interview-mastery/tree/main/multithreading-concurrency), [JVM](https://github.com/kavinash0310/java-backend-interview-mastery/tree/main/jvm-memory-management), and [Spring Boot](https://github.com/kavinash0310/java-backend-interview-mastery/tree/main/spring-boot) guides only where you need review.
- [CS Fundamentals](https://github.com/kavinash0310/cs-fundamentals): selected notes on [operating systems](https://github.com/kavinash0310/cs-fundamentals/blob/main/subjects/operating-systems.md), [networks](https://github.com/kavinash0310/cs-fundamentals/blob/main/subjects/computer-networks.md), [databases and SQL](https://github.com/kavinash0310/cs-fundamentals/blob/main/subjects/database-systems-sql.md), and [discrete math](https://github.com/kavinash0310/cs-fundamentals/blob/main/subjects/discrete-mathematics.md).
- For interview practice after Chapter 10, see [your System Design Primer fork](https://github.com/kavinash0310/system-design-primer) and its [upstream source](https://github.com/donnemartin/system-design-primer). Treat upstream content as third-party material and follow its license and attribution.

## Completion criteria

A chapter is complete when you can explain its terms, reproduce the exercise, test a failure case, and state a trade-off. The capstone should include a threat model, evaluation data, latency/cost report, traces, runbook, rollback instructions, and limitations.