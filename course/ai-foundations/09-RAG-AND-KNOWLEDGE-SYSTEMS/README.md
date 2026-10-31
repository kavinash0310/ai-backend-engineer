# RAG and Knowledge Systems

RAG combines an ingestion/index pipeline with a permission-aware retrieval and generation path. It does not guarantee truth.

## Scope

Parsing/OCR, metadata, chunking, embeddings, vector indexes (HNSW/IVF concepts), exact/approximate search, cosine/dot/L2 metrics, BM25, hybrid retrieval, reranking, query rewrite/decomposition, parent-child retrieval, context compression, citations, abstention, GraphRAG, agentic RAG, evaluation, security, freshness, deletion, observability and cost.

Compare PostgreSQL full-text search, pgvector, FAISS, Chroma, Qdrant, Weaviate and Milvus using corpus size, filtering, latency, operations, backup, consistency, and benchmarks. Do not pick by popularity alone.

Primary reference: [AI Backend Engineer RAG chapters](https://github.com/kavinash0310/ai-backend-engineer/tree/main/course/java-spring), plus [AI From Scratch GenAI](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/generative-ai).


## Detailed lesson

- [RAG from first principles](01-rag-from-first-principles.md) — ingestion, chunking, embeddings, hybrid search, permissions, citations, evaluation and security.
- Practice end-to-end in the [AI knowledge assistant project guide](../15-PORTFOLIO-PROJECTS/01-ai-knowledge-assistant-project-guide.md).

- [Embeddings and vector search](02-embeddings-and-vector-search.md) — bi-encoders/rerankers, similarity, ANN, evaluation and versioning.
