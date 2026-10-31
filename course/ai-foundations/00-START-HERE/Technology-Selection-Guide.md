# Technology selection guide

Learn the capability before learning its framework. Choose a tool only after naming the requirement, operational burden, simpler alternative, and evidence that would justify it.

| Need | Start with | Add later when justified |
|---|---|---|
| Tabular ML | scikit-learn | XGBoost/LightGBM or distributed training |
| Neural networks | PyTorch | Distributed training/compilation after profiling |
| Spring model integration | Spring AI or a narrow HTTP adapter | Multi-provider gateway when routing needs exist |
| Python service | FastAPI | gRPC or specialized serving for concrete requirements |
| RAG storage | PostgreSQL full-text search; add pgvector for vectors | Dedicated vector DB after corpus-specific benchmarking |
| Workflow | Explicit state machine | LangGraph/durable workflow engine after state complexity warrants it |
| Agents | Narrow allowlisted tools | Multi-agent orchestration only when evaluations justify it |
| Queue | Managed queue for durable jobs | Kafka when stream/replay/throughput requirements justify it |
| Deploy | Docker and one cloud platform | Kubernetes when orchestration complexity exists |

Spring AI provides Spring-oriented AI model and vector integrations; LangChain is a broad LLM application integration/composition ecosystem; LlamaIndex focuses on data connectors/indexing/retrieval abstractions; LangGraph models stateful graph workflows/agents. None supplies your access-control policy, quality evaluation, or business correctness. Learn underlying HTTP, state, retrieval and authorization first.

With 16 GB RAM on Windows, begin on CPU with small data. For bigger experiments use hosted notebooks only after checking terms/privacy and use controlled hosted model APIs with strict budgets. Do not depend on local frontier-model training.

**Version note checked 2026-10-31:** current Spring AI docs state Spring AI 2.0.x supports Spring Boot 4.0.x and 4.1.x and recommend its BOM. If your existing service uses another Boot line, follow its compatible dependency line rather than upgrading for this handbook.
