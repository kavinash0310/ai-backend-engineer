# Project 2: grounded RAG service

## Goal

Build a question-answering API over a small, permission-scoped document collection. The backend must retrieve evidence, cite it, and abstain when the corpus does not support an answer.

## Requirements

- Upload or ingest documents asynchronously with size/type checks.
- Store original files outside the relational database and record metadata/status in PostgreSQL.
- Extract text, chunk it, retain source, version, tenant, and access metadata.
- Index embeddings in a vector search system; add lexical search if useful.
- Authenticate the caller and filter retrieval before constructing model context.
- Return citations that point to the supporting source passages.
- Add tests for missing evidence, stale versions, cross-tenant access, provider timeout, and prompt injection inside a document.
- Evaluate retrieval separately from answer support and citation correctness.

## Report

Include the retrieval method, chunk policy, test questions, top-k recall, citation score, no-answer behavior, latency, cost, and known limitations. Do not claim that RAG guarantees factual answers.
