# AI Backend Engineer — Java and Spring Boot path

A structured path for Java/Spring Boot developers who want to design, build, secure, evaluate, and operate production systems that use machine-learning models and large language models.

**Start here:** [AI foundations course](course/ai-foundations/README.md) · [Java + Spring AI course](course/java-spring/README.md) · [Roadmap](ROADMAP.md) · [Progress](PROGRESS.md)

This repo teaches the engineering around models. The [AI foundations course](course/ai-foundations/README.md) provides a connected theory-first path inside this repository, and [AI From Scratch](https://github.com/kavinash0310/ai-from-scratch) is a companion for broader AI/ML theory, Python, mathematics, NLP, transformers, LLMs, embeddings, and generative AI. You do not need to switch your backend language from Java to follow this path.

## The route

~~~mermaid
flowchart TD
  A[Java and Spring Boot you already know] --> B[AI and model literacy]
  B --> C[Modern Java for model I/O]
  C --> D[Spring service boundaries and APIs]
  D --> E[Identity, privacy, threat modeling]
  E --> F[Spring AI integration]
  F --> G[PostgreSQL, embeddings, permission-aware RAG]
  G --> H[Durable jobs and messaging]
  H --> I[Testing and AI evaluation]
  I --> J[Observability, JVM performance, cost]
  J --> K[Docker, cloud, Kubernetes]
  K --> L[Senior production capstone]
~~~

## Study order and evidence

| Phase | Learn | Evidence |
| --- | --- | --- |
| 0. AI literacy | Inference, tokens, context, embeddings, retrieval, uncertainty, evaluation | Explain what a model can and cannot guarantee |
| 1. Java readiness | Immutability, records, error modeling, timeouts, executors, virtual threads | Typed provider adapter with bounded waiting and tests |
| 2. Spring architecture | MVC, validation, use-case boundaries, configuration, REST contracts | Documented API with stable errors and clear layers |
| 3. Security and data | PostgreSQL, transactions, OAuth2/OIDC, JWT, tenant/object access | Threat model and tests proving tenant isolation |
| 4. Model integration | Spring AI, provider adapters, structured output, streaming, budgets | Inference endpoint with limits and failure policy |
| 5. RAG and workflows | Ingestion, embeddings, retrieval, citations, queues, idempotency | Evaluated RAG service and recoverable background jobs |
| 6. Quality | JUnit, integration/contract tests, Testcontainers, AI evaluation | Repeatable software tests and a versioned quality report |
| 7. Production | Docker, one cloud, Kubernetes basics, OpenTelemetry, SLOs, JVM tuning | Deployed service with traces, dashboard, runbook, and rollback |
| 8. Senior practice | Capacity, trade-offs, architecture decisions, incidents, communication | Capstone design review with measurements and limitations |

## Technology map

**Primary:** Java, Spring Boot, Spring MVC, Spring AI, Maven or Gradle, PostgreSQL, Flyway or Liquibase, JUnit 5, Testcontainers, Docker, Micrometer/OpenTelemetry, and one cloud.

**Add when justified:** Redis for bounded caching or coordination; Kafka or a managed queue for durable work; pgvector or a dedicated vector store for semantic retrieval; Kubernetes for orchestration; Python for notebooks, data preparation, training, or model-serving components.

Check the current Spring AI/Spring Boot compatibility matrix and BOM instead of copying old version numbers. Framework, Java, provider, and cloud support change. See [official references](SOURCES.md).

## Use your AI From Scratch repository

Follow the [textbook index](https://github.com/kavinash0310/ai-from-scratch/blob/main/course/TEXTBOOK.md), then use these theory subjects as needed:

1. [Mathematics](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/mathematics) and [ML](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/ml): probability, generalization, leakage, calibration, and evaluation.
2. [NLP](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/nlp) and [LLMs](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/llm): tokens, embeddings, attention, context, inference, and fine-tuning.
3. [Generative AI](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/generative-ai): prompting, RAG, tool use, evaluation, and safety.
4. Return here for APIs, security, persistence, queues, testing, deployment, and operations.

Most backend engineers integrate models; ML engineers may also train or serve them. Senior AI backend engineers need enough model literacy to challenge designs, evaluate behavior, and avoid treating probabilistic output as truth.

## Use your existing CS and Java repositories selectively

You already have supporting material, so do not re-study every foundation chapter if you can demonstrate the skill. Use these repos to fill specific gaps:

| Need a refresher in | Your repository |
| --- | --- |
| Core Java, Spring Boot, concurrency, JVM, messaging, AWS, Docker/Kubernetes, and interview practice | [Java Backend Interview Mastery](https://github.com/kavinash0310/java-backend-interview-mastery) |
| Operating systems, computer networks, SQL/databases, algorithms, discrete math, and OOP | [CS Fundamentals](https://github.com/kavinash0310/cs-fundamentals) |
| System design interview patterns after completing this course's design chapter | [Your System Design Primer fork](https://github.com/kavinash0310/system-design-primer), with the [upstream project](https://github.com/donnemartin/system-design-primer) as the canonical source |

Use these as references, not prerequisites to finish from cover to cover. For the fork, follow its upstream license and attribution; the AI backend lessons here provide the AI-specific design context.

## Senior-level outcome

You should be able to explain a request end to end; choose synchronous or asynchronous execution; set quality, latency, privacy, and cost budgets; protect tenant boundaries; measure retrieval separately from generation; manage provider outages and model changes; and present trade-offs using evidence.

This curriculum structures learning and portfolio practice. Reading alone does not certify expertise or guarantee a job; seniority also comes from owning production systems and learning from real constraints and incidents.

## Repository guide

- [Full roadmap](ROADMAP.md)
- [AI foundations course](course/ai-foundations/README.md)
- [Java/Spring course index](course/java-spring/README.md)
- [Progress tracker](PROGRESS.md)
- [Glossary](course/GLOSSARY.md)
- [Projects](projects/README.md)
- [Senior readiness assessment](assessments/README.md)
- [Official references](SOURCES.md)

The existing [backend](course/backend/README.md), [data](course/data/README.md), [security](course/security/README.md), [AI systems](course/ai-systems/README.md), [testing](course/testing/README.md), and [operations](course/operations/README.md) folders contain cross-stack references. FastAPI is optional on this Java-first route.