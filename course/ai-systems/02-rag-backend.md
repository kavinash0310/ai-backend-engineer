# Building a retrieval-augmented generation backend

## RAG is a multi-stage data product

Retrieval-augmented generation (RAG) supplies relevant external passages to a language model at answer time. It is useful when the answer should be grounded in a changing or private corpus. It does not guarantee factuality: search can retrieve the wrong text and the generator can misread correct text.

~~~mermaid
flowchart LR
  A[Upload] --> B[Validate and authorize]
  B --> C[Extract and normalize]
  C --> D[Chunk and attach metadata]
  D --> E[Embed and index]
  Q[Question] --> R[Authenticate and authorize]
  R --> S[Retrieve allowed chunks]
  S --> T[Rank and assemble context]
  T --> U[Generate answer with citations]
  U --> V[Validate and return]
~~~

Ingestion is usually a background job. Store original files in object storage and metadata/status in a relational database. Validate file type, size, ownership, and parser behavior. Attach document version, tenant, source, section, timestamp, and permission metadata to every chunk.

## Chunking and retrieval

A chunk is a passage indexed independently. Too-small chunks may lose context; too-large chunks may dilute matches and exceed context budgets. Split on document structure where possible, preserve titles and section headings, and test several sizes on a small evaluation set.

Embedding models map text into vectors where semantically related passages may be near each other. Vector search returns approximate similarity candidates. Lexical search is often better for exact names, codes, and numbers; hybrid retrieval can combine both. A reranker can reorder a shortlist using a stronger query-passage score but adds latency.

At query time, authenticate first. Apply tenant and document permissions before model context is assembled. Then retrieve candidates, rerank, enforce a context budget, call the model, and return source IDs or links. A vector similarity score is not an authorization decision.

## Evaluation by stage

Measure retrieval separately: did a relevant passage appear in top k? Measure generation: are answer claims supported by evidence? Measure citations: does each citation support the attached claim? Include no-answer questions, stale documents, conflicts, and documents the user cannot access.

When an answer fails, identify which stage failed: extraction, chunking, indexing, permission filter, retrieval, ranking, context assembly, generation, or citation formatting. Changing the prompt cannot fix a missing source document.

## Worked example

For a support knowledge base, ingest versioned policy documents. A user asks about a refund. The backend verifies tenant and role, retrieves only current accessible refund sections, reranks them, and asks the model to answer only from evidence. It returns an answer and source references. If no relevant current passage is found, it says the source does not answer and offers escalation instead of guessing.

Questions: Why preserve source metadata? The application needs citations, version checks, permissions, and debugging. Why evaluate retrieval separately? It distinguishes search failure from generation failure.

Practice: write ten answerable, five unanswerable, and five adversarial queries. Record relevant chunks, expected answer claims, access rules, retrieval recall at five, citation correctness, and abstention.

Further reading: [AI From Scratch RAG lesson](https://github.com/kavinash0310/ai-from-scratch/blob/main/course/generative-ai/02-rag.md) and [Google ML monitoring guidance](https://developers.google.com/machine-learning/crash-course/production-ml-systems/monitoring).
