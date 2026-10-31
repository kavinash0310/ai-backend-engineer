# AI backend engineering interview question bank

Use two passes for every question: first answer in 30–60 seconds, then expand with assumptions, example, trade-off, failure mode and evidence. These are study prompts, not memorized scripts.

## A. Python and service engineering

### 1. What is the difference between a Python list and a NumPy array?
**Short answer:** a list is a general object sequence; a NumPy array is a typed multidimensional structure with vectorized numerical operations and shape semantics.  
**Deeper answer:** Python loops over object references, while NumPy operations can execute optimized kernels over contiguous data. Dtypes, broadcasting and views introduce their own correctness concerns. Use an array for numerical workloads; use a list for general heterogeneous objects.  
**Follow-up:** How can a NumPy slice unexpectedly change the original array? Explain view versus copy.

### 2. How do you keep a blocking model call from freezing an async API?
Identify whether the runtime is async and whether inference is CPU/GPU blocking. Use a dedicated model-serving runtime, bounded worker executor or separate service; bound queue/concurrency and propagate cancellation/deadline. Do not create unlimited threads or assume an async keyword makes native inference nonblocking.

### 3. How do you design an API when the model output is nondeterministic?
Return a stable product schema with model/pipeline version, status, evidence and trace ID. Validate output. Define idempotency, timeout, streaming and retry behavior; treat model behavior as versioned and evaluate changes before rollout.

## B. ML and evaluation

### 4. Explain data leakage.
Leakage occurs when unavailable-at-prediction-time information influences fitting, feature construction or model selection. Examples include future fields, preprocessing before splitting, duplicate users across folds, and repeatedly tuning against a test set. Use deployment-realistic splits and pipelines fitted within training folds.

### 5. Why can accuracy mislead?
For rare positive cases, always predicting the majority can achieve high accuracy while missing every positive. Show confusion matrix, precision/recall, threshold trade-offs, uncertainty and business costs.

### 6. How would you select a threshold?
Use a validation set with representative prevalence; define the relative cost or operational constraint, such as minimum recall or review capacity; inspect the precision-recall curve and subgroup errors; freeze the selected threshold before final test. Revisit after distribution shift.

### 7. What is calibration?
Among examples receiving probability near p, roughly fraction p should be positive under the evaluated population. Ranking quality and calibration are different. Evaluate reliability and calibration if probability is used for costs or decisions.

### 8. When do you use group or time splits?
Use group splits when the deployment target is a new person/organization/device and repeated entities otherwise leak. Use temporal splits when predicting future cases. Split logic must reflect the intended generalization claim.

## C. Deep learning and Transformers

### 9. What is backpropagation?
Efficient application of the chain rule through a computational graph to calculate loss derivatives with respect to parameters. An optimizer consumes those gradients to update parameters; backpropagation itself is not the update rule.

### 10. Why does a Transformer need a causal mask for language generation?
The objective predicts future tokens from prefixes. A causal mask prevents each position from observing later target tokens during training; inference also only has the prefix.

### 11. Explain Q, K and V.
Learned projections create query vectors that represent matching needs, key vectors that represent matchable features, and value vectors carrying information. Scaled query-key dot products become softmax weights over values. It is a mathematical mixing mechanism, not proof of human-like attention.

### 12. Why does long context affect latency and memory?
Prefill must process the prompt; dense self-attention has pairwise position interactions. During autoregressive generation, KV cache grows with layers, sequence length and concurrency. Exact cost depends on architecture and runtime.

## D. RAG and retrieval

### 13. When is RAG useful?
When answers need fresh/private/attributable knowledge held outside model weights, and query-time evidence can be retrieved under correct permissions. It is not always right: deterministic lookup, classification or small stable content may be simpler.

### 14. How do you evaluate RAG?
Separate retrieval quality from answer quality. Use labeled queries and source IDs; measure recall/ranking at k, citation validity/support, correctness, groundedness, abstention, latency and cost. Review realistic no-answer and access-control cases.

### 15. What happens when a document is deleted?
Remove it from source storage as required, lexical index, vector index, caches and derived artifacts; retain only permitted audit metadata. Use versioning, deletion jobs, retry/verification, and a test that proves it no longer appears. The exact legal retention policy is organization-specific.

### 16. How do you defend against a malicious retrieved document?
Treat retrieved content as untrusted data. Do not let it grant tools or authority. Keep permissions and tool allowlists in code, limit outputs, validate proposals, isolate external network access, test indirect injection and audit retrieval provenance.

## E. Agents and tools

### 17. What makes an agent different from a normal LLM call?
A model participates in a control loop that can select actions, receive tool results and continue. More autonomy adds risk and complexity; deterministic workflows are often preferable.

### 18. How do you safely expose a write tool?
Narrow schema, server-derived identity, authorization on every call, bounded arguments, idempotency, preview/approval when impact warrants, audit, rollback and strict stop budgets. The model only proposes; trusted code executes.

### 19. What does MCP solve and not solve?
It standardizes a way for AI applications to connect to tools/resources/prompts. It does not automatically solve identity, authorization, consent, trust, validation or safe execution.

## F. Backend and distributed systems

### 20. How do you avoid duplicate model actions after a timeout?
Distinguish timeout from confirmed failure. Use idempotency keys and persisted job state; reconcile with the upstream provider if possible. Bound retries and avoid repeating user-visible side effects without knowing whether the first action completed.

### 21. Explain bulkhead, circuit breaker and backpressure.
Bulkhead limits resource consumption by one dependency; circuit breaker reduces calls to a failing dependency; backpressure bounds work or slows producers under load. They must be configured with queue/concurrency limits and meaningful metrics.

### 22. When would you split a Python model worker from a Java service?
When model-specific libraries/runtime, accelerator scaling, release lifecycle, resource isolation or team ownership justifies the network/deployment boundary. Account for latency, contracts, auth, failure handling and operations. Do not split merely because different languages exist.

### 23. How do you design for provider outage?
Use deadlines, bounded retries, circuit breaker and clear degraded behavior. A fallback must be evaluated and policy-compatible. Surface technical failure without inventing an answer; alert and support rollback.

## G. Security and governance

### 24. Is a system prompt a security control?
It is a behavior instruction, not an authorization boundary. Enforce access and action limits in trusted code. Keep secrets out of prompts and validate tool proposals and outputs.

### 25. How do you prevent cross-tenant retrieval?
Derive tenant identity from authenticated server context, filter before content reaches the model, scope caches/index queries, recheck permissions on citations, and test adversarial cross-tenant access.

### 26. What should model telemetry include?
Trace ID, model/version, timing phases, token counts, status, retry/finish metadata and quality/evaluation signals. Avoid raw personal prompts by default; minimize and control access/retention if sampled content is necessary.

## H. System design prompts

For each, clarify scale, latency, privacy, consistency, quality and failure requirements before drawing components:
1. Design an internal policy assistant with citations and revocable document access.
2. Design an AI ticket router that must hit a minimum urgent-ticket recall.
3. Design a batch document summarization platform with retry and cost quotas.
4. Design a model-provider abstraction supporting controlled model migration.
5. Design an online evaluation/feedback loop without using user corrections as unreviewed training labels.

Answer structure:
1. requirements and assumptions;
2. API/data model;
3. high-level architecture;
4. critical sequence and storage;
5. capacity estimate with explicit assumptions;
6. failure/security/abuse cases;
7. observability and SLO;
8. trade-offs and staged rollout.

## Behavioral evidence prompts

Prepare truthful STAR stories (Situation, Task, Action, Result) for:
- a difficult production incident;
- an architectural decision you revisited;
- a quality/security issue you found;
- a disagreement resolved with evidence;
- a project where you reduced latency/cost or improved reliability;
- a mistake and concrete learning.

Do not invent metrics. If you lack AI production experience, clearly distinguish shipped Java/backend evidence from personal AI project evidence.

## Self-scoring rubric

For each answer score 0–2:
- 0: incorrect, vague or unsafe;
- 1: correct core idea but lacks assumptions/trade-offs;
- 2: correct, scoped, example-driven, includes failure mode/evaluation.

A score of 2 requires explaining what would change the design and how you would verify it.