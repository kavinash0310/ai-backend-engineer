# AI backend engineering course

This course focuses on the backend around AI: APIs, data, identity, model calls, retrieval, tools, testing, deployment, and operations. Use [AI From Scratch](https://github.com/kavinash0310/ai-from-scratch) for deeper programming, mathematics, machine-learning, NLP, and LLM foundations. This repository links those lessons instead of duplicating them.

## Mental model

~~~mermaid
flowchart LR
  U[Client or user] --> A[API gateway and authentication]
  A --> V[Validation and application logic]
  V --> D[(PostgreSQL and object storage)]
  V --> Q[Queue for slow work]
  Q --> M[Model provider or inference service]
  M --> R[Retrieval and tools]
  R --> D
  V --> O[Response or stream]
  A --> T[Traces, metrics, and safe logs]
  Q --> T
  M --> T
~~~

A production AI backend is ordinary software plus model-specific concerns. It must handle identity, data correctness, timeouts, concurrency, cost, model uncertainty, and changing dependencies.

## Learning order

1. **Reuse foundations:** [Python from zero](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/python), [AI/ML textbook](https://github.com/kavinash0310/ai-from-scratch/blob/main/course/TEXTBOOK.md).
2. **Backend basics:** [Linux, processes, and networking](backend/01-computer-networking-and-linux.md), [HTTP and API design](backend/02-http-api-design.md), [FastAPI and async Python](backend/03-fastapi-and-async.md).
3. **Software quality:** [testing and architecture](testing/01-testing-ai-backends.md), [authentication and API security](security/01-identity-and-api-security.md).
4. **Data:** [PostgreSQL and transactions](data/01-postgresql-and-data-modeling.md), [caches, queues, and storage](data/02-caching-queues-and-vector-storage.md).
5. **AI services:** [model provider integration](ai-systems/01-model-provider-integration.md), [RAG backend](ai-systems/02-rag-backend.md), [tools and workflows](ai-systems/03-tools-workflows-and-streaming.md), [LLM application security](security/02-llm-application-security.md).
6. **Ship and operate:** [Docker and deployment](operations/01-docker-deployment-and-configuration.md), [observability and scaling](operations/02-observability-reliability-and-scaling.md).\n7. **Advanced electives:** [system design and cloud](operations/03-system-design-and-cloud.md), [self-hosted model serving](ai-systems/04-self-hosted-model-serving.md).

## Subject folders

| Folder | What it teaches |
| --- | --- |
| [Backend](backend/README.md) | Computer processes, networks, HTTP, APIs, FastAPI, async |
| [Data systems](data/README.md) | PostgreSQL, transactions, caching, queues, object and vector storage |
| [Security](security/README.md) | Identity, authorization, API risks, LLM-specific threats |
| [AI systems](ai-systems/README.md) | Model calls, RAG, tools, workflows, streaming, self-hosted serving |
| [Testing](testing/README.md) | Software tests, API contracts, AI evaluation, load and failure tests |
| [Operations](operations/README.md) | Containers, configuration, system design, cloud, deployment, observability, reliability |
| [Projects](../projects/README.md) | Three projects from model API to production capstone |

## What “done” means

Reading alone does not complete a stage. Build the small system, test successful and failing cases, describe the data and security boundary, measure behavior, and document one limitation. Track that evidence in [PROGRESS.md](../PROGRESS.md). Use the [readiness rubric](../assessments/README.md) to decide what to practice next.
