# Transformers and attention, from tokens to predictions

## Learning goals

You should be able to trace text through a Transformer, explain what Query/Key/Value vectors do, calculate a tiny attention example, distinguish encoder and decoder behavior, and understand why context length and token count affect inference.

## 1. The language-modeling task

A language model estimates a distribution over tokens. Given a token sequence t1,...,tn, an autoregressive model learns to estimate:

P(t1,...,tn) = product over i of P(ti | t1,...,t(i-1)).

In generation, the model processes a prompt and predicts a distribution for the next token. A decoding rule selects or samples a token, appends it, and repeats. This is not the same as searching a database or verifying truth. The objective rewards plausible continuations in training data; factuality and instruction-following require additional methods and evaluation.

## 2. Text becomes token IDs

A tokenizer maps text into tokens and token IDs. A token can be a word, subword, punctuation mark, whitespace-related piece, or byte-like unit depending on the tokenizer. Token boundaries are not identical to words. Different languages and unusual strings can require very different numbers of tokens.

A vocabulary has V entries. The model receives integer IDs, not raw characters. The tokenizer and model must match. Using the wrong tokenizer can change input IDs, token count and output interpretation.

A prompt is limited by a context window measured in tokens. The usable budget includes system/developer instructions, conversation history, retrieved passages, tool results and generated answer. Counting only the user's latest message underestimates the actual context.

## 3. Embeddings and positions

A token embedding table maps each token ID to a learned vector of width d_model. For a sequence of length T, the token vectors form a matrix with conceptual shape (T,d_model).

Self-attention alone does not know the order of tokens; positions or relative relationships must be injected. Families include learned absolute positional embeddings, sinusoidal encodings and rotary position methods. They differ in how positions interact with attention and generalize to longer sequences. Do not treat a specific positional method as universally best.

A residual stream is the sequence of d_model-wide vectors flowing through the network. Layers repeatedly add information to this stream.

## 4. Query, Key and Value intuition

For hidden states X, learned matrices project each position into:
Q = X W_Q
K = X W_K
V = X W_V

- Query: what information is this position seeking?
- Key: what matching information does each position advertise?
- Value: what content is passed along if a position receives attention?

For one query vector q and keys k_j, scaled dot-product attention computes scores s_j = q·k_j / sqrt(d_k), converts scores to weights with softmax, and returns weighted values sum_j weight_j v_j.

The division by sqrt(d_k) keeps dot-product magnitudes in a useful range as dimension grows; very large score magnitudes can make softmax extremely peaked and gradients less useful.

### Tiny worked attention example

Let one-dimensional query q=1, keys k1=1 and k2=2, values v1=10 and v2=20. Ignore the scale divisor for this tiny example:
- scores are [1,2];
- softmax([1,2]) is approximately [0.269, 0.731];
- output is 0.269×10 + 0.731×20 ≈ 17.31.

The second value contributes more because its key better matches the query. In an actual model, many dimensions, positions, masks, heads and learned projections are involved. This example explains the weighting operation, not language understanding.

## 5. Self-attention matrix form

For sequence matrices Q, K, V, attention is:
Attention(Q,K,V) = softmax(Q K-transpose / sqrt(d_k)) V.

QK-transpose creates one score per query-position/key-position pair. For a sequence of T tokens, this score matrix is T×T. Standard dense self-attention therefore has a sequence-length-squared score/mixing cost. This quadratic dependence is why long prompts can increase computation and memory sharply. Optimized attention kernels reduce intermediate memory traffic; they do not make every exact dense attention operation independent of sequence length.

## 6. Causal masks and padding masks

An autoregressive decoder must not use future tokens when predicting the next token. A causal mask prevents position i from attending to positions j>i, commonly by assigning those scores negative infinity before softmax so their weights become zero.

Padding masks stop attention from treating padding positions as real content in a batch. Padding and causal masks solve different problems. Incorrect mask shape or polarity can silently leak future information or hide valid tokens.

Encoder models may attend bidirectionally across all input tokens. Decoder-only models use causal attention. Encoder-decoder models first encode an input and then decode while attending to encoded states. These architectures suit different objectives; encoder-only models are not simply smaller chat models.

## 7. Multi-head attention

Multi-head attention runs several learned attention projections in parallel. Each head can form a different relational pattern; the outputs are concatenated and projected back to d_model.

If d_model=512 and there are 8 heads, a common design uses d_k=64 per head. This is a shape illustration, not a universal configuration. The model does not promise that one named head represents a clean human concept. Interpretability claims need evidence.

## 8. Transformer block

A typical block includes:
1. multi-head attention;
2. residual addition;
3. normalization;
4. position-wise feed-forward network;
5. another residual and normalization.

Exact ordering varies. Pre-normalized and post-normalized designs have different training properties. The feed-forward network projects each token vector to a wider hidden dimension, applies a nonlinear transformation (often a gated variant in newer models), then projects back. It mixes features within a position; attention mixes information across positions.

Residual connection: x_next = x + F(Norm(x)) in a common pre-norm form. This lets a layer add a refinement while preserving a direct signal path. Layer normalization generally operates over features within a token representation and does not depend on other examples in the batch.

## 9. Encoder, decoder and encoder-decoder families

- **Encoder-only** (for example, BERT-style): bidirectional context representation; often useful for classification, retrieval embeddings or token tagging after task adaptation.
- **Decoder-only** (GPT-style): causal next-token prediction; common for text generation and chat.
- **Encoder-decoder** (T5-style): encoder represents an input, decoder generates an output conditioned on it; useful for sequence-to-sequence tasks.

These are broad patterns, not hard limits. Compare objective, architecture, available weights, licensing, serving support and task evaluation.

## 10. Pretraining, instruction tuning and preference alignment

- **Pretraining**: learn broad statistical patterns from large corpora, often with next-token prediction or another self-supervised objective.
- **Supervised fine-tuning (SFT)**: train on input/output demonstrations to shape task behavior.
- **Preference optimization**: use preference comparisons or a reward signal to encourage desired outputs. Approaches include reward-model-based RL and direct preference objectives.
- **Alignment**: broad, contested term for shaping behavior toward intended constraints and preferences; it is not a formal guarantee of safety.

Each stage can improve some behaviors and introduce trade-offs. A model can still hallucinate, reveal sensitive training artifacts, follow malicious instructions or perform unevenly across languages and groups. Evaluation must target the intended use.

## 11. Next-token loss and teacher forcing

During training, the model is shown a sequence and is trained to predict the next token at every eligible position. For tokens t1...tT, loss is commonly the negative log probability assigned to the target next token, averaged or summed with a masking convention.

Teacher forcing feeds the true preceding tokens during training. At generation, the model consumes its own previous outputs. Errors can compound at inference even if one-step validation loss is good. This is one reason sequence-level behavior needs direct evaluation.

## 12. Decoding methods

- **Greedy**: choose the highest-probability next token. Deterministic but may be repetitive or dull.
- **Temperature**: rescales logits before softmax. Lower values sharpen the distribution; higher values flatten it. It does not add knowledge.
- **Top-k**: sample only among k highest-probability tokens.
- **Top-p / nucleus**: sample from the smallest set whose cumulative probability reaches p.
- **Beam search**: track several candidate sequences, often useful in structured tasks but not a universal chat improvement.
- **Stop sequence / max tokens**: bound generation length and detect intended end markers.

Sampling choices change style and variability as well as quality. Fix decoding configuration when comparing models.

## 13. Context length, truncation and position limits

A larger context window can make more information available, but it does not guarantee the model will attend equally well to every part. Long context increases prefill work and often memory. Relevant evidence should be selected and organized; dumping every document into a prompt is rarely robust.

Truncation can silently remove the system instruction, question or evidence depending on policy. Define which content is preserved, summarize or retrieve intentionally, and report when required context cannot fit.

## Questions and answers

**Q: Does attention mean the model reasons like a person paying attention?**  
A: No. Attention is a learned weighted information-mixing operation. The name is an analogy; it does not establish human-like awareness or reasoning.

**Q: Why are there separate Q, K and V matrices?**  
A: Learned projections let the model compute match signals and transported content in different representations. It can learn what to look for separately from what to pass along.

**Q: Why can a Transformer not look at future tokens during generation?**  
A: The causal mask enforces the training/inference condition that the next token depends only on prior tokens. Without it, training targets would leak.

**Q: Does a bigger context window solve retrieval?**  
A: No. It gives a larger possible input, but relevant evidence still needs selection, ordering, citation and evaluation; long contexts can be costly and hard to use reliably.

**Q: Is a token the same as a word?**  
A: No. Tokenizers split text according to learned or designed vocabulary units. One word can be multiple tokens, and punctuation/whitespace may affect tokenization.

## Practice

1. Compute softmax for scores [0, 0] and [0, 2], then calculate a weighted value.
2. Draw shapes for Q, K, QK-transpose, attention weights and output for T=5, d_k=4, d_v=6.
3. Explain what breaks if the causal mask is inverted.
4. Compare encoder-only, decoder-only and encoder-decoder using a classification, completion and translation task.
5. Estimate how the pairwise attention matrix size changes when T doubles.

## Completion check

You can trace text to token IDs, embeddings, positional information, attention blocks and next-token distributions; explain causal masking and multi-head attention; and describe why long sequences have higher inference cost.