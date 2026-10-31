# 9. Cloud, Kubernetes, and Spring services

Deployment is part of system design. A container that works on a laptop is not production-ready until secrets, networks, scaling, persistence, health, upgrades, and recovery are deliberate.

## 9.1 Learn one cloud path

Choose the cloud used at work or by target employers. Learn its network/VPC, private subnets, DNS/TLS, ingress/firewall; managed PostgreSQL/backups/connection limits; secret manager/workload identity; object storage; queue/event bus; container registry/CI/CD; and logs, metrics, traces, alerts, cost visibility.

Cloud abstractions are similar, but permissions and operations differ. Practice one deeply, then compare others.

## 9.2 Container and configuration

Build a reproducible image with a non-root user, pinned base, health behavior, and no embedded credentials. Multi-stage builds can keep build tools out of runtime. Set CPU/memory requests and limits based on measurement, including Java heap headroom inside the container limit.

Inject configuration and secrets from managed storage. Never bake secrets into image layers or print them at startup. Make schema migrations explicit. Use backward-compatible expand-migrate-contract changes while old and new app versions overlap.

## 9.3 Kubernetes concepts

Learn Pods, Deployments, Services, ingress/gateway, ConfigMaps, Secrets, probes, resource requests/limits, autoscaling, service accounts, and network policy. Understand termination: in-flight requests, queue visibility timeout, lease expiry, graceful shutdown, and duplicate processing.

- **Liveness** detects a stuck process, not a provider outage.
- **Readiness** decides whether the instance accepts traffic.
- **Startup probe** protects slow initialization.
- **Graceful shutdown** stops new work, safely finishes or abandons work, releases leases.
- **Autoscaling** uses relevant demand such as queue age/concurrency, while respecting DB/provider capacity.

Autoscaling cannot create provider quota or database connections. Set global or tenant-aware concurrency budgets across replicas.

## 9.4 Rollout, data, recovery

Use health gates and gradual rollout where justified. Make prompt/model/app versions visible in telemetry. Canary/shadow evaluation requires privacy controls and cost limits. Roll back code safely; database changes need expand-migrate-contract because schemas may be shared by old instances.

Document backups and test restores. A backup never restored is an assumption. Define recovery time and point expectations for job state, documents, vector data, and evaluation metadata. Know which derived data can be rebuilt and how long.

## 9.5 Cost and quotas

Estimate model input/output, embeddings, reranking, retries, storage, database, queue, network, and observability. Bound request size, tokens, concurrency, tenant usage, and batch jobs. Alert on spend anomalies and quota exhaustion.

## Practice

Containerize and deploy the capstone. Configure workload identity and secret injection. Kill a worker during a job, roll back an app version, and restore a database snapshot in a test environment. Document the result.

## Questions and answers

**Does Kubernetes make a service reliable?**  
It schedules/restarts containers, but idempotency, recovery, limits, observability, and rollout compatibility remain application responsibilities.

**Should API pods run all embedding jobs?**  
Separate interactive and batch scaling when needs differ. Start simple, but keep a clear worker boundary.

**Can replicas scale only on CPU?**  
Model calls often wait on I/O, so CPU may not match demand. Use concurrency/queue signals while respecting downstream limits.

**What if the provider is down but the app is healthy?**  
Return controlled degraded behavior, stop wasteful retries, preserve accepted durable jobs where appropriate, alert, and communicate status.

## Mastery checkpoint

Explain how secrets, probes, shutdown, job recovery, provider limits, and DB capacity behave during deployment and restart.