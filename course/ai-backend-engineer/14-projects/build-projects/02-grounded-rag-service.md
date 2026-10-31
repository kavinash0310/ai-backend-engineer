# Project 2: permission-aware grounded RAG service

## Goal

Build a Java/Spring question-answering API over a small permission-scoped document collection. Retrieve authorized evidence, cite it, and abstain when evidence does not support an answer. RAG reduces some failure modes; it does not guarantee truth.

## Suggested stack

Spring Boot, Spring AI, Spring Security resource-server support, PostgreSQL, Flyway or Liquibase, pgvector or an evaluated vector service, object storage for original files, a queue/worker for ingestion, JUnit/Testcontainers, and an evaluation dataset.

## Design the two workflows

### Ingestion

1. Authenticate uploader and verify document ownership/tenant.
2. Enforce file type/size limits and safe parsing policy.
3. Store the source in object storage; record source ID, tenant, version, parser version, status, and retention metadata in PostgreSQL.
4. Extract and normalize content while preserving page/section metadata.
5. Chunk by document structure; record chunking and embedding model versions.
6. Embed and index chunks with tenant and access metadata.
7. Expose job status and safe failure reason.
8. On replacement or deletion, remove or supersede every derived chunk under the documented retention policy.

### Query

1. Authenticate caller and derive tenant/actor from trusted claims.
2. Apply document ACL filters during vector/keyword retrieval, before constructing model context.
3. Retrieve a bounded candidate set; optionally rerank and deduplicate.
4. Build a prompt with stable source IDs/sections and limited context.
5. Generate response with output budget and timeout.
6. Validate citations point only to retrieved authorized passages.
7. Return answer, citations, and explicit abstention status when evidence is insufficient.

## Requirements and tests

- Ingestion runs asynchronously and is idempotent.
- Source files remain outside the relational database; metadata and job state are durable.
- Record source version, tenant/ACL, parser, chunking, embedding model, and index status.
- Include lexical search for exact names/IDs if useful; measure whether it improves retrieval.
- Test direct-ID and semantic cross-tenant access, revoked permission, stale source, malformed file, duplicate job, provider timeout, unanswerable query, and prompt injection inside a document.
- Do not log document contents or prompts by default.

## Evaluation report

Create a versioned set with expected relevant source IDs and answer criteria. Report retrieval recall@k/precision@k separately from answer correctness, groundedness, and citation support. Include unanswerable/conflicting/outdated cases. Record model, prompt, dataset, parser, embedding, and retrieval configuration versions. Human-review a sample; automated judge scores alone are insufficient.

## Questions for design review

How quickly does deletion affect the vector index? What happens when permissions are revoked while ingestion is running? What if a chunk is relevant but unauthorized? How do you detect embedding-version mismatch? What evidence shows a citation actually supports a claim?