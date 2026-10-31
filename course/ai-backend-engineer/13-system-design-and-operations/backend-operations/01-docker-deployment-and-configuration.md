# Docker, configuration, and deployment

## Image and container

A container image packages an application and runtime dependencies. A container is a running instance of that image with a process, filesystem view, network, and resource limits. Containers make local and deployment environments more repeatable; they do not automatically make a service secure or highly available.

A Dockerfile describes how to build the image. Keep the build reproducible, use a small trusted base, copy only required files, install pinned dependencies, and run as a non-root user when practical. Never bake API credentials into image layers: layers can persist after a file is deleted. Supply secrets at runtime through a protected mechanism.

## Configuration and health

Separate code from environment-specific configuration. Validate required settings on startup and fail clearly if a secret or database URL is absent. Use distinct settings for local, test, staging, and production. Avoid printing secrets during startup.

A liveness check asks whether the process should be restarted. A readiness check asks whether it can accept traffic, perhaps because the database connection or model dependency is unavailable. Do not make health endpoints expose internal secrets or unlimited dependency details. Shutdown should stop new work and allow in-flight requests to finish within a limit.

## Deployment lifecycle

A CI pipeline can run formatting, static checks, unit tests, and build the image. A release should identify the image and configuration version. Use a staging environment with safe test data before production. A rolling or canary deploy reduces risk by limiting initial exposure. Keep a rollback path and plan database migrations so old and new app versions can coexist during a rollout.

Container orchestration helps run multiple services and recover processes, but adds concepts: desired state, service discovery, resource requests, rollout strategy, and secrets. Learn one-container deployment first; adopt orchestration when the operational need justifies it.

## Worked local stack

A learning application may run three containers: API, PostgreSQL, and a background worker. Compose defines services, a shared network, health checks, environment configuration, and a persistent database volume. The database data should survive container replacement; the container itself can be disposable.

Questions: Why are containers not virtual machines? Containers share the host kernel while isolating processes and filesystems; they package a different operational model. Why not put a secret in a Dockerfile? It can remain visible in image history. Why use readiness? A running process may not yet be able to serve requests.

Practice: package a FastAPI app, run it as a non-root user, pass configuration at runtime, connect it to a database container, and verify health and shutdown behavior.

Further reading: [Docker getting started](https://docs.docker.com/get-started/introduction/index.html) and [Docker containers learning path](https://docs.docker.com/learn/paths/get-started-with-docker-containers/).

## Release safely and keep configuration outside the image

A multi-stage Docker build can keep compilers and development tools out of the final runtime image. Pin dependencies and base versions, run as a non-root user, scan packages, and set CPU and memory limits. Never bake credentials into image layers because deleted files can remain in image history.

Inject environment-specific settings at runtime and validate them before readiness. Separate liveness (should this process restart?) from readiness (can it serve now?). Stop taking new traffic during shutdown, finish or cancel work, then close pools.

Database changes should support overlapping old/new application replicas during a rolling deploy. Add fields before requiring them, backfill in batches, switch code, then remove obsolete fields later. Run a migration as a controlled release step, not as a race among every replica.

Practice: deploy an API change that adds a database column. Document backward compatibility, migration order, health verification, rollback, and secret rotation.
