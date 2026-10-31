# Caching, queues, and vector storage

## Choose storage by its job

A relational database is useful for durable structured records and transactions. Object storage is suited to large files such as PDFs, images, and audio. A cache stores values that can be recomputed or fetched again. A queue holds work to be processed later. A vector index helps retrieve items by embedding similarity. These components complement each other; a vector database is not a replacement for a user's permission records or transaction database.

## Cache correctness

A cache maps a key to a value for a period or until invalidated. Cache-aside means the application checks cache, fetches the source on a miss, then stores the result. Choose keys that include tenant/user context where output depends on permissions. Caching only by prompt text may expose one user's private answer to another.

Decide time-to-live, invalidation, and behavior during cache outage. A cache should not become the only copy of important state unless it is designed for durability. Stale answers need a policy, especially when source documents change.

## Queues and background work

A queue decouples request acceptance from expensive work. The API validates and stores a job, then returns a job ID. A worker consumes the message, performs the operation, and updates status. Queues enable retry and smoothing bursts, but messages may be delivered more than once. Consumers should be idempotent: processing the same job twice should not charge twice or create duplicate results.

Retries should be bounded and use backoff. Separate transient failures, such as temporary network errors, from permanent failures, such as invalid input. A dead-letter queue or failed-job store makes repeated failures visible. A scheduled cleanup policy prevents abandoned jobs from accumulating forever.

## Object storage and vector search

Store original uploaded files outside the relational row when they are large. Save metadata and a storage key in the database. Validate file size and type, scan or safely process content, and avoid trusting a filename or declared MIME type.

For semantic retrieval, the ingestion pipeline extracts text, splits it into chunks, computes embeddings, and stores vectors with document, tenant, version, and access metadata. Query time embeds the question, retrieves candidates, filters by access, optionally reranks, and returns passages to a generator. The vector match is a relevance guess, not an authorization decision. Always enforce permissions from a trusted data source.

Questions: Can the vector store decide who can read a chunk? No; access control belongs to a trusted application/data layer. Why must queue workers be idempotent? Redelivery is normal after crashes or network uncertainty. When is a cache unsafe? When invalidation, tenant scoping, or freshness is not correctly handled.

Practice: design a queued PDF-ingestion job with job states, retry limit, duplicate handling, file size checks, tenant-specific chunk metadata, and cleanup behavior.

Further reading: [Redis documentation](https://redis.io/docs/latest/) and [PostgreSQL documentation](https://www.postgresql.org/docs/current/).

## Retries, duplicate delivery, and the outbox

Queues commonly provide at-least-once delivery: a worker can receive a message again after a crash or acknowledgement failure. Make consumers idempotent with a unique job ID and conditional state transitions. A retry should not send a message twice or charge twice.

There is a gap between committing a job row and publishing its queue message: the database can commit while the broker is unavailable. A transactional outbox writes the job and an outbox event in the same database transaction. A publisher later sends the event and marks it delivered. Duplicate sends can still happen, so the worker remains idempotent.

Cache keys must include tenant, user, document version, and model/prompt version when those affect the result. Otherwise matching prompts can leak one user's cached answer to another. Expire or invalidate results when permissions or source documents change.

Embedding models and chunking rules may change dimensions or meaning. Version the embedding model, chunker, and source; build a new index, evaluate it, then switch traffic. Vector similarity is relevance, never authorization. Verify access from trusted permission data before context reaches a model.

Practice: draw a crash-recovery sequence for ingestion, then describe how an outbox and idempotent worker prevent a lost job or duplicate side effect.
