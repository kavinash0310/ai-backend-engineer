# Project guide: evidence-grounded AI knowledge assistant

## Project objective

Build a small but production-minded service that answers questions from a controlled document collection, cites its evidence, respects tenant access, and reports when the corpus does not support an answer. The learning value is the complete engineering cycle: backend APIs, data ingestion, retrieval, model integration, evaluation, security and operations.

The project should begin with synthetic/public non-sensitive documents. Never upload employer or customer data without authorization.

## User stories and non-goals

User stories:
1. An authorized user can add or update a document.
2. An authorized user can ask a question and receive an answer with valid source references.
3. A user with no evidence or insufficient access receives a clear abstention.
4. An operator can see request status, latency stage, token usage and quality/evaluation version without seeing private content by default.
5. An operator can remove a document and verify that it is no longer retrievable.

Non-goals for the first version: autonomous write actions, training a foundation model, processing arbitrary unbounded files, and claiming the answer is guaranteed correct.

## Architecture

~~~mermaid
flowchart LR
  UI[Client] --> API[Spring Boot API]
  API --> AUTH[Identity, tenant, limits]
  AUTH --> CHAT[Question orchestration]
  CHAT --> SEARCH[Hybrid retrieval service]
  SEARCH --> DB[(Postgres / search index)]
  CHAT --> LLM[Model provider adapter]
  LLM --> MODEL[Hosted or local model]
  API --> JOB[Ingestion job API]
  JOB --> QUEUE[Bounded worker queue]
  QUEUE --> PARSE[Parser and chunker]
  PARSE --> EMBED[Embedding model]
  EMBED --> DB
  API --> OBS[Trace, metrics, audit metadata]
~~~

Keep the first version as a modular application if that is enough; extraction into separate services is an optional scale step justified by measurements. The diagram shows responsibility boundaries, not a demand for microservices.

## Stage 0: repository and learning contract

Create a README with:
- what the service does and does not promise;
- setup and configuration using a secret-free example file;
- architecture diagram;
- API examples;
- data lifecycle and deletion procedure;
- evaluation method and known limitations;
- deployment and rollback steps.

Create ADRs for the model provider, storage/index, chunking, permissions and synchronous/asynchronous boundaries. Each decision record should state context, decision, alternatives, consequences and date.

## Stage 1: deterministic baseline before LLM

Implement document metadata storage and lexical search first. Define a small evaluation set with queries, relevant source IDs, and no-answer cases. Report recall@k, MRR/nDCG if useful, and response latency. This proves retrieval can return relevant evidence before an LLM is introduced.

Acceptance:
- document IDs are stable and versioned;
- update/delete is idempotent;
- access filtering occurs before results leave the repository;
- exact identifiers and keywords work;
- evaluation cases cover paraphrase, exact lookup, ambiguity and no answer.

## Stage 2: ingestion and chunking

Support a narrow set of safe file types at first (plain text/Markdown, then a chosen PDF path). Validate type and size; do not trust file extensions alone. Extract text, preserve headings, store page/section offsets, compute a content hash, and record parser/version metadata.

Chunk using structure boundaries, then enforce a token budget. Keep chunk IDs, parent document/version, start/end offsets, ACL metadata and display title. Add overlap only if retrieval evaluation demonstrates a benefit. Handle parse failure explicitly; do not index an empty document as if it succeeded.

Acceptance:
- retries do not duplicate chunks;
- update replaces or versions prior chunks;
- delete removes vector and lexical representations and cache entries;
- parser errors are visible to the user/operator without leaking stack traces;
- extraction quality is reviewed on sample files.

## Stage 3: embeddings and hybrid retrieval

Choose an embedding model based on language, license, deployment, dimensions and measured retrieval quality. Version the model identifier and normalization policy. Build vector retrieval and combine with lexical retrieval for identifiers and rare terms. Apply trusted ACL filters before context assembly. Optionally rerank the top candidate set.

Compare:
- lexical-only;
- vector-only;
- hybrid;
- hybrid plus reranking;
- several chunk and top-k settings.

Keep a fixed evaluation set. Do not pick settings from anecdotal demos. Report retrieval recall, precision/quality at k, latency and index size.

## Stage 4: grounded generation

Construct the prompt from system policy, the user's question and retrieved evidence with stable reference IDs. Tell the model to answer only when evidence supports the claim and to state uncertainty otherwise. Treat retrieved text as untrusted content, not instructions.

Require a structured response:
- answer text;
- cited reference IDs;
- insufficient-evidence flag or answer status;
- optional short supporting excerpt IDs.

Validate output shape and every cited ID server-side. A citation ID must belong to the retrieved set and caller's permissions. Do not let the model create source URLs or decide access.

Acceptance:
- every returned citation maps to a displayed source/section;
- unknown IDs are rejected or removed and trigger a safe status;
- answer is withheld/abstained for test cases with no supporting evidence;
- retrieved prompt-injection text cannot invoke tools or override backend access rules.

## Stage 5: evaluation harness

Use a versioned JSONL or tabular evaluation set with:
- query and authorized user profile/tenant;
- expected relevant document/chunk IDs;
- expected answer facts or rubric;
- acceptable abstention status;
- language and difficulty tags.

Measure each component separately:
1. retrieval recall/ranking;
2. context quality and token usage;
3. citation validity and support;
4. answer correctness and groundedness;
5. abstention behavior;
6. latency and cost.

Use deterministic checks where possible (citation exists, ACL, schema); use expert review for semantic correctness. Model-as-judge can help triage but has bias and variance: calibrate it against human judgments and do not present its score as ground truth.

Prevent evaluation contamination: developers should not repeatedly tune to the final test set. Keep a development set, a fixed holdout and, later, a time-based prospective set.

## Stage 6: API and service behavior

A useful API surface might include:
- POST document metadata/upload → ingestion job ID;
- GET ingestion job → state and safe error category;
- POST question → answer, evidence, trace ID and usage;
- DELETE document version → deletion status;
- GET readiness/health → dependency state without exposing secrets.

Set request-byte and token limits, authenticated tenant, quotas, deadlines, bounded queue, idempotency key and clear error categories. Separate liveness from readiness: a process can be alive while its model/retrieval dependencies are not ready.

If streaming answers, define partial citation behavior, disconnect cancellation, safety checks on partial output, terminal events and retry behavior.

## Stage 7: security and privacy review

Threat-model malicious users, malicious documents, cross-tenant access and provider compromise. Implement:
- authenticated principal-derived tenant ID;
- document ACL filtering at query time;
- per-tenant cache keys;
- output escaping and schema validation;
- secret manager for provider credentials;
- no raw prompt logging by default;
- retention, deletion and data export behavior;
- quotas and rate limits;
- prompt-injection evaluation cases;
- dependency/artifact provenance.

Do not grant the assistant write tools in the first release. If later required, add them as separate scoped capabilities with approval and audit.

## Stage 8: reliability, operations and capacity

Measure realistic document sizes, question lengths, output lengths and concurrency. Record ingestion lag, queue depth, parser failures, retrieval latency, model TTFT, completion latency, token counts, errors and per-stage timeouts.

Load test with synthetic documents and scripted queries. Establish an SLO meaningful for the user journey. Define what happens if the model provider is unavailable, search is unavailable, or the queue is full. Keep a deterministic fallback where it is useful, such as keyword results with source links.

Version model, embeddings, prompt, chunking pipeline and index schema. Roll out changes against the evaluation set, canary a small fraction, compare failures, then expand or roll back.

## Suggested implementation milestones

1. Markdown corpus, lexical search and CLI evaluation.
2. Spring Boot question/document APIs with auth stub only for local synthetic use.
3. Structured ingestion job and durable metadata.
4. Embeddings and vector search; compare against lexical baseline.
5. Grounded model response with validated citations and abstention.
6. Tenant ACL, deletion, prompt-injection tests and privacy-minimized observability.
7. Containerized deployment, load profile, runbook and rollback.
8. Portfolio demo and architecture review.

Build each milestone in a separate branch/PR if working in a team. Keep commits reviewable and document how to reproduce measurements.

## Portfolio evidence

The project should demonstrate:
- a runnable, documented vertical slice;
- architecture and sequence diagrams;
- an ADR with trade-offs;
- an evaluation dataset and results;
- a threat model and security tests;
- latency/cost worksheet using measured inputs;
- service dashboard/runbook and failure demo;
- reflection on what the system cannot guarantee.

A polished UI cannot substitute for evidence of retrieval quality, permissions, failure handling and reproducibility.

## Interview walkthrough

Present it in 8 minutes:
1. user problem and chosen constraints;
2. architecture and why each boundary exists;
3. how retrieval is built and evaluated;
4. how you prevent cross-tenant leaks and prompt-injection impact;
5. measured quality, latency and cost;
6. one failure you found and changed;
7. remaining limitations and the next experiment.

## Completion gate

A reviewer can clone the project, understand its scope, reproduce evaluation, observe citations and abstention, verify permissions/deletion behavior, inspect failure handling, and find no secrets or private data in the repository.