# AI backend engineering projects

Build these projects in order using **Java and Spring Boot** as the default. Include a README, architecture/trust-boundary diagram, setup, API examples, tests, model/retrieval evaluation where relevant, cost/latency observations, security notes, and known limitations. Use the [readiness rubric](../assessments/README.md).

1. [Model inference API](01-model-inference-api.md) — authenticated Spring API, validated input, provider port, timeout, structured response, usage/cost metadata.
2. [Grounded RAG service](02-grounded-rag-service.md) — durable ingestion, tenant-aware retrieval, citations, deletion behavior, versioned evaluation.
3. [Production capstone](03-production-ai-backend-capstone.md) — jobs, database, provider limits, cloud deployment, observability, security review, recovery.

Older cross-stack examples may use Python/FastAPI. Treat them as concepts or optional comparisons; implement the primary route in Java/Spring. Use synthetic or licensed data. Never commit API keys or private user information.