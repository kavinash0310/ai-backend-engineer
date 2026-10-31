# Information theory for machine learning

## Why this subject appears

Information theory studies uncertainty and information. It gives useful language for classification losses, language-model training, compression, entropy and KL divergence. You need the definitions and examples first; deep coding theorems can follow later.

## 1. Entropy

For a discrete random variable X with outcomes x and probabilities p(x), Shannon entropy is:

H(X) = - sum_x p(x) log2 p(x).

Entropy is expected surprise under the distribution. A fair coin has 1 bit of entropy. A coin that always lands heads has 0 bits because its outcome is known. Entropy is not the amount of meaning in a sentence; it is uncertainty under a defined distribution and logarithm base.

For probabilities [0.5, 0.5], entropy is 1 bit. For [0.9,0.1], entropy is about 0.469 bits. The more concentrated distribution is less uncertain.

## 2. Cross-entropy

If the true data distribution is p and a model predicts q, cross-entropy is:

H(p,q) = - sum_x p(x) log q(x).

For one observed class y, the loss is -log q(y): assigning low probability to the observed outcome incurs a high penalty. For one-hot targets, cross-entropy differs from entropy by a KL term:
H(p,q) = H(p) + D_KL(p||q).

During supervised classification, minimizing cross-entropy encourages probability assigned to the correct labels to increase. In next-token training, each observed next token is a target. A low average cross-entropy means the model assigns higher probability to observed tokens under the training distribution; it does not prove generated answers are factual or useful.

## 3. KL divergence

Kullback-Leibler divergence is:

D_KL(p||q) = sum_x p(x) log(p(x)/q(x)).

It measures a directional difference between distributions under p's weighting. It is not symmetric: D_KL(p||q) generally differs from D_KL(q||p), and it is not a geometric distance in the ordinary metric sense. It is nonnegative when distributions are valid and the usual assumptions hold.

KL divergence appears in variational methods, distribution matching and some preference/training objectives. The direction matters: switching p and q changes which errors are emphasized.

## 4. Entropy in classifiers and language models

Softmax turns logits into a categorical distribution. Entropy of that output can summarize how spread out the model's scores are, but high or low entropy does not guarantee correctness or calibrated uncertainty. A confidently wrong model can have low entropy.

Per-token negative log likelihood averaged over a held-out corpus is cross-entropy. Perplexity is exp(cross-entropy) if natural logarithms are used (or 2 to the cross-entropy if measured in bits). Lower perplexity means better assigned probability to that evaluation text under the tokenizer/data convention. It does not directly measure instruction following, factuality, toxicity or downstream task success.

## 5. Mutual information

Mutual information I(X;Y) measures how much knowing X reduces uncertainty about Y:
I(X;Y)=H(Y)-H(Y|X).
It is symmetric and nonnegative under standard definitions. It helps reason about dependence and feature-target relationships, but estimating it reliably from finite high-dimensional data is hard. Association measured by mutual information is not automatically causal.

## 6. Compression and representations

Compression replaces a representation with fewer bits while preserving chosen information. Lossless compression preserves exact data; lossy compression discards detail. Quantization and dimensionality reduction are lossy transformations that trade precision/quality for size or compute. The useful question is which downstream behavior is preserved and how it is measured.

## Questions and answers

**Q: What does a high-entropy prediction mean?**  
A: The model's probability mass is spread across outcomes. It may reflect ambiguity, uncertainty or a poorly trained model; it is not a calibrated confidence claim by itself.

**Q: Is cross-entropy just accuracy?**  
A: No. Accuracy checks the chosen top class. Cross-entropy also penalizes how little probability the model assigns to the actual class.

**Q: Is KL divergence a symmetric distance?**  
A: No. Direction matters, and it generally is not symmetric.

**Q: Does low perplexity mean a good assistant?**  
A: No. It means strong next-token probability on the evaluated corpus under a specific tokenizer and setup. Product quality needs task and safety evaluations.

## Practice

1. Compute entropy for a fair coin and a 90/10 coin.
2. Compare cross-entropy when the correct class probability is 0.9 versus 0.1.
3. Explain why KL(p||q) differs from KL(q||p) using a rare class.
4. List four behaviors perplexity does not measure.
5. Describe what information a lossy 8-bit representation could discard and how you would test impact.

## Completion check

You can interpret entropy, cross-entropy, KL divergence and perplexity; explain their assumptions and limits; and connect them to classification and language-model training.