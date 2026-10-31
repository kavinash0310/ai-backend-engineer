# AI Application Engineering

This is where model concepts become reliable product behavior.

## Scope

Spring AI and model clients; Python/FastAPI/Pydantic/PyTorch/Hugging Face inference; provider and model routing; Java↔Python REST/gRPC/messaging; streaming and cancellation; queues and asynchronous workflows; retries, timeouts, circuit breakers and idempotency; caching and invalidation; quotas/cost; testing and evaluation; observability.

Design choices should compare a modular monolith and a service split. A separate Python service is useful when model/data dependencies, scaling, ownership or GPU runtime warrant isolation; it also adds network failure, deployment, auth, schema, tracing and consistency cost.


## Detailed lesson

- [Java and Python AI service architecture](01-java-python-ai-service-architecture.md) — deployment shapes, contracts, Spring responsibilities, inference worker boundaries and failure handling.
- Check current [Spring AI compatibility and BOM guidance](https://docs.spring.io/spring-ai/reference/getting-started.html) before selecting versions.

- [MLOps and model lifecycle](02-mlops-and-model-lifecycle.md) — reproducibility, registries, rollout, monitoring, drift and retraining policy.
