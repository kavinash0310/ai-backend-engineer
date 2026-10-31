# 5. Spring AI and retrieval-augmented generation backends

Retrieval-augmented generation (RAG) finds relevant material and provides it to a language model. The backend must implement the data pipeline, access policy, and evidence trail. Adding a vector database alone does not create a trustworthy RAG system.

## 5.1 End-to-end flow

~~~mermaid
flowchart TD
  A[Authorized upload or sync] --> B[Validate type, size, tenant, malware]
  B --> C[Extract text and source metadata]
  C --> D[Normalize and split into chunks]
  D --> E[Embed chunks]
  E --> F[(Store chunks, vectors, ACL, versions)]
  Q[User question] --> P[Authenticate and authorize]
  P --> QE[Embed query]
  QE --> R[Filtered vector or hybrid retrieval]
  F --> R
  R --> RR[Optional reranking and deduplication]
  RR --> CTX[Bounded cited context]
  CTX --> LLM[Generate grounded answer]
  LLM --> V[Validate citations and response]
  V --> OUT[Answer with sources or abstention]
~~~

Ingestion and query workflows have different latency, retry, authorization, and monitoring needs. Ingestion usually runs as a durable background job; query may be synchronous if bounded.

## 5.2 Ingestion as data engineering

Track source ID/version, tenant, status, content type, parser version, chunking strategy, embedding model/version, timestamps, and safe failure reason. This allows reprocessing, comparison, and deletion of derived data.

A robust sequence is:
1. Verify caller permission, type/size, ownership, and content policy.
2. Store the original safely or reference its authorized source.
3. Extract text with a suitable parser; extraction may fail or produce misleading text.
4. Normalize boilerplate while preserving headings, page, and section metadata.
5. Split into chunks and evaluate overlap. Tiny chunks lose context; huge chunks dilute relevance and consume token budget.
6. Embed using a documented model and dimension.
7. Persist chunks and ACL metadata; expose job status and retry policy.
8. Delete or supersede old chunks consistently when a source changes.

## 5.3 Storage and retrieval choices

Keep authoritative source/version data in relational tables. A vector index supports semantic similarity but does not replace primary keys, ownership, deletion, or audit history. PostgreSQL with a vector extension may suit moderate workloads; a dedicated vector database may fit scale, filtering, latency, or operational constraints. Benchmark on your corpus.

Useful retrieval signals:
- vector similarity for paraphrases and conceptual matches;
- keyword/full-text search for exact names, IDs, dates, rare terms;
- metadata filters for tenant, ACL, status, language, and time;
- reranking of a small candidate set;
- deduplication/diversity to avoid many near-identical chunks.

Apply access filters during retrieval, before text enters a prompt. Recheck access when rendering citations. A permission revocation needs a defined invalidation/filter policy.

## 5.4 Context and answer behavior

Pass only a small relevant authorized set. Include stable source IDs and human-readable metadata; do not ask the model to invent links. Limit context tokens and reserve room for question and output. Instruct the model to abstain when evidence is missing, then measure whether it does.

Separate answer text, source IDs, and abstention status in the response contract. Verify every citation was retrieved and authorized. A citation's presence does not prove it supports the answer.

## 5.5 Evaluate stages independently

Build a test set with questions and expected relevant source IDs. Measure retrieval recall@k/precision@k or another suitable retrieval metric separately from answer correctness, groundedness, citation support, completeness, and abstention.

Include:
- direct answer, paraphrase, exact keyword;
- answer absent from corpus;
- conflicting or outdated documents;
- long document/chunk boundary;
- cross-tenant query and revoked permission;
- malformed file or ingestion failure;
- prompt injection embedded in a document;
- provider outage and token limit.

Human review matters. Automated judges can scale comparison but are biased and can be fooled; record rubric, evaluator model/version, prompt, and human spot checks.

## 5.6 Spring implementation boundary

Use documented Spring AI embedding/vector integrations where useful. Keep orchestration in an application service; isolate parsers, embedding providers, and vector storage behind adapters. Persist job/source metadata using JDBC or JPA as appropriate. Object storage, SQL, and a queue usually do not share one ACID transaction; make the consistency/reconciliation design explicit.

## Practice

Build ingestion for licensed documents. Store source version and tenant metadata, retrieve with vector plus keyword search, return validated citations, and test unauthorized and unanswerable queries. Publish retrieval metrics and at least five failure cases.

## Questions and answers

**Does higher similarity mean the chunk answers the question?**  
No. Similarity is a ranking signal, not proof of relevance or entailment. Measure on your corpus.

**Can the model decide which tenant documents it may see?**  
No. Filter candidates using trusted identity and policy in backend code before constructing context.

**Why store parser and embedding versions?**  
Vectors depend on preprocessing and embedding model. Versioning supports reproducibility, migration, regression diagnosis, and reprocessing.

**Can the vector index be the only document store?**  
Usually not. You still need authoritative source/version metadata, ownership, deletion, and audit behavior.

## Mastery checkpoint

Build a small ingestion job, access-aware retrieval, cited response, and versioned evaluation set. Demonstrate a cross-tenant denial and an explicit abstention when evidence is missing.