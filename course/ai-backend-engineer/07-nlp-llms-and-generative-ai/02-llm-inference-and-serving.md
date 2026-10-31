# LLM inference and serving fundamentals

## 1. What inference does

Inference runs a trained model to produce outputs. For a text-generation model there are two broad phases:
- **Prefill**: process the prompt and compute internal states for its tokens.
- **Decode**: generate tokens sequentially, each new token conditioned on prior prompt and generated tokens.

Prefill can parallelize work across prompt positions within an implementation; autoregressive decode depends on previously generated tokens. This distinction affects latency: a long prompt can make time-to-first-token slow, while a long answer increases time spent decoding.

Useful latency terms:
- **Time to first token (TTFT)**: time until the first generated token.
- **Inter-token latency**: spacing between generated tokens.
- **End-to-end latency**: full request duration.
- **Throughput**: requests or tokens processed per unit time.
- **Concurrency**: simultaneous work in flight.
- **Queue time**: waiting before processing begins.

A single average hides tail behavior. Track percentiles and workload shape.

## 2. Parameters, precision and rough memory

A parameter is a learned numeric value. A crude lower-bound estimate for weight memory is:
parameter_count × bytes_per_parameter.

For example, 7 billion parameters at 2 bytes each is about 14 billion bytes (about 13 GiB depending on units). This omits runtime overhead, temporary buffers, activations, cache, allocator fragmentation and framework needs. It is not a hardware fit guarantee.

Quantization represents weights or activations with fewer bits, often reducing memory and sometimes improving throughput. It can change quality, numerical stability and hardware behavior. Evaluate the exact quantized artifact on the actual target workload. A label such as “4-bit” alone does not specify all storage, scaling or runtime details.

## 3. KV cache

In causal generation, prior tokens' keys and values can be stored so the model need not recompute every earlier projection at each next-token step. This stored state is called the KV cache.

A rough per-request cache estimate for a conventional Transformer:
2 × layers × sequence_length × number_of_KV_heads × head_dimension × bytes_per_value.

The factor 2 is for keys and values. Actual implementations may use grouped-query attention, cache quantization, sliding windows, paged allocation, alignment and metadata, so use this as an intuition/check rather than a precise capacity quote.

The cache grows with context length and active concurrent sequences. Thus a model can fit in device memory for a single short prompt but fail at a larger context or concurrency. Admission control should consider both model weights and per-request cache.

## 4. Batching and scheduling

Batching combines requests to improve accelerator utilization. Continuous/dynamic batching can admit new sequences while others generate. Bigger batches can increase throughput but also waiting time, memory pressure and interference among latency-sensitive requests.

A serving scheduler may use:
- max queued requests;
- per-tenant concurrency;
- request deadlines;
- max input/output token budgets;
- priority classes;
- cancellation handling;
- fairness policy;
- overload rejection or graceful fallback.

A queue without bounds converts overload into unbounded latency and memory use. It is often better to reject quickly with a retry hint than to accept work that cannot meet its deadline.

## 5. Cost and capacity model

For hosted APIs, cost may depend on input/output tokens, model choice, caching, region or service tier. For self-hosting, cost includes accelerators, idle capacity, engineering, storage, bandwidth, redundancy and operations. Pricing changes: consult the provider's current official price sheet when making an estimate.

A first estimation worksheet:
1. expected requests per day and peak requests per second;
2. input and output token distributions, not just averages;
3. TTFT and completion-latency targets;
4. concurrency and burst profile;
5. model and serving runtime;
6. cache/reuse opportunity;
7. measured throughput under realistic prompt lengths;
8. failure, retry and fallback rates;
9. cost per successful task, not merely cost per call.

If output length doubles, generation work and token-related cost can rise materially. Token budgets are both product controls and capacity controls.

## 6. API boundary and request contract

A backend should expose a stable product contract rather than leaking provider-specific request formats. Typical fields:
- authenticated principal and tenant;
- conversation/task identifier;
- user message and bounded context;
- requested operation and allowed tool scope;
- response format or schema;
- deadline, token budgets and idempotency key;
- safety and data-handling policy.

Return a clear response with answer, citations or evidence where relevant, finish reason, model/pipeline version, usage metadata and trace ID. Do not expose hidden reasoning or provider internals as though they were verified facts.

Validate size and content before invoking expensive work. Treat user text as untrusted data. Keep secrets in a secret manager, never in prompts or logs.

## 7. Retries, timeouts and streaming

Set separate connection, response and overall deadlines. Retry only transient failures, with bounded exponential backoff and jitter. Retrying a generation request can duplicate cost or user-visible output; use idempotency where the provider and operation support it. Do not retry permanent validation, policy or authentication failures.

Streaming can improve perceived latency by returning partial output. It complicates cancellation, moderation, schema validation, retries and connection management. Define what happens if the client disconnects, the upstream stream fails halfway, or a partial answer violates output requirements.

A timeout must be budgeted across queueing, retrieval, model call and post-processing. If each hop gets the full end-to-end timeout, the request can exceed the user's deadline.

## 8. Reliability and fallback

Model providers and GPU workers can fail, throttle, degrade or change behavior. Design:
- circuit breakers and concurrency limits;
- bounded queues and overload response;
- health/readiness checks;
- fallback model or deterministic workflow where appropriate;
- graceful degradation when retrieval or a tool is unavailable;
- version pinning and controlled rollout;
- rollback path;
- durable audit of configuration changes.

A fallback is only good if its behavior is understood and safe. Falling back to a smaller model can change quality and policy behavior; indicate that in internal telemetry and evaluate it.

## 9. Observability without leaking content

Record low-risk operational fields: trace ID, model/deployment version, prompt token count, output token count, latency phases, retry count, status, finish reason and safety outcome. Avoid recording raw prompts/completions by default. If debugging samples are necessary, minimize, redact, restrict access, set retention, and follow policy/consent requirements.

Separate three questions:
- Did the service succeed technically?
- Did the model produce a useful/correct result?
- Did the product decision safely use that result?

A 200 response is not proof of a correct answer.

## Questions and answers

**Q: Why is KV cache important?**  
A: It reuses key/value states from previous tokens during autoregressive decoding. It avoids recomputing some work but consumes memory that grows with sequence length and concurrent requests.

**Q: Does quantization make every model 4x smaller?**  
A: Not necessarily. Metadata, scales, unquantized layers, runtime buffers and storage formats add overhead. Throughput and quality also depend on hardware and kernels.

**Q: Why is a long prompt expensive if the model output is short?**  
A: The prefill phase must process the prompt, and attention operations depend on sequence length. Input token cost and time can be large even when generation is brief.

**Q: Should I retry any model error?**  
A: No. Retry only transient failures within a bounded deadline and retry budget. Never blindly repeat invalid, unauthorized or policy-rejected requests.

**Q: Is streaming always faster?**  
A: It may improve time to first visible content but does not make total computation disappear. It adds complexity around partial outputs, cancellation and validation.

## Practice

1. Calculate the rough weight memory of 3B parameters at 16 bits; state what your estimate omits.
2. Compare KV cache for 2× context length and 4× concurrent requests.
3. Design an API response that includes citation metadata and usage without exposing secret prompts.
4. Draw a timeout budget across queue → retrieval → model → post-processing.
5. Propose overload behavior when the queue reaches its bound.

## Completion check

You can explain prefill versus decode, estimate rough memory without overclaiming, reason about KV-cache scaling, design bounded request handling, and name the telemetry required to operate inference responsibly.