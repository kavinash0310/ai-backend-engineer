# LLM training and post-training: from pretraining to adaptation

## Purpose and limits

Most AI backend engineers will use pretrained models rather than train a foundation model from scratch. Still, you should understand what the training stages do, what fine-tuning can change, what it cannot reliably add, and why training data/evaluation matter.

## 1. Pretraining

A decoder language model commonly learns next-token prediction on large text corpora. For each position, it predicts a token from prior context and minimizes negative log likelihood. This objective teaches language patterns, syntax and broad associations, but the training corpus is not a verified knowledge base.

Data pipelines deduplicate, filter, normalize and mix sources. Choices affect language coverage, code/domain representation, privacy, copyright/licensing and toxicity. A model's behavior reflects architecture, data, objective, optimization, scale and later training stages—not one magic parameter count.

Training from scratch requires a very large, well-governed corpus, accelerator infrastructure, distributed optimization, data-loading throughput, checkpoint/recovery and extensive evaluation. A 16 GB consumer machine is suitable for learning toy models and small experiments, not training a competitive foundation model.

## 2. Supervised fine-tuning

Supervised fine-tuning (SFT) continues training a pretrained model on example conversations or instruction/answer pairs. It can teach response format, task patterns, domain terminology and desired interaction style.

Data quality requirements:
- examples reflect intended use and allowed data;
- labels are accurate and consistent;
- conversations include proper role/end markers;
- train/validation/test examples are separated by source/user/time where needed;
- sensitive or copyrighted data are handled under policy;
- format and tokenizer match the base model.

SFT does not guarantee a model knows a changing fact, can cite a private document or obey all policies. RAG or a deterministic data source is often better for mutable facts. Fine-tuning changes parameters; retrieval supplies external evidence at request time.

## 3. Preference training and alignment

A preference dataset contains alternatives with a preference judgment or ranking. Common broad approaches:
- **RLHF**: supervised examples plus a learned reward model and reinforcement-learning updates, often with a constraint to avoid drifting too far from a reference.
- **DPO-style methods**: optimize a preference objective directly against chosen/rejected responses and a reference model, avoiding a separately sampled online RL loop in common forms.
- Other preference/feedback methods vary in objective and data assumptions.

Preference data are subjective and can be inconsistent. The system can learn annotator bias, reward-model loopholes or stylistic preference instead of substantive correctness. “Aligned” is not a binary certification; test the concrete use policy, refusals, errors and user groups.

The AI assistant's authorization still belongs in deterministic service code. Post-training can change model behavior but does not create a security boundary.

## 4. Parameter-efficient fine-tuning

Full fine-tuning updates many or all model weights. Parameter-efficient methods freeze most base parameters and train a smaller added component.

**LoRA** injects low-rank trainable updates into selected weight matrices. Instead of changing a full large matrix directly, it learns a product of smaller matrices whose rank is much lower. This reduces trainable parameter count and adapter storage. It still needs a compatible base model, training data and enough compute for activations/optimizer work.

**QLoRA** combines quantized base weights with trainable low-rank adapters, reducing memory in many settings. Exact feasible batch/context depends on architecture, quantization format, hardware kernels, optimizer and sequence length. The name alone is not a capacity guarantee.

Adapters must be versioned with their base model, tokenizer, config and evaluation. Merging or loading an adapter incorrectly can change behavior silently.

## 5. Training mechanics at a high level

- tokenize examples and apply correct role/loss masks;
- batch sequences and pad/pack according to runtime;
- forward pass computes token logits;
- cross-entropy is applied only to intended target positions;
- backward pass computes gradients;
- optimizer updates trainable parameters;
- checkpoint, evaluate, compare and preserve lineage.

Masking matters: in instruction fine-tuning, teams may train only on assistant response tokens rather than the prompt. If labels are aligned to the wrong positions, the objective becomes nonsensical. Verify a tiny batch by decoding inputs/targets and inspecting loss masks.

Training can diverge due to learning rate, precision, bad samples, optimizer state, NaNs or configuration errors. Track training/validation loss, token throughput, gradient norms and checkpoints. A falling training loss is not enough to select a model.

## 6. Data contamination and evaluation

A benchmark may appear in pretraining or fine-tuning data, inflating scores. Near duplicates, answer keys, test prompts and public benchmark leakage all threaten evaluation. Record sources and time boundaries where possible; use fresh task-specific evaluations and private holdouts responsibly.

Evaluate:
- task success and factual correctness;
- instruction adherence and structured-output validity;
- refusal/abstention and harmful behavior;
- robustness to paraphrases, long inputs and malicious content;
- language/domain/subgroup performance;
- latency, memory and cost;
- regression against prior model behavior.

Automated judges can speed triage but have preferences and blind spots. Calibrate with human review and fixed rubrics; do not treat judge outputs as unquestionable labels.

## 7. Choosing prompting, RAG or fine-tuning

| Need | First approach to assess | Why |
|---|---|---|
| Better instruction wording or output schema | Prompt/schema controls | Low cost and fast to iterate |
| Current private facts with citations | RAG or direct data API | Facts remain external and updateable |
| Stable task-specific style/behavior | Fine-tuning may help | Repeated examples can shape model behavior |
| Deterministic authorization or exact transaction | Backend rules/database | Must be authoritative and auditable |
| Better factual accuracy | Improve evidence/retrieval and evaluate | Fine-tuning alone does not ensure factual grounding |
| Very high volume/low latency | Optimize serving/model size/cache | Training is not the only cost lever |

These are starting hypotheses. Run controlled evaluations, including operational cost and failure behavior.

## Questions and answers

**Q: Can fine-tuning teach a model new private facts?**  
A: It may memorize examples, but retrieval or a database interface is usually better for facts that change, require citations or need permission checks.

**Q: What is the difference between fine-tuning and RAG?**  
A: Fine-tuning updates model parameters; RAG retrieves external evidence for a request. They can be combined, but they solve different problems.

**Q: What does LoRA save?**  
A: It reduces the number of trainable parameters and adapter storage by learning low-rank updates. Training still uses compute/memory for activations and base-model execution.

**Q: Does SFT make a model safe?**  
A: It can encourage desired behavior but cannot guarantee policy compliance. Enforce permissions and tool restrictions outside the model and evaluate adversarial cases.

**Q: Why not train a model from scratch?**  
A: It is expensive in data governance, accelerator compute, engineering and evaluation. Most product teams should begin with a suitable existing model and measure adaptation needs.

## Practice

1. Decide between prompt, RAG, SFT and backend rules for a live HR-policy assistant.
2. Draw the objective masks for a user prompt and assistant completion.
3. Explain how LoRA changes the trainable parameter set and what memory it does not eliminate.
4. Design a holdout plan resistant to duplicate users and benchmark contamination.
5. Compare base model and adapted model on quality, refusal, language coverage, latency and cost.

## Completion check

You can distinguish pretraining, SFT, preference training, LoRA/QLoRA, prompting and RAG; explain adaptation trade-offs; and design a trustworthy evaluation plan.