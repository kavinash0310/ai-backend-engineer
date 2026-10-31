# Embeddings and semantic retrieval

## 1. What an embedding represents

An embedding model maps an input item (token, sentence, image, audio segment or document) to a vector in a learned space. Training encourages some relationships to be reflected geometrically—for example, related texts may have higher similarity than unrelated texts.

The vector coordinates are typically distributed features, not explicit labels. A nearest neighbor is similar under the learned representation and chosen metric; it is not guaranteed relevant, correct or permitted to be shown.

## 2. Similarity functions

For vectors q and d:
- dot product: q·d;
- cosine: (q·d)/(||q|| ||d||);
- Euclidean distance: sqrt(sum_i (q_i-d_i)^2).

Cosine ignores magnitude after normalization. Dot product includes magnitude. Euclidean distance emphasizes absolute geometric separation. The model's training objective and expected normalization often determine which metric is suitable. Test metric and normalization together; don't assume cosine is always best.

**Normalization** rescales vectors, commonly to unit length. With unit vectors, dot product equals cosine similarity. Zero vectors require special handling.

## 3. How text embedding models learn

A bi-encoder independently encodes query and document, allowing document vectors to be precomputed. Training may use contrastive objectives:
- positive pair: query and relevant passage;
- negative pair: irrelevant passage;
- loss encourages positive similarity above negative similarity.

Hard negatives are plausible but incorrect passages; they can improve discrimination but may contain false-negative labels. In-batch negatives treat other examples in the batch as negatives, which assumes those pairs are not actually relevant. Data construction is therefore central.

A cross-encoder jointly reads query and candidate passage and scores the pair. It can model interactions more directly but usually cannot precompute one reusable document vector, making it more expensive over large corpora. A common system uses a bi-encoder to retrieve candidates and a cross-encoder/reranker to reorder a small set.

## 4. Pooling, length and representation

Token-level encoders produce one vector per token. A sentence embedding needs a pooling rule, such as a designated token vector, mean pooling or a model-specific trained projection. The model's documentation matters: arbitrary pooling can damage retrieval quality.

Long input may be truncated or pooled differently. Keep chunks within supported token lengths and inspect truncation. Prefixes/instructions can be part of some embedding model's expected interface (for example, query versus passage mode); follow the model contract.

## 5. Retrieval evaluation

Build a labeled query set that includes:
- exact names/identifiers;
- paraphrases;
- ambiguous or underspecified questions;
- multiple relevant passages;
- no-answer cases;
- languages/domains expected in production.

Measures:
- recall@k: whether a relevant passage appears;
- precision@k: share of top-k that is relevant;
- MRR: rank of the first relevant passage;
- nDCG: graded relevance and rank position;
- latency and index resource use.

Measure embedding generation failure, dimension, normalization, query length and retrieval score distribution. Similarity thresholds are model/corpus-specific.

## 6. Approximate nearest-neighbor indexes

Exact nearest-neighbor search compares a query against all stored vectors; cost grows with corpus size and vector width. Approximate nearest-neighbor (ANN) indexes trade some recall for latency/memory.

Broad index families:
- **HNSW graph**: navigates a layered neighbor graph; fast retrieval under good tuning, with memory/build trade-offs.
- **IVF/inverted file**: partitions vectors into clusters and searches selected partitions; tuning balances recall and speed.
- **Product quantization**: compresses vectors into codes to reduce memory, introducing approximation.

Index parameters such as search breadth or partition count affect recall, latency and memory. Benchmark with representative queries and filtering. An index can have high recall for global search but poor recall after selective ACL filters, so test production filters.

## 7. Metadata filters and permissions

Vector similarity is not authorization. Apply trusted tenant/ACL filters from the caller identity. Some stores filter before ANN search, some afterward or with different execution behavior; understand recall and isolation implications. Do not retrieve broadly and rely on a model to ignore forbidden passages.

A document ACL update may need to update indexed metadata and cached results. Define revocation delay and verify that no stale vector result can cross tenant boundaries.

## 8. Versioning and migration

Record embedding model name/revision, tokenizer, dimensions, normalization, text preprocessing, query/document prefix and index configuration. Changing these can make stored and query vectors incompatible or alter ranking.

For migration:
1. build a new versioned vector set/index from source of truth;
2. evaluate old and new retrieval on fixed and recent query sets;
3. dual-run or shadow compare if safe;
4. switch reads with rollback;
5. remove old vectors after retention/permission review;
6. report cost and latency differences.

Never mix vectors from incompatible embedding spaces in one nearest-neighbor ranking unless a model explicitly supports that setup.

## 9. Common failure patterns

- A generic embedding model is used for a specialized language/domain without evaluation.
- Similarity score is treated as calibrated probability.
- Query and passage prefixes are omitted.
- Source passage is truncated by a token limit.
- Exact identifiers are missed because only vector retrieval exists.
- ANN filters reduce recall unexpectedly.
- Changed model but old vectors remain.
- Duplicate chunks dominate top-k.
- A threshold selected on developer demos fails on real queries.
- Private content is embedded by an unapproved external service.

## Questions and answers

**Q: What makes bi-encoders useful for search?**  
A: Documents can be embedded once and searched efficiently. The independent encoding limits some query-document interaction, so a reranker may refine candidates.

**Q: Are hard negatives always beneficial?**  
A: Only if they are true negatives and representative. A mislabeled relevant passage trains the model away from useful evidence.

**Q: How do I choose cosine versus dot product?**  
A: Follow model guidance and evaluate both with the correct normalization on labeled queries. The choice affects ranking and threshold scale.

**Q: Why retain lexical search?**  
A: Semantic vector representations can blur exact IDs, names, legal references and rare terms. Lexical retrieval complements them.

## Practice

1. Compute cosine/dot similarities for normalized and unnormalized vectors.
2. Build a labeled query set with exact, paraphrased and no-answer cases.
3. Compare bi-encoder top-k alone with a reranker over candidates.
4. Test ANN recall with and without tenant filters at several latency budgets.
5. Write an index migration plan after changing embedding model revision.

## Completion check

You can explain how embeddings are trained and used, distinguish bi-encoders and cross-encoders, choose/evaluate similarity metrics, and reason about ANN recall, filtering, versioning and access control.