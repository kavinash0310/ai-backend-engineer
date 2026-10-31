# System Design

System design is a required skill, not an interview-only diagram exercise.

## Fundamentals

Requirements, APIs, capacity and bandwidth estimates, storage, latency budgets, availability/reliability, load balancing, gateways, caching, SQL/NoSQL, indexes, replication/sharding, consistency, ACID/CAP, transactions, saga/outbox, queues/Kafka, rate limits, breakers/bulkheads/backpressure, disaster recovery, observability and evolution.

## AI-specific capacity

Estimate prompt/output tokens, requests/second, concurrent inference, model load/memory, embedding throughput, retrieval latency, time-to-first-token, total response latency, provider quotas and cost/request. Design exercises: search, document processing, large-scale RAG, LLM gateway, research workflow, enterprise knowledge system, and agent platform.

Use [AI Backend Engineer](https://github.com/kavinash0310/ai-backend-engineer/tree/main/course/ai-backend-engineer/02-programming/java-spring) and the [system design primer](https://github.com/kavinash0310/system-design-primer) as supporting references; write your assumptions and failure model.


## Detailed AI system-design lesson

- [AI system design: capacity, latency and reliability](./01-ai-system-design-capacity-and-reliability.md) — requirements, estimates, latency budgets, overload, consistency and SLOs.
