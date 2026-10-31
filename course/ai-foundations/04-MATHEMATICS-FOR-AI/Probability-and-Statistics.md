# Probability and statistics for AI

## Purpose

Probability describes uncertainty about outcomes. Statistics uses observed data to estimate, compare or test claims about a population or process. ML uses both: a classifier may estimate class probabilities, a training sample estimates a data distribution, and an evaluation score is itself uncertain because it is measured on finite examples.

## 1. Events, random variables and distributions

An **event** is a possible outcome or set of outcomes. A **random variable** maps outcomes to numeric values. A **probability distribution** assigns probability across values; probabilities are between 0 and 1 and total probability over all mutually exclusive outcomes is 1.

A discrete variable has countable values (number of requests in a minute); a continuous variable can take values across an interval (latency). For a continuous variable, probability is assigned to ranges via density; the probability at one exact real-valued point is generally zero.

- **Mean / expected value**: probability-weighted average outcome.
- **Variance**: expected squared deviation from the mean.
- **Standard deviation**: square root of variance, in the original measurement units.
- **Quantile**: value below which a chosen fraction of observations fall; the median is the 50th percentile.
- **Distribution tail**: extreme values; p99 latency describes a tail, not typical latency.

The mean can be pulled by extreme values. For skewed costs or latencies, inspect median and high percentiles as well.

## 2. Conditional probability and Bayes' rule

Conditional probability P(A|B) is the probability of event A given evidence B, assuming P(B)>0. Bayes' rule reverses the conditioning:

P(A|B) = P(B|A) P(A) / P(B).

Worked fraud example:
- fraud base rate P(F)=0.01;
- alert sensitivity P(Alert|F)=0.90;
- false positive rate P(Alert|not F)=0.05;
- P(Alert)=0.90×0.01 + 0.05×0.99 = 0.0595;
- P(F|Alert)=0.009/0.0595 ≈ 0.151.

So only about 15.1% of alerts are fraud under these assumptions. A seemingly good 90% sensitivity can still produce many false alerts when the base rate is low. This is why accuracy alone is misleading for rare events.

## 3. Sampling and estimation

A **population** is the full set of cases of interest; a **sample** is the subset observed. A statistic calculated from a sample estimates a population quantity. Random sampling can reduce systematic selection bias, but it does not cure a badly defined population or measurement errors.

For observations x1,...,xn, the sample mean is sum(xi)/n. The unbiased sample variance uses denominator n−1 because estimating the mean consumes one degree of freedom. The standard error measures variability of an estimator across repeated samples; for independent observations, standard error of mean is approximately s/sqrt(n). Dependence between observations changes this calculation.

A confidence interval is produced by a method that, under assumptions, covers the fixed population parameter in a stated fraction of repeated samples. A 95% interval does not mean there is a 95% posterior probability that this particular fixed parameter lies inside the already computed interval. In practice, state assumptions and avoid overclaiming precision.

## 4. Hypothesis tests and p-values

A hypothesis test compares data to a null model. A p-value is the probability, assuming the null and test assumptions are true, of data at least as incompatible with the null as the observed data. It is not the probability the null is true, nor the probability a result happened “by chance.”

Statistical significance is not practical importance. A tiny difference can become significant with a huge sample; a meaningful difference can be hard to detect with a small sample. Avoid repeated peeking and unplanned subgroup searches: they inflate false-positive risk. Prefer a pre-specified evaluation plan and effect sizes with uncertainty.

## 5. Likelihood and maximum likelihood

A likelihood measures how compatible a parameter setting is with observed data under a model. Maximum likelihood chooses parameters that maximize the likelihood of training observations. For numerical stability, implementations commonly minimize negative log-likelihood.

For binary labels y in {0,1} and predicted probability p, binary cross-entropy for one example is:
L = -[y log(p) + (1-y) log(1-p)].
A confident correct probability incurs low loss; a confident wrong probability incurs a large penalty. Numerical implementations clip or use stable logit formulas to avoid log(0).

## 6. Correlation and causation

Correlation measures association, not proof that one variable causes another. Confounders can affect both; reverse causality and selection effects can create misleading associations. Predictive ML can be useful without causal understanding, but a system that recommends interventions or explains impact needs stronger causal design.

## 7. Class imbalance and metrics

For a binary classifier:
- true positive (TP): positive correctly predicted;
- false positive (FP): negative incorrectly predicted positive;
- true negative (TN): negative correctly predicted;
- false negative (FN): positive missed.

Accuracy=(TP+TN)/all cases. Precision=TP/(TP+FP), the fraction of positive predictions that are correct. Recall=TP/(TP+FN), the fraction of actual positives found. F1 is the harmonic mean of precision and recall. ROC-AUC summarizes ranking across thresholds; precision-recall analysis is often more informative for rare positive classes. Every metric depends on the sample, label quality and operating threshold.

A score is not a decision. The threshold encodes error trade-offs, costs and policy. For high-consequence applications, analyze error severity, subgroups, abstention and human review.

## 8. Calibration

A model is calibrated if among cases assigned probability near 0.8, roughly 80% are positive over the relevant population. A model can rank cases well but be poorly calibrated. Calibration matters when probabilities drive decisions or are combined with costs. Reliability diagrams and Brier score are common diagnostics. Calibration can shift when the serving population differs from the calibration data.

## 9. Data leakage and evaluation uncertainty

**Leakage** occurs when information unavailable at prediction time influences training or evaluation. Examples:
- preprocessing all rows before splitting, thereby learning test-set statistics;
- putting near-duplicate documents across train and test;
- using a future field to predict the past;
- letting labels or target proxies enter features;
- repeatedly tuning against the final test set.

Split by time, user, organization or source when deployment requires generalization across that boundary. Use validation for choices and keep the final test set for a limited final estimate. Cross-validation reduces dependence on one split when data are appropriately independent; it cannot fix a biased dataset or an unrealistic split.

## 10. Practical Python check

~~~python
import numpy as np

scores = np.array([0.1, 0.2, 0.3, 0.9])
print("mean", scores.mean())
print("median", np.median(scores))
print("p95", np.quantile(scores, 0.95))
print("sample std", scores.std(ddof=1))
~~~

Here **ddof=1** uses the sample correction. Percentile implementations interpolate between observations, so document the convention for exact reporting. A tiny sample gives a very unstable p95: arithmetic can be precise while the estimate is not.

## Questions and answers

**Q: If a model is 99% accurate, is it good?**  
A: Not necessarily. If positives are 1% of cases, predicting “negative” always yields 99% accuracy and zero recall. Choose metrics around the errors that matter.

**Q: What does a p-value of 0.03 mean?**  
A: Under the specified null model and assumptions, data at least this incompatible would occur with probability 3%. It does not assign a 97% probability to the alternative.

**Q: Why does precision fall when a detector is applied to a new population?**  
A: The positive base rate may have changed, and conditional error rates can also shift. Bayes' rule shows that precision depends on prevalence as well as sensitivity and false-positive rate.

**Q: Can I use accuracy to compare two models?**  
A: Only if the classes, costs, threshold and use case make it relevant. Inspect a confusion matrix, threshold behavior, uncertainty, subgroup performance and operational costs.

## Practice

1. Recalculate the fraud example for a 0.1% base rate.
2. Construct a confusion matrix where accuracy is high but recall is zero.
3. Explain in plain language what a confidence interval procedure guarantees.
4. Identify leakage in a time series example where random splitting mixes past and future.
5. Design metrics for a support-ticket classifier where missed urgent tickets are much more costly than extra reviews.

## Completion check

You can distinguish probability, likelihood, statistics and causal claims; calculate precision/recall; explain base-rate effects, confidence intervals and p-values; and identify leakage paths before training a model.