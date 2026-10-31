# Natural language processing foundations

## 1. What NLP studies

Natural language processing builds systems that process text and language. Tasks include classification, sentiment, entity extraction, search, translation, summarization, question answering, speech-text interfaces and generation. A task definition needs input, output, label/source of truth, target population and error costs.

Language is ambiguous, contextual, multilingual, evolving and full of formatting variation. A model may learn spelling, topic or source artifacts instead of intended meaning. Evaluate realistic inputs and inspect language/domain slices.

## 2. Text preparation is task-dependent

Common operations include Unicode normalization, whitespace cleanup, sentence segmentation, tokenization, stemming and lemmatization. None is universally beneficial:
- lowercasing can merge useful distinctions such as product codes;
- stemming can damage names and technical tokens;
- stop-word removal can erase negation;
- punctuation can encode sentiment, code structure or identifiers;
- aggressive normalization can erase multilingual distinctions.

Preserve the original source and version transformations. Use training-fitted vocabularies/statistics for leakage-sensitive transforms.

## 3. Bag of words, n-grams and TF-IDF

A bag-of-words vector counts or weights terms while largely ignoring order. An n-gram is a contiguous sequence of n tokens; bigrams can capture phrases such as “not useful.” The representation is sparse and often high-dimensional.

Term frequency-inverse document frequency (TF-IDF) weights terms by their frequency in a document and rarity across a corpus. It is a strong, inexpensive retrieval and classification baseline. Vocabulary is fitted on the training/index corpus and versioned. Rare entities and new vocabulary behave differently from dense embeddings.

## 4. Language tasks and labels

- **Text classification** assigns a label to a whole input.
- **Token classification** assigns labels to tokens, such as entity tags.
- **Sequence-to-sequence** maps one sequence to another, such as translation.
- **Language modeling** predicts token distributions conditioned on context.
- **Information extraction** maps text into structured fields or relations.

Annotation conventions must match the task. For entity spans, define tokenization and overlap behavior. For summaries, exact string match is too crude; review factuality, omissions, coverage and risk.

## 5. Static and contextual representations

Static word embeddings map a vocabulary entry to one vector. Polysemous words receive the same vector across contexts. Contextual encoders produce token representations that depend on surrounding text. Sentence/document embedding models map larger text spans to vectors for similarity or retrieval.

Embedding similarity can find paraphrases but may miss exact identifiers, negation, freshness or permission. Evaluate lexical, vector and hybrid methods on representative queries.

## 6. Tokenization and subwords

Subword tokenizers divide rare or novel words into reusable pieces. Byte-pair encoding is one family that repeatedly merges frequent adjacent symbols; unigram and byte-level methods make different trade-offs. The tokenizer vocabulary and normalization affect token IDs and token count.

A tokenizer is part of the model interface. Changing it changes representation. Some languages, scripts, code and unusual strings require more tokens than everyday English, affecting context budget and serving cost. Use the matching tokenizer when estimating budgets.

## 7. Transformer contextualization and evaluation

A Transformer builds contextual token representations through attention and feed-forward blocks. An encoder may use both left and right context; a causal decoder uses prior context. See [Transformers and attention](../08-GENERATIVE-AI-AND-LLMS/01-transformers-and-attention.md) for the full Q/K/V derivation, masks and worked calculation.

Metrics should follow task meaning:
- classification: confusion matrix, precision/recall, calibration and subgroup errors;
- search: recall@k, MRR/nDCG and human relevance labels;
- translation: automatic metrics plus human adequacy/meaning review;
- generation: task completion, factuality, omissions, safety and human utility;
- extraction: field/span precision-recall and schema-valid rate.

Reference answers can be incomplete or disputed. Version evaluation sets and rubrics; assess reviewer disagreement. Automated language scores are proxies, not proof of usefulness or truth.

## Questions and answers

**Are stop words always removed?** No. Common words and negations can carry meaning. Test task-specific preprocessing and keep the original data.

**Does an embedding understand a sentence?** It learns a representation useful for some similarity behavior. Similarity is not factual verification or complete reasoning.

**Why combine lexical and vector search?** Lexical retrieval handles identifiers and exact rare terms; vectors can help paraphrases. Hybrid search combines them when evaluation shows value.

## Practice

1. Build a TF-IDF plus logistic-regression baseline on a small text dataset.
2. Compare unigrams and bigrams on examples involving negation.
3. Write a normalization policy that preserves product IDs and multilingual text.
4. Label relevant documents for 20 queries and compare lexical/vector retrieval.
5. Define a summary rubric scoring coverage and unsupported claims.

## Completion check

You can distinguish common text tasks, explain sparse lexical and dense contextual representations, and select evaluation methods tied to task meaning.