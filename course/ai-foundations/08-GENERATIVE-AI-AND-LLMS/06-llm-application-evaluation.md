# Evaluating LLM applications

## Why evaluation is hard

LLM outputs are open-ended, stochastic and context-sensitive. A single exact target string is often too narrow; an automatic similarity score can reward plausible wording while missing factual errors. There is no one universal “LLM accuracy” metric.

Evaluation should answer a product question: does this version safely complete the intended task for the relevant users, under realistic cost and latency constraints?

## 1. Define task-specific quality dimensions

For a knowledge assistant, assess:
- answer correctness;
- grounding in supplied evidence;
- citation validity and support;
- completeness and relevance;
- abstention when evidence is missing;
- adherence to output schema and policy;
- robustness to paraphrase, language, long inputs and adversarial content;
- latency, availability and cost.

For extraction, assess field-level exactness, missing-field behavior and schema validity. For classification, use confusion matrices and threshold metrics. For agents, measure task completion, action correctness, authorization failures, steps and recovery.

## 2. Build a versioned evaluation set

Each case should include:
- input and relevant context;
- user/tenant policy where applicable;
- expected result or rubric;
- acceptable evidence/source IDs;
- no-answer/abstention label;
- language/domain/difficulty tags;
- expected behavior for adversarial or boundary cases.

Use synthetic or public data to start. For real data, follow privacy/access rules and minimize sensitive fields. Include typical, edge, negative, ambiguous and known-failure examples. Track annotator disagreement instead of pretending every reference is objective.

Separate:
- development set for iteration;
- validation set for design/threshold choices;
- final holdout for a limited launch estimate;
- online monitored sample or prospective set for changing populations.

If developers repeatedly view holdout outputs and change the system, that holdout has become part of development.

## 3. Deterministic and semantic checks

Deterministic checks include:
- response parses against schema;
- cited IDs exist and were retrieved;
- no unauthorized source is returned;
- required fields/limits are present;
- output does not contain a forbidden operation;
- tool arguments meet server rules.

Semantic checks include correctness, support, usefulness, tone and completeness. Use expert review or a carefully calibrated rubric. Deterministic checks are precise but narrow; semantic review covers meaning but can be subjective.

## 4. Human evaluation

A rubric should define dimensions and examples of each score. Blind reviewers to model version where possible. Randomize answer order in pairwise comparisons. Give reviewers source evidence when judging groundedness. Measure inter-annotator agreement and resolve disagreements for high-impact categories.

Review a sample of both successes and failures; selection only from thumbs-down reports is biased. Protect reviewer privacy and avoid unnecessary exposure to sensitive inputs.

## 5. LLM-as-judge

A second model can score or compare outputs. Risks:
- preference for verbose or stylistically similar answers;
- position/order bias in pairwise comparison;
- shared blind spots with the evaluated model;
- hallucinated justification;
- prompt sensitivity and nondeterminism.

Mitigations:
- give a precise rubric and source evidence;
- randomize order and repeat where needed;
- test judge results against expert-labeled examples;
- report agreement and uncertainty;
- use judges for triage/regression detection, not unquestioned ground truth.

## 6. Benchmarking and contamination

Public benchmarks can help compare broad capability but may be present in pretraining data, optimized against, unrepresentative of the deployment domain, or poorly aligned to user utility. Record benchmark version, prompt, model version, decoding, sample, judge and date. Pair them with fresh task-specific tests.

A higher aggregate score can hide regressions on an important language, subgroup or no-answer case. Report disaggregated results where lawful and statistically meaningful.

## 7. Statistical uncertainty

A measured score on finite examples has uncertainty. For proportions, report counts and intervals; for model comparisons, use paired evaluation on the same examples where appropriate. Repeated prompt/model experimentation increases selection bias. Avoid claiming a tiny difference is meaningful without considering uncertainty, cost and practical impact.

For rare harms, zero observed failures does not prove the true risk is zero. Estimate how much data would be needed to detect the failure rate of concern and use adversarial testing plus safeguards.

## 8. Regression and release gates

For every change to model, prompt, retrieval, tokenizer, index, tools or policy:
1. run deterministic tests;
2. run fixed task evaluation;
3. inspect important slices and known failures;
4. measure latency/cost under representative load;
5. review security/adversarial cases;
6. compare against current production version;
7. canary or shadow where appropriate;
8. record release decision and rollback trigger.

A model-provider change can be a behavior change even if the API schema remains identical.

## 9. Online feedback and quality signals

User ratings and corrections are useful signals but are not unbiased labels. Acceptance may mean convenience, not correctness; dissatisfied users may leave without feedback. Combine feedback with sampled expert review, downstream outcomes and operational signals. Do not automatically fine-tune on every user correction without validation, privacy review and poisoning controls.

## 10. Evaluation report template

Document:
- system/model/prompt/index/tool versions;
- task, population and intended use;
- dataset size, period, sampling, exclusions and privacy basis;
- metrics, confidence intervals and subgroup slices;
- judge/human rubric and agreement;
- examples of strengths and failures;
- latency, cost and load profile;
- unresolved risks and launch limits;
- approval owner, monitoring and rollback plan.

## Questions and answers

**Q: Can BLEU, ROUGE or semantic similarity prove an answer is correct?**  
A: No. They compare text under a chosen metric and can disagree with correctness or usefulness. Use task-specific human/grounded checks.

**Q: Are thumbs-up/down enough for evaluation?**  
A: No. Feedback is sparse and selected. It should complement representative, labeled evaluation and outcome review.

**Q: Is an LLM judge useless?**  
A: No. It can scale triage and pairwise comparisons when calibrated and audited. Treat it as a noisy measurement instrument.

**Q: How do I compare stochastic model outputs?**  
A: Fix model/settings and use repeated runs where variability matters; compare paired examples, distributions, error slices and cost. Set deterministic decoding only if it matches product behavior.

## Practice

1. Build 30 test cases for a small RAG app, including no-answer, stale-source and prompt-injection examples.
2. Create a 4-point groundedness rubric and test reviewer agreement.
3. Compare LLM-judge decisions with human ratings on 20 examples and analyze disagreements.
4. Write a release gate for changing the embedding model or system prompt.
5. Estimate what “0 failures in 100 cases” does and does not establish.

## Completion check

You can create a versioned evaluation set, distinguish deterministic checks from semantic judgments, calibrate evaluators, report uncertainty and gate releases on quality, safety, latency and cost.