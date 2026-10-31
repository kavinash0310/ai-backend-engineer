# Learning roadmap — November 2026 to December 2027

**Audience:** Java/Spring Boot engineers who are new to AI/ML, or other programmers who want a structured route from foundations to production AI systems.

The target is credible capability for AI Backend Engineer, GenAI Engineer or AI Software Engineer roles. It does not promise seniority or a specific compensation. Senior responsibility normally requires real production ownership and engineering judgment beyond a reading plan.

| Month | Focus | Exit evidence |
|---|---|---|
| Nov 2026 | Python environment, syntax, functions, data structures, files, exceptions, Git | Tested CLI data-cleaning script; explain every line |
| Dec 2026 | NumPy/Pandas, visualization, vectors/matrices, descriptive statistics | Reproducible exploration and hand-checked numerical examples |
| Jan 2027 | Probability, derivative/gradient intuition, optimization, ML problem framing | Derive a small gradient and create a clean leakage-aware dataset |
| Feb 2027 | Regression/classification, trees, validation, metrics, imbalance, tuning | Reproducible scikit-learn report with untouched test set |
| Mar 2027 | Neural networks, losses, backprop, PyTorch training loop | Small neural model, debug log, comparison with baseline |
| Apr 2027 | CNN/RNN context, attention, transformer blocks, tokenization | Tiny self-attention implementation with tensor-shape tests |
| May 2027 | LLM training stages, decoding, context, KV cache, quantization, serving | LLM lifecycle explanation and memory/cost worksheet |
| Jun 2027 | Embeddings, parsing/chunking, vector and lexical search, RAG evaluation | Versioned RAG baseline and retrieval/answer metrics |
| Jul 2027 | RAG security/production, Spring AI, streaming and API integration | Spring feature with tenant checks, timeouts, citations, tests |
| Aug 2027 | Python inference API, Java/Python communication, jobs, modular monoliths | Working boundary and architecture decision record |
| Sep 2027 | Tools, bounded agents, LangGraph concepts, MCP, approvals/evaluation | Audited workflow with allowlist, durable state, negative tests |
| Oct 2027 | Distributed systems, capacity, cloud, Docker/Kubernetes, observability | Design review, staging deployment, dashboard and runbook |
| Nov 2027 | Capstone integration, evaluation, load/security hardening | Reviewable integrated candidate with measured evidence |
| Dec 2027 | Portfolio, interview revision, mock interviews, job transition | Project evidence, role-targeted resume and application plan |

## Workload and priority

24–30 hours weekly gives roughly 1,450–1,830 hours over the 14-month window. Allocate approximately 8–10 hours to learning/derivation, 10–13 to coding/debugging, 3–4 to exercises and recall, and 3–4 to integration/review.

- **Must master:** Python for data/model collaboration; practical math; ML evaluation/leakage; neural-net and transformer mechanics; API/security boundaries; RAG evaluation; reliability, testing and cost/latency reasoning.
- **Strong working knowledge:** PyTorch, embeddings, hybrid retrieval/reranking, agents/MCP, async work, cloud/containers, observability and system design.
- **Awareness:** industrial-scale pretraining, large distributed GPU training, model families outside project scope, regulations outside target market.
- **Optional specialization:** CUDA/kernel tuning, RL research, multimodal model training, advanced causal inference, on-device quantization. Select based on actual job postings.

## Daily rhythm

Use four focused blocks: 75–90 min study, 90 min coding, 60–75 min exercise/project work, 30–45 min recall/logging, with breaks between. On study day six, revise, integrate, clean code, and assess; rest on day seven.

If a prerequisite gate fails, repair it before building on it. If work/life reduces available hours, extend the timeline rather than skipping exercises. Use CPU-size examples and controlled hosted notebooks/APIs; do not depend on training large models on a 16 GB Windows machine. Current framework versions and compatibility were checked on 2026-10-31; recheck before installing.
