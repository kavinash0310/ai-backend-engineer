# System design and cloud foundations for AI backends

> **Place in the course:** Advanced study after API, database, queue, and deployment foundations. Design for measured requirements; do not add infrastructure merely because a diagram looks more professional.

## Start from requirements

Ask who uses the system, what requests it handles, how much data it stores, what latency and availability users need, what data is sensitive, and what failures are acceptable. Estimate request rate, average and peak concurrency, prompt size, expected model tokens, file volume, and retention. These estimates expose cost and capacity constraints before implementation.

A first architecture may be one API service, PostgreSQL, object storage, a queue, a worker, and an external model provider. Keep boundaries clear but deploy fewer services until independent scaling or ownership justifies splitting them. A microservice adds network failures, deployment coordination, observability requirements, and data consistency problems.

## Stateless API and stateful resources

An API replica should ideally be stateless between requests: durable state lives in a database or object store, not one process's memory. Then a load balancer can route requests to any healthy replica. A user's session may be represented by a signed/opaque token or stored server-side; a websocket or stream is a temporary connection, not durable application state.

A database is stateful and needs backups, migrations, access control, and restore testing. Object storage holds large files; a relational row stores key, owner, type, checksum, and lifecycle. Queues smooth bursts and separate request acceptance from work. Each resource has its own scaling and failure mode.

## Capacity and bottleneck reasoning

If each API worker allows 20 concurrent model requests and there are 5 workers, the service could create 100 concurrent downstream calls. If the provider limit is 30, the backend needs a semaphore, queue, or lower worker concurrency. More API replicas can worsen overload.

Database pools multiply by replica count. Ten replicas with a pool size of 20 can create 200 connections. Compare that with the database connection limit and reserve capacity for migrations and admin tasks. Queue-based processing gives control over worker concurrency and retry rate.

The slowest component sets throughput. Measure end-to-end latency and each dependency. Model generation may dominate time, while database queries or document parsing may dominate cost. Use load tests with realistic prompt lengths and a traffic pattern that includes bursts.

## Cloud building blocks

A cloud deployment typically has compute (containers or serverless functions), private networking, managed database, object storage, identity/roles, secret management, logs/metrics, and a load balancer. A VPC or private network reduces exposure; IAM roles constrain what services can access. Store secrets in a managed system and rotate them. Use provider-specific services only after understanding the general role and exit/migration costs.

Serverless can simplify low-volume, bursty APIs but may have cold starts, execution limits, and streaming constraints. Long-running generation and background jobs often fit containers and queues better. Managed PostgreSQL reduces operational burden but does not eliminate backup, migration, connection, or access-policy responsibility.

## Availability and recovery

Define what happens when each dependency is unavailable. If the provider fails, can the API return a retryable error or queue the request? If the database is down, should the system reject writes clearly rather than accept unpersisted work? A cache may serve a labeled stale result only when product policy permits it.

Backups are useful only if restoration is tested. Define recovery point objective (how much data loss is acceptable) and recovery time objective (how quickly service should recover). Practice restoring a backup in an isolated environment. Write an incident runbook with owner, alerts, pause switch, rollback, and communication plan.

Questions: Why not split every domain into a microservice? Coordination and failure overhead can exceed the benefit. Why do pools need global planning? Each replica has its own pool. Why is an autoscaler not enough? It can amplify requests to a saturated database or provider.

Practice: draw an architecture for 1,000 daily users and for 100 requests per second. Mark API replicas, database pool limit, queue, worker concurrency, provider quota, storage, and observability. State what changes and what stays a single component at each scale.

Further reading: [Docker containers overview](https://docs.docker.com/get-started/docker-overview) and [OpenTelemetry signals](https://opentelemetry.io/docs/concepts/signals/).
