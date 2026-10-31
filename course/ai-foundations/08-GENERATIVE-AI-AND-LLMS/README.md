# Generative AI and LLMs

This subject connects NLP foundations to model internals and then to safe application behavior. A language model estimates distributions over tokens; fluent text is not a verified fact or permission.

## Study sequence

1. [NLP foundations](03-nlp-foundations.md): text tasks, preprocessing, n-grams, TF-IDF, embeddings and language evaluation.
2. [Transformers and attention](01-transformers-and-attention.md): tokenization, embeddings, Q/K/V, masks, blocks and next-token training.
3. [LLM training and adaptation](04-llm-training-and-adaptation.md): pretraining, SFT, preference training, LoRA/QLoRA and limits.
4. [Prompt design and structured output](05-prompt-design-and-structured-output.md): task contracts, context, schemas, injection boundaries and iteration.
5. [LLM application evaluation](06-llm-application-evaluation.md): eval datasets, human/LLM judges, uncertainty and release gates.
6. [LLM inference and serving](02-llm-inference-and-serving.md): prefill/decode, memory, KV cache, batching, reliability and cost.

Enforce authorization and output rules outside the model. Pair theory with the free [Hugging Face LLM Course](https://huggingface.co/learn/llm-course/en/chapter1/1) and your [AI From Scratch LLM volume](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/llm). Check current implementation docs before choosing versions.