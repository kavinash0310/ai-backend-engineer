# MLOps: versioning, deployment and model monitoring

## What MLOps means

MLOps applies software, data and operations practices to machine-learning systems. It connects experiments to reliable releases and feedback. It is not merely a model registry or a CI tool. A production ML system includes data collection, labels, feature code, model artifacts, service code, policy, infrastructure and monitoring.

## 1. Reproducible experiment and artifact lineage

For every candidate model, record:
- source-data snapshot/query and access basis;
- label definition and split strategy;
- preprocessing and feature version;
- code commit and dependency/runtime versions;
- model family, parameters and training configuration;
- random seed where relevant;
- evaluation set, metrics, uncertainty and error slices;
- artifact checksum and model/schema version;
- intended use, exclusions and limitations.

A model file alone is insufficient. The same parameters with different preprocessing can behave like a different system.

## 2. Training-serving consistency

Training and inference must agree on feature definitions, units, missing-value handling, categorical mappings, time zones, tokenization and normalization. **Training-serving skew** can make production input differ from what the model learned.

Controls include:
- shared versioned transformation code where practical;
- explicit feature contracts;
- online/offline comparison on sampled records;
- feature freshness and missingness monitoring;
- schema compatibility checks before rollout;
- shadow predictions against real traffic without changing user outcomes.

For a RAG system, analogues include ingestion/chunking version, embedding model, query preprocessing, ACL semantics and prompt construction.

## 3. Model registry and promotion

A registry tracks artifacts and metadata across stages such as experiment, candidate, staging and production. Promotion should be controlled by evidence, not by a filename or latest timestamp.

A release record should bundle application code, model and tokenizer, preprocessing, prompt, retrieval/index schema, policy and evaluation report. Immutable identifiers make it possible to answer which system handled a request and to roll back consistently.

## 4. Deployment strategies

- **Offline batch scoring**: precompute predictions; useful when fresh response is not required.
- **Online synchronous inference**: return prediction in the request path; needs strict latency and availability budgets.
- **Asynchronous job**: return job ID and process later; useful for long document/model tasks.
- **Shadow**: candidate receives traffic but does not affect user decisions; compare behavior safely.
- **Canary**: route a small controlled fraction to a candidate, observe quality and service impact, then expand.
- **Blue/green**: keep old and new deployment environments and switch traffic with rollback capability.

A canary must respect privacy and evaluation policy. A new model can alter quality, cost and harmful behavior even if HTTP success remains unchanged.

## 5. Monitoring has multiple layers

### Data/input monitoring
Schema changes, missingness, ranges, category rates, language, input length and feature freshness.

### Prediction monitoring
Score distributions, class/abstention rates, output schema validity, retrieval scores, token use and tool proposals.

### Service monitoring
Availability, errors, latency percentiles, queue age, CPU/RAM/GPU, provider throttling, retry amplification and cost.

### Outcome/quality monitoring
Correctness, precision/recall, calibration, user correction, human escalation, citation support and subgroup errors once labels become available.

A shift alarm is not itself proof of failure. Investigate source, instrumentation, seasonality and consequences.

## 6. Delayed labels and feedback loops

Ground truth may arrive days or months after prediction. Do not judge model quality solely by immediate user clicks or acceptance; those are biased proxies. Users may accept wrong answers, stop using the feature, or only correct certain error types.

A feedback loop occurs when predictions change which cases are reviewed or collected. For example, a classifier sends only high-risk cases to human review, so labels become available mainly for high-risk cases. This creates selection bias. Record sampling/decision policy and use suitable evaluation designs.

## 7. Drift and retraining policy

Monitor covariate, label, concept and data-quality drift. Decide in advance:
- what signal triggers investigation;
- who reviews it;
- how to obtain trusted labels;
- when retraining is allowed;
- what validation gates apply;
- how to rollback;
- how to retire a model.

Do not automatically retrain on all incoming user interactions. Data may be malicious, unrepresentative, duplicated, private or incorrectly labeled.

## 8. Model cards and data cards

A model card summarizes intended use, prohibited use, evaluation, populations, known limits, risk mitigations and version. A data card describes collection, population, labels, transformations, privacy, quality and limitations. These documents help users make informed decisions; they do not certify safety.

## 9. MLOps and LLM applications

LLM systems add versioned prompts, tool schemas, retrieval indexes, embeddings, rerankers and evaluation rubrics. Log the versions needed for reproducibility. Do not log sensitive full prompts by default.

An LLM release gate should include:
- deterministic schema/security checks;
- fixed task evaluation set;
- adversarial and no-answer cases;
- human review for semantic correctness;
- latency/cost profile;
- rollback or kill switch.

## Questions and answers

**Q: What is the difference between data drift and concept drift?**  
A: Data/covariate drift changes the input distribution. Concept drift changes how inputs relate to outcomes. Either may or may not reduce quality; measure outcomes where possible.

**Q: Should retraining happen whenever a feature distribution changes?**  
A: No. A distribution change triggers investigation. It may be expected seasonality, instrumentation change or meaningful population change.

**Q: What is shadow deployment useful for?**  
A: It compares a candidate on real-like traffic without affecting user decisions. Protect data and ensure candidate outputs are not exposed unintentionally.

**Q: Why version prompts and indexes?**  
A: They change system behavior just as model weights do. Reproducing or rolling back an answer requires all relevant versions.

## Practice

1. Define a release manifest for an ML model and an LLM/RAG pipeline.
2. Design a shadow experiment that avoids sending disallowed data to a candidate provider.
3. Create monitoring for schema break, quality degradation, provider throttling and delayed label shift.
4. Explain how selective human review can bias measured performance.
5. Write model-card sections for a support-ticket priority classifier.

## Completion check

You can trace model/data lineage, deploy with controlled rollout, monitor input/service/outcome behavior, diagnose delayed feedback and define a safe retraining/rollback policy.