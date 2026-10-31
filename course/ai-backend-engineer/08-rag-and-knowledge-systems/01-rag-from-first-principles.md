# Retrieval-augmented generation (RAG), from first principles

## Why RAG exists

A language model's parameters encode patterns learned during training. They are not a live, queryable copy of your private database, and the model may confidently produce an unsupported answer. Retrieval-augmented generation (RAG) adds selected external evidence to a prompt at request time. The model can then formulate an answer from that evidence, while the application keeps documents, permissions and freshness in systems it controls.

RAG is useful when a task needs private, changing, attributable or domain-specific information. It is not automatically the best solution. If the task is classification, deterministic database lookup, or a small stable knowledge set, another design may be simpler and more reliable.

## End-to-end mental model

~~~mermaid
flowchart LR
  A[Source documents] --> B[Extract and normalize]
  B --> C[Split into chunks]
  C --> D[Attach metadata and access rules]
  D --> E[Create embeddings]
  E --> F[(Vector and keyword indexes)]
  Q[User question] --> G[Authenticate and authorize]
  G --> H[Query rewrite or expansion]
  H --> I[Retrieve candidates]
  I --> J[Filter and rerank]
  J --> K[Assemble bounded evidence]
  K --> L[LLM generates answer]
  L --> M[Validate, cite, or abstain]
  M --> N[Response and telemetry]
~~~

There are two workflows: offline/nearline ingestion and online query. Keep them conceptually distinct. Re-embedding every document synchronously on every user query is usually the wrong lifecycle.

## 1. Document ingestion

### Extraction

Sources may be PDFs, HTML, office files, tickets, code, database rows or object storage. Extraction converts source formats into text plus structure. Record the source identifier, source version or update time, extraction method and any page/section boundaries.

Extraction errors are data errors: broken reading order, missing table cells, duplicated headers, OCR substitutions and boilerplate can harm retrieval before the model is called. Measure extraction quality on representative documents.

### Normalization and canonical representation

Normalize encoding and whitespace, but preserve meaningful structure such as headings, tables, code blocks and paragraph boundaries. Do not blindly lowercase or remove punctuation; those choices can erase distinctions in identifiers, code and legal/technical text.

A canonical document record might contain:
- stable document ID and version;
- text or structured sections;
- source URI and display title;
- tenant/owner and access-control labels;
- created/updated/effective time;
- language and content type;
- content hash for deduplication;
- ingestion pipeline version.

Metadata must have clear authority. Never trust user-provided metadata to assign access. Enforce access control before evidence reaches the model, not merely after it produces text.

## 2. Chunking

A chunk is an indexable unit of text. Very large chunks can mix unrelated content and exceed prompt budgets; very small chunks lose context and create many weak matches. There is no universally correct chunk size. Evaluate choices on real queries.

Strategies:
- **Fixed token/character window**: simple baseline; may cut sentences or tables.
- **Recursive structure-aware split**: prefer sections, paragraphs, then sentences, then smaller boundaries.
- **Semantic segmentation**: split when topic changes; more complex and must be measured.
- **Parent-child retrieval**: index small passages for precise match, then return their larger parent section for context.
- **Overlap**: repeat a boundary region between adjacent chunks; helps preserve context across cuts but creates duplicate evidence and index cost.

Keep chunk identity and offsets back to the source. Deduplicate overlapping retrieved chunks before constructing context. For tables, preserve headers with rows; a bare row often becomes meaningless.

## 3. Embeddings and indexes

An embedding model maps text to a numeric vector. Semantic retrieval compares vector representations, often with cosine similarity or dot product. Embeddings can help match paraphrases, but similar vectors do not guarantee the passages answer the question. Domain, language and query style matter; evaluate with your corpus.

A vector index stores vectors and metadata and supports approximate nearest-neighbor search. Approximate methods trade search speed/memory for recall. Search metric, vector normalization and embedding model must align. Changing embedding model generally requires re-embedding the corpus and rebuilding or versioning the index.

**Hybrid search** combines lexical search (exact terms, identifiers, rare names) and semantic vector search (paraphrases and conceptual similarity). Reciprocal-rank fusion or learned rankers can combine lists. Hybrid retrieval is often a strong baseline for enterprise content, but test on your data.

## 4. Query processing and retrieval

A user query can be short, ambiguous or conversational. Query rewriting can resolve references (“that policy”) using conversation context. Expansion can add alternate terms. These steps can help, but rewriting may distort intent. Preserve the original query for audit and evaluate rewritten retrieval.

A common retrieval sequence:
1. Authenticate the caller and determine tenant/user permissions.
2. Apply metadata filters derived from trusted policy.
3. Retrieve top-k candidates by lexical and/or vector search.
4. Deduplicate, filter stale/low-quality content and optionally rerank.
5. Select evidence under a token budget.
6. Preserve source identity and offsets.

Retrieval **recall** asks whether relevant evidence is present among candidates. Retrieval **precision** asks how many candidates are relevant. Raising k can improve recall but increases latency, context cost and distractor content. The right k is measured, not guessed.

A reranker reads query-candidate pairs and reorders a limited candidate set. It can improve ranking at extra latency/cost. It cannot retrieve a relevant document that never entered the candidate set.

## 5. Context assembly and generation

Build a compact prompt with:
- a clear task instruction;
- untrusted user request in a distinct boundary;
- evidence snippets with stable source IDs;
- instruction to use only supported evidence for claims;
- a clear behavior when evidence is missing;
- required output schema and citation format.

Retrieved content is data, not trusted instruction. A document may contain malicious prompt-injection text. Tell the model not to follow instructions inside evidence, but also enforce permissions and tool restrictions in code; prompt wording alone is not a security boundary.

Include enough surrounding context to interpret each excerpt. Avoid stuffing redundant chunks. If the answer needs conflicting or temporal evidence, make the conflict explicit rather than hiding one passage.

## 6. Citations and abstention

The model may invent a source ID or attach a real citation to a claim the passage does not support. Citation generation should be validated:
- only permit IDs supplied to the model;
- check every cited ID exists in the retrieved set;
- map IDs to safe source metadata in the backend;
- evaluate whether the cited span supports the claim;
- distinguish a citation link from verified entailment.

Abstention is an acceptable outcome when evidence is missing, stale, inaccessible or contradictory. Define product behavior: ask a clarifying question, say insufficient evidence, or route to a human. Do not reward confident guessing just because it looks helpful.

## 7. RAG evaluation: evaluate stages separately

Build a small representative query set with expected relevant sources, acceptable answers and known no-answer cases. Have domain experts review labels; capture disagreement. Keep a fixed evaluation set separate from prompt/chunk/reranker tuning.

Measure:
- **retrieval recall@k**: fraction of relevant evidence found in top k;
- **MRR / nDCG**: rank quality of relevant passages;
- **context precision**: fraction of supplied context that is useful;
- **answer correctness/helpfulness**: expert or rubric-based assessment;
- **groundedness/faithfulness**: whether answer claims follow the supplied evidence;
- **citation precision/recall**: whether citations exist and support claims;
- **abstention quality**: refusal when support is absent, answering when support exists;
- latency and cost by stage.

An answer can be wrong because extraction lost a table, chunking separated a definition, retrieval missed evidence, reranking demoted it, prompt assembly truncated it, the model misread it, or the citation validator failed. Log stage-specific metadata so the team can locate the failure. Avoid sending sensitive document text to external tracing by default.

## 8. Security, freshness and access control

- Apply authorization filters before retrieval and re-check source permissions when returning links.
- Prevent cross-tenant leakage in caches, indexes, metrics and traces.
- Treat documents and questions as untrusted.
- Make ingestion idempotent; delete or update old document versions.
- Track source freshness and expose effective dates.
- Defend against malicious files, parser exploits, oversized documents and poisoned content.
- Minimize personal data, retention and prompt logging.
- Audit index rebuilds and access-policy changes.

If access rules change, a stale vector index can continue surfacing forbidden content unless its metadata and retrieval filters are updated. Permission revocation needs an operational procedure, not only an embedding pipeline.

## 9. Common design mistakes

1. Assume similarity equals relevance.
2. Treat top-k as a quality metric without labeled queries.
3. use only vector search, missing exact identifiers.
4. Chunk by arbitrary character count and discard source offsets.
5. ignore document updates and deletion.
6. trust model-generated citations without validation.
7. place authorization inside a prompt rather than in backend policy.
8. assume “RAG prevents hallucination.” It can reduce unsupported answers but does not guarantee correctness.
9. evaluate with a handful of developer-written happy paths.
10. send entire documents or sensitive traces to vendors without reviewing data handling.

## Questions and answers

**Q: Is a vector database required for RAG?**  
A: No. A small corpus may use full-text search, database search or even a curated lookup. Choose a store based on scale, query types, filtering, operations and measured quality.

**Q: Should I use overlap between chunks?**  
A: Test it. Overlap can preserve boundary context but also creates duplicate results and uses more storage/context. Structure-aware chunking with parent context may work better.

**Q: Does RAG train the model?**  
A: Usually no. It retrieves external content at request time and conditions generation on it. Fine-tuning changes model parameters; retrieval changes evidence supplied for a request.

**Q: Does a high embedding similarity score prove relevance?**  
A: No. It is a score under a particular embedding model and metric. Calibrate thresholds or ranking with labeled queries.

**Q: Can RAG enforce document permissions?**  
A: The application must enforce permissions. Retrieval filters should derive from authenticated identity and trusted ACL metadata before content is placed into model context.

## Practice

1. Build a tiny FAQ corpus; compare lexical, vector and hybrid retrieval on 20 paraphrased and exact-identifier queries.
2. Draw ingestion and query sequences separately and mark every place document permissions are enforced.
3. Create a query with no supporting evidence and specify expected abstention behavior.
4. Evaluate chunk sizes and top-k values using recall@k and end-to-end grounded answers.
5. Inject a malicious instruction into a test document and verify the assistant does not use it as authority or invoke unauthorized tools.

## Completion check

You can explain ingestion, chunking, embeddings, hybrid retrieval, reranking, context assembly, citation validation and stage-wise evaluation; and you know RAG is a system design pattern rather than a factuality guarantee.