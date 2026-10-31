# Evaluation and MLOps

Evaluate both software behavior and model behavior, then make model/data changes reproducible and safe to release.

## Chapters

- [MLOps and model lifecycle](mlops-and-model-lifecycle.md): lineage, training/serving consistency, rollout, monitoring, drift, and retraining.
- [AI backend testing](testing/README.md): unit, integration, contract, resilience, regression, and quality tests.
- [LLM application evaluation](../07-nlp-llms-and-generative-ai/06-llm-application-evaluation.md): datasets, metrics, human review, retrieval and answer quality, and release gates.

For model-backed services, combine deterministic software tests with versioned evaluation cases. A passing API test does not prove answer quality, and a quality score does not prove authorization correctness.
