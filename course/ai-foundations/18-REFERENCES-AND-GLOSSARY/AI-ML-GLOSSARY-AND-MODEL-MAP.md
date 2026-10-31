# AI/ML glossary and model-selection map

This page is a working reference, not a substitute for the lessons. Learn each term in context, then return here to check definitions. For a larger companion field reference see [AI From Scratch: AI/ML Field Reference](https://github.com/kavinash0310/ai-from-scratch/blob/main/course/AI-ML-FIELD-REFERENCE.md).

## Model and method map

“Efficiency” can mean compute, memory, latency, data need, interpretability, or cost per useful result. No method wins on all of them. The table describes typical tendencies; dataset size, implementation and hardware can reverse them.

| Family | Common use | Typical strengths | Typical cost / limitation |
|---|---|---|---|
| Linear regression | Numeric prediction | Fast, simple, interpretable baseline | Misses nonlinear structure unless features encode it; sensitive to assumptions/outliers |
| Logistic regression | Classification and probability baseline | Fast, compact, useful calibrated baseline after calibration | Linear decision boundary in feature space |
| Naive Bayes | Text classification, small datasets | Very fast, low memory, works with sparse counts | Strong conditional-independence assumption; probability calibration may be weak |
| k-nearest neighbors | Similarity classification/regression | Simple, no complex training, uses local examples | Prediction/search can be expensive; scaling and metric matter |
| Decision tree | Tabular classification/regression | Human-readable small trees, nonlinear splits | Unstable/high variance without constraints; large trees hard to interpret |
| Random forest | Tabular baseline | Robust nonlinear ensemble, parallelizable prediction | Larger memory/model; less transparent; may be weaker than boosted trees on some tabular tasks |
| Gradient-boosted trees | Structured/tabular data | Strong on many tabular problems, handles nonlinear interactions | Tuning and sequential training; model explanation needs care |
| Support vector machine | Margin classification, moderate data | Effective in high-dimensional spaces with suitable kernel | Kernel training can scale poorly; probability output needs additional calibration |
| k-means | Clustering | Simple, fast baseline with numeric vectors | Requires k; assumes distance geometry/cluster shape; sensitive to scaling and initialization |
| PCA / SVD | Compression, exploration, denoising | Reduces dimensions and exposes dominant variance directions | Linear; high-variance directions are not always task-relevant |
| MLP / feed-forward net | Tabular or vector inputs | Flexible nonlinear function approximator | Often needs tuning/data; weaker interpretability and resource cost than simple baselines |
| CNN | Images and local spatial/time patterns | Parameter sharing and local receptive fields exploit structure | Architecture/task-specific; not all signals are spatially local |
| RNN / LSTM / GRU | Sequential signals | Sequential state and variable-length sequences | Less parallel training; long-range dependencies can be hard |
| Transformer encoder | Text understanding, representation, classification | Bidirectional context and parallel sequence processing | Attention cost/memory grows with sequence length; task and scale matter |
| Decoder-only Transformer | Text generation, chat, code | One next-token objective supports flexible generation | Autoregressive token generation is sequential; can hallucinate and use substantial memory |
| Encoder-decoder Transformer | Translation, transformation, summarization | Separate input encoding and output generation | More components/compute; less universal tooling than decoder-only in some ecosystems |
| Embedding model | Search, clustering, matching | Compact semantic vectors and fast approximate retrieval | Similarity is not proof; domain/language mismatch and freshness matter |
| Diffusion model | Image/audio generation | High-quality iterative generation and flexible conditioning | Multiple denoising steps can be slow; evaluation/control can be difficult |
| Graph neural network | Relationships and graph data | Uses node/edge structure directly | Neighbor sampling, oversmoothing and graph scale complicate training/serving |
| Recommender models | Ranking and personalized suggestions | Optimizes relevance across users/items/context | Feedback loops, cold start, privacy and ranking bias; online serving complexity |
| Reinforcement learning | Sequential action under reward | Optimizes cumulative reward and interaction strategy | Expensive/unstable; reward misspecification and safety are hard |
| Retrieval (BM25/vector/hybrid) | Finding evidence/documents | Fresh corpus and source traceability; no weight training required for each fact | Retrieval misses/permission issues; answer quality depends on index and query |

### How to choose

1. Define the prediction or generation job and decision.
2. Choose a split matching future use.
3. Establish the simplest baseline.
4. Define quality, latency, memory, cost, privacy and interpretability requirements.
5. Compare candidates on a representative held-out set and realistic serving load.
6. Select a model plus threshold/abstention/fallback policy.
7. Document evidence, limits and rollback trigger.

Avoid statements such as “Transformers are always best” or “random forests are efficient” without saying which resource and workload you mean.

## Core AI and machine-learning terms

- **AI (artificial intelligence):** broad field of systems performing tasks associated with perception, language, planning or decision-making.
- **Machine learning:** methods that fit behavior from data rather than encode every rule manually.
- **Deep learning:** ML using neural networks with multiple learned transformations.
- **Generative model:** estimates or produces data-like outputs, such as tokens or images.
- **Foundation model:** broadly pretrained model adaptable to many downstream tasks; term is not a guarantee of quality or scope.
- **Algorithm:** procedure for learning or inference.
- **Model:** fitted parameters plus architecture/configuration and runtime behavior.
- **Parameter:** learned numeric value inside a model.
- **Hyperparameter:** training/design choice selected outside ordinary parameter fitting.
- **Feature:** input representation used by a prediction model.
- **Label/target:** desired output used as a supervised signal.
- **Inference:** applying a fitted model to new input.
- **Training:** fitting parameters from data and objective.
- **Fine-tuning:** additional training from pretrained weights for a target distribution/task.
- **Pretraining:** broad initial training, often with self-supervision.
- **Generalization:** performance on relevant unseen cases.
- **Overfitting:** fitting idiosyncrasies of training data that do not carry to new cases.
- **Underfitting:** model/optimization fails to capture useful patterns, even on training data.
- **Bias-variance trade-off:** conceptual balance between restrictive assumptions and sample-sensitive estimates.
- **Baseline:** simple reference method against which added complexity is compared.
- **Data leakage:** information from outside the legitimate prediction process contaminates fitting/evaluation.
- **Distribution shift:** evaluation or serving data differs from training assumptions.
- **Calibration:** agreement between predicted probability and observed frequency.
- **Threshold:** cutoff converting scores/probabilities into a decision.
- **Precision:** fraction of positive predictions that are correct.
- **Recall/sensitivity:** fraction of actual positives detected.
- **Specificity:** fraction of actual negatives correctly rejected.
- **F1:** harmonic mean of precision and recall.
- **ROC-AUC:** ranking summary across false-positive/true-positive thresholds.
- **PR-AUC:** precision-recall ranking summary, often useful for rare positives.
- **Cross-validation:** repeated train/holdout evaluation across folds under a chosen splitting assumption.
- **Regularization:** training constraint/penalty intended to discourage undesirable complexity.
- **Gradient:** vector of partial derivatives of a scalar objective with respect to parameters.
- **Backpropagation:** chain-rule computation of gradients through a network.
- **Optimizer:** rule for parameter updates using gradients.
- **Epoch:** one pass over the training dataset.
- **Batch:** group processed together in one update.
- **Embedding:** learned vector representation of an item/token/document.
- **Token:** tokenizer vocabulary unit; not necessarily a word.
- **Tokenizer:** mapping between text and token IDs.
- **Logit:** raw model score before probability transform.
- **Softmax:** transformation of scores into nonnegative values summing to one.
- **Attention:** learned weighted mixing of sequence representations.
- **Context window:** maximum token sequence supported/accepted by a model configuration.
- **KV cache:** stored key/value tensors reused during autoregressive decoding.
- **Quantization:** lower-precision representation of model values to reduce storage/compute, with potential quality trade-offs.
- **Hallucination:** plausible output unsupported or contradicted by the relevant facts/evidence.
- **RAG:** retrieval-augmented generation; retrieve external evidence and condition a model on it.
- **Chunk:** bounded segment of a document used for indexing/retrieval.
- **Reranker:** model that scores query-candidate pairs to reorder retrieved passages.
- **Groundedness/faithfulness:** degree to which an answer's claims follow the supplied evidence.
- **Abstention:** intentionally not answering when support or confidence is inadequate.
- **Agent:** model-in-the-loop control system that can select actions and iterate under constraints.
- **Tool/function call:** structured proposal for an application capability; backend still validates and authorizes it.
- **MCP:** protocol for connecting AI applications with servers that expose tools/resources/prompts.
- **Prompt injection:** hostile content attempting to steer model behavior or misuse tools.
- **Evaluation set:** fixed, versioned examples used to assess system behavior.
- **SLO:** service-level objective for an operational measure such as latency or availability.
- **Drift:** change over time in input, prediction, label or error behavior.
- **Idempotency:** repeated operation with same identifier has one intended effect.
- **Lineage:** trace of where data/artifacts came from and how they were transformed.

## LLM and RAG model quality vocabulary

- **Perplexity:** exponentiated average next-token negative log likelihood; useful for language-model distribution fit but not a direct measure of helpfulness or truth.
- **Exact match:** prediction string equals target under a defined normalization.
- **Semantic similarity:** vector/model-based similarity; does not establish factual equivalence.
- **Citation precision:** share of supplied citations that are valid/supporting under a rubric.
- **Citation recall:** share of claims requiring evidence that receive relevant citations.
- **Retrieval recall@k:** share of known relevant evidence present in top k results.
- **MRR:** reciprocal rank of the first relevant result, averaged across queries.
- **nDCG:** ranking measure that accounts for graded relevance and rank position.
- **TTFT:** time to first generated token.
- **Inter-token latency:** interval between output tokens.
- **Tokens per second:** generation throughput under a specified workload and measurement window.
- **Cost per successful task:** total relevant compute/provider/operation cost divided by completed useful tasks.

## Important distinctions

- Model score ≠ calibrated probability.
- Retrieval similarity ≠ evidence support.
- Valid JSON ≠ authorized/correct action.
- Training loss ≠ product quality.
- A successful HTTP response ≠ a correct answer.
- A benchmark result ≠ performance for every language or user group.
- A large context window ≠ reliable use of every included passage.
- An open-weights license ≠ unrestricted commercial use; read the actual license.
- A model card or system card documents claims/limits; it does not replace independent evaluation.