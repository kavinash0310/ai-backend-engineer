# Cloud and deployment foundations for AI backend services

## 1. Deployment is more than putting code in a container

Production deployment includes artifact creation, identity, configuration, network access, secret handling, data stores, health checks, telemetry, scaling, rollback and incident ownership. A container can start successfully while the application is insecure, unreachable or unable to serve its latency target.

Choose one cloud for hands-on study. Learn portable concepts first: compute, private networking, IAM, object storage, managed SQL, queues, secret management, monitoring, and deployment automation. Avoid learning three vendor consoles at once.

## 2. Containers

A container image packages application code and runtime dependencies. A container is an isolated process using the host kernel; it is not a full virtual machine.

For a Java service, create a reproducible build artifact and runtime image. For Python, install from a pinned/locked dependency set and avoid copying development environments or secrets. Use multi-stage builds when useful, non-root users, a small supported base image, explicit ports and graceful shutdown.

Container hygiene:
- build from reviewed base images;
- pin base versions/digests under a patch policy;
- scan dependencies and image;
- avoid credentials in layers or build logs;
- set CPU/memory limits and understand JVM/container sizing;
- send logs to standard output/error for collection;
- define liveness/readiness correctly;
- run as non-root where practical;
- rebuild when dependencies or base images need security fixes.

A local Compose setup can run API, database, object store emulator and queue for development. It is not automatically production architecture; production managed services have different identity, networking, backup and reliability assumptions.

## 3. Identity and network

Cloud IAM defines workload identity and allowed API actions. Give each service a narrowly scoped identity; avoid long-lived static access keys in environment files.

A virtual private cloud/network isolates address ranges and routing. Subnets, route tables, firewalls/security groups, NAT, private endpoints and DNS determine which services can communicate. Keep database, vector store and model workers private where possible. Public ingress should pass through an authenticated edge/gateway.

TLS protects traffic in transit; verify certificates and avoid disabling hostname checks. Service-to-service authentication should identify workloads, not only trust a network location.

## 4. Storage and queues

- **Object storage**: raw documents and durable blobs; define encryption, versioning, retention and deletion.
- **Managed relational database**: metadata, tenants, jobs, transactional state and audit references; plan backups, schema migration, connection limits and restore tests.
- **Queue**: decouples ingestion/batch jobs; monitor lag, retries, visibility timeout and dead-letter messages.
- **Vector/search index**: retrieval capability; plan backups/rebuild, ACL propagation, index version and capacity.
- **Cache**: reduce repeated work only when keys include tenant, data/model version and policy context; define invalidation and retention.

A vector index can often be rebuilt from approved source data, but that does not remove the need for recovery time, access controls or deletion guarantees.

## 5. Deploying hosted-model integration versus self-hosting

Hosted inference reduces hardware operations but needs provider review, egress protection, request budgets, quota management, region/retention controls and failure handling.

Self-hosting adds model weight distribution, accelerator drivers/runtime, batching, KV cache, autoscaling, capacity planning, rollout and security patching. GPU capacity can be expensive and specialized. Start with the operational shape actually needed; do not assume a 16 GB development machine can serve a production LLM.

Keep model/provider credentials in a secret manager. Rotate them and grant only the required endpoint/operation. Avoid logging request headers that contain secrets.

## 6. CI/CD and releases

A pipeline typically:
1. checks source format/type/lint and appropriate test suites;
2. builds a versioned image/artifact;
3. scans dependencies and container;
4. signs or records provenance;
5. deploys to a staging environment;
6. runs smoke/contract/security/evaluation checks;
7. promotes gradually;
8. watches SLO and quality signals;
9. rolls back or disables a bad version.

For AI systems, deployment version includes more than application code: model identifier, prompt/template, tokenizer, embedding model, index schema, preprocessing, policy and evaluation set. Record these together.

Use database migrations with a rollback/compatibility plan. Never publish a new index/model/prompt version directly to all traffic without evaluating task quality and failure behavior.

## 7. Kubernetes concepts

Kubernetes orchestrates containers. Core concepts include Pod (one or more co-located containers), Deployment (desired replica state), Service (stable network abstraction), ConfigMap/Secret (configuration/secret reference), Job/CronJob (bounded/scheduled work), probes, resource requests/limits and autoscaling.

Kubernetes adds a control plane and operational complexity. Learn basic container and cloud runtime behavior first. For AI workloads, a CPU web API, GPU model server and asynchronous ingestion worker may need distinct scaling policies. Autoscaling on CPU alone may not reflect queue delay or tokens/second.

## 8. Observability and runbooks

Use metrics for rates, errors, duration and saturation; logs for event context; traces for cross-service request paths. OpenTelemetry provides standard instrumentation concepts; export to an approved backend.

AI-specific panels may include:
- request/queue/token volume by model and tenant-safe category;
- p95 TTFT and completion time;
- upstream throttles/timeouts and retries;
- retrieval/index freshness and ingestion lag;
- worker CPU/RAM/GPU memory/utilization;
- cost per task;
- sampled task-quality and citation outcomes.

A runbook should say how to detect, diagnose, mitigate and recover: provider outage, full queue, stale index, credential rotation, bad model rollout, resource exhaustion and accidental sensitive logging.

## 9. Backups, recovery and deletion

Backups are useful only if restore is tested. Define recovery point/time objectives and owners. Decide which derived artifacts can be rebuilt and from what source.

Deletion may need to reach primary storage, object versions, search/vector indexes, caches, backups according to policy, and external provider retention controls. The product should communicate realistic deletion timing and maintain an auditable process without retaining deleted content unnecessarily.

## 10. Cost controls

Cloud cost includes idle replicas, data transfer, object/database/index storage, logging, provider inference, accelerators, backups and engineer operations. Apply budgets and alerts by workload/tenant. Use autoscaling carefully: rapid scale-up can encounter quotas and cold starts; scale-down can interrupt model warm caches.

For a GPU service, measure model load time, memory, throughput, cache, concurrency and cost at representative sequence lengths. For a hosted API, measure input/output tokens, retry amplification and per-successful-task cost.

## Questions and answers

**Q: Is a container the same as a virtual machine?**  
A: No. A container packages and isolates processes while sharing the host kernel; a VM includes a separate guest operating system layer.

**Q: Should I start with Kubernetes?**  
A: Only if the learning goal/job or workload requires it. First understand processes, containers, networking, configuration, health and deployment; Kubernetes adds orchestration complexity.

**Q: Can I put a cloud secret in a container environment variable?**  
A: Avoid baking it into an image or source. Prefer workload identity/secret manager and control process/log access.

**Q: How do I prove backups work?**  
A: Perform a restore exercise, measure recovery time and validate application-level consistency. A successful backup job alone is not proof.

## Practice

1. Diagram production network boundaries for a Spring API, Python inference worker, SQL database, object store and hosted model provider.
2. Write a container hardening checklist for a Python worker.
3. Define staged rollout signals for a new model and prompt version.
4. Create a runbook outline for model-provider outage and queue saturation.
5. Plan restore and deletion checks for document storage plus vector index.

## Completion check

You can explain containers, cloud identity/network/storage/queues, deployment pipelines, Kubernetes basics, observability, rollback, backups and AI-specific cost/data controls.