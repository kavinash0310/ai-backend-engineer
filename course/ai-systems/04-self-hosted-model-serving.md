# Self-hosted model serving

> **Place in the course:** Optional specialization after provider integration and Docker. Most beginners should first build a correct service using a hosted model API. Self-hosting becomes relevant when control, privacy, predictable volume, latency, or customization justifies its operational cost.

## Hosted API or self-hosted model?

A hosted API transfers hardware and model-runtime operations to a provider, but adds network dependency, per-use cost, provider-specific behavior, and data-governance review. Self-hosting gives more control over model version, hardware, and data path, but your team owns GPU availability, model loading, batching, scaling, security patches, and capacity planning.

Compare total cost, not only token price. Include GPU/CPU instances, idle time, storage, engineering on-call, network, retries, monitoring, and quality. A small or variable workload may cost less through a hosted API; a steady high-volume workload may make dedicated serving worth evaluating. Benchmark with your actual inputs.

## Serving architecture

A model-serving process loads weights onto CPU or GPU memory and exposes an inference API. The backend calls it over an internal network, applies auth and quotas, validates outputs, and records request metadata. Keep model-server credentials and network access internal. Do not expose a raw model endpoint publicly without authentication, resource limits, and abuse controls.

Model startup can take minutes. Readiness should remain false until weights are loaded and a small health inference succeeds. Liveness should indicate whether the process is irrecoverably stuck, not whether one model call is slow. On deployment, load a new model version alongside the old where resources allow, run smoke/evaluation checks, shift traffic gradually, and retain rollback capacity.

## GPU memory and concurrency

Memory use includes model weights, activations or KV cache, request batch, and runtime overhead. For a decoder model, longer contexts and more concurrent sequences increase KV-cache memory. A model that loads successfully may still run out of memory under real concurrency.

Batching improves hardware utilization by combining compatible requests. Dynamic batching waits briefly to form a batch; this can improve throughput while adding latency. Continuous batching admits new sequences as others finish. Tune batch size, sequence length, and concurrency together, and measure p50/p95 latency and tokens per second.

Quantization lowers weight precision and may reduce memory; quality can change. Tensor parallelism splits computation across GPUs but adds communication and operational complexity. Use a model server that supports the required runtime and features, then test actual hardware.

## API and operational safeguards

Set maximum input/output tokens, request timeout, concurrency, queue length, and per-user quota. Cancel work when clients disconnect if possible. Keep health, metrics, and logs. Track model load time, queue delay, generation latency, error and out-of-memory rates, utilization, tokens per second, and cost per successful request.

Model artifacts are software supply-chain inputs. Pin a model revision, verify source and license, restrict download locations, scan artifacts where possible, and keep weights away from public writable paths. Avoid loading untrusted serialized code.

Questions: Does self-hosting automatically keep data private? No; logs, networking, storage, and access still need controls. Does quantization always improve speed? No; hardware/runtime support matters. Why separate readiness from liveness? A model still loading should not receive traffic but also may not need a restart.

Practice: benchmark hosted and self-hosted options on the same evaluation set and prompts. Record quality, p95 latency, throughput, peak memory, uptime assumptions, and monthly cost. Define the traffic level at which self-hosting becomes worth reconsidering.

Further reading: [NVIDIA performance guide](https://docs.nvidia.com/deeplearning/performance/dl-performance-getting-started/index.html) and [Docker overview](https://docs.docker.com/get-started/docker-overview).
