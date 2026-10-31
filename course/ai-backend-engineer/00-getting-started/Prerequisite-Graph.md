# Prerequisite graph

~~~mermaid
flowchart TD
  CS[CS: data structures, OS, networks, concurrency] --> PY[Python programming]
  PY --> DATA[NumPy, Pandas and data quality]
  MATH[Math: vectors, probability, calculus, statistics] --> ML[ML lifecycle and evaluation]
  DATA --> ML
  ML --> DL[Neural nets and optimization]
  MATH --> DL
  DL --> TR[Attention and transformers]
  TR --> LLM[LLM lifecycle and inference]
  LLM --> RAG[Embeddings and RAG]
  J[Existing Java/Spring] --> API[Secure service boundaries]
  API --> SAI[Spring AI and provider integration]
  LLM --> SAI
  RAG --> SAFE[Retrieval security/evaluation]
  API --> SAFE
  SAFE --> AG[Bounded agents and tools]
  CS --> DS[Distributed systems and capacity]
  API --> DS
  SAI --> OPS[Cloud and operations]
  DS --> OPS
  SAFE --> OPS
  AG --> CAP[Capstone and interview evidence]
  OPS --> CAP
~~~

Do not tune models before defining target, split and metric; do not learn backprop before chain-rule intuition; do not ship RAG before authorization and evaluation; do not add autonomy before you can write the workflow deterministically.
