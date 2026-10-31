# Classical machine-learning model families

## Why compare families

Every model family encodes assumptions about how inputs relate to outputs. Choose a candidate because its assumptions and cost fit the task, then compare it to a baseline on a deployment-realistic evaluation set. “More advanced” does not mean “better for this dataset.”

## 1. Linear regression

For numeric target y and feature vector x, linear regression predicts y_hat = w·x+b and often minimizes mean squared error. It is linear in its parameters and creates a flat hyperplane in feature space.

Strengths: fast, simple, interpretable under conditions, strong baseline. Limits: nonlinear patterns need feature transforms/interactions; outliers affect squared error; correlated features can make coefficients unstable. Ridge adds an L2 penalty; Lasso adds an L1 penalty that can drive some coefficients to zero; Elastic Net combines them. Scaling matters for regularized optimization. A coefficient is not automatically causal.

## 2. Logistic regression

Logistic regression calculates a linear score z=w·x+b and maps it through a sigmoid for binary classification. Softmax is common for mutually exclusive multiclass labels. It directly models class probabilities/scores given features.

Strengths: efficient, compact, useful baseline for sparse text. Limits: linear boundary unless features transform; probabilities can be miscalibrated; the decision threshold must match error costs. Never assume 0.5 is operationally correct.

## 3. Decision trees

A tree recursively splits features into regions, choosing splits that reduce an impurity measure such as Gini/entropy or regression squared error. Leaves emit a class or numeric prediction.

Strengths: nonlinear splits, little feature scaling, small trees can be inspected. Limits: deep trees overfit and can be unstable; large trees are difficult to interpret. Control depth and minimum leaf size, then validate.

## 4. Random forests

A forest trains many trees on bootstrap samples and random feature subsets; outputs are aggregated. Averaging often lowers variance relative to one tree.

Strengths: robust tabular baseline, nonlinear interactions, parallelizable. Limits: larger model and inference footprint; less transparent; regression trees extrapolate poorly beyond observed ranges. Feature importance can mislead when features are correlated or high-cardinality.

## 5. Gradient-boosted trees

Boosting adds trees sequentially so new trees reduce errors left by the previous ensemble. Implementations vary in objective, sampling and regularization.

Strengths: often strong for structured/tabular data. Limits: tuning matters; rounds are sequential; linear models can be better for sparse high-dimensional text; explanations are not automatically causal. Compare fairly under the same split and reasonable tuning budget.

## 6. Nearest neighbors and support vector machines

**k-nearest neighbors** predicts from nearby training examples. It is a simple local baseline but stores data and can be slow at prediction. Distance becomes less informative in high dimensions; scaling and metric matter. Approximate-neighbor indexes trade exactness for speed.

**Support vector machines** choose a decision boundary with a large margin; kernel functions can represent nonlinear boundaries. Linear SVMs can work well on sparse text. Kernel training may scale poorly; probability estimates need calibration.

## 7. Naive Bayes

Naive Bayes applies Bayes' rule with a conditional independence assumption among features given class. The assumption is often false, yet the method can work well for sparse text and small datasets.

Strengths: fast, low memory. Limits: simplification may reduce accuracy and probability calibration. Validate variants on the task instead of choosing by name.

## 8. Clustering and dimensionality reduction

**k-means** assigns vectors to k clusters to reduce within-cluster squared distance. It is sensitive to scale, initialization and outliers, and favors roughly spherical clusters. Density-based and hierarchical approaches make different assumptions.

Clustering has no truth labels by default. A cluster ID is not automatically a meaningful business segment; assess stability and usefulness with domain reviewers.

**PCA** projects data onto directions of greatest variance. It can compress correlated numeric features, but high variance is not necessarily predictive. Fit PCA only within training folds; fitting it on all data leaks test information.

## 9. Preprocessing and model pairing

Standardization subtracts training mean and divides by training standard deviation. Min-max scaling maps training range to an interval. Robust scaling uses medians/quantiles. Fit these transforms on training data only and apply the fitted transform unchanged to validation, test and production.

Trees often need less scaling than distance/gradient models. Missing values, categories and text still require explicit handling. One-hot encoding creates sparse columns; target encoding requires leakage-safe cross-fitting.

## 10. Candidate-selection recipe

1. Define task and prediction time.
2. Establish a simple baseline and realistic split.
3. Select families based on data type, sample size, latency and interpretability.
4. Fit preprocessing inside a pipeline.
5. Tune only on training/validation procedures.
6. Inspect error slices and probability calibration.
7. Measure inference latency, memory and artifact size.
8. Keep an untouched final evaluation.
9. Document intended use, data, metrics, limits and monitoring.

## Questions and answers

**Why might boosted trees beat a neural network on tabular data?**  
Trees can represent thresholds and interactions effectively on modest datasets. Neural nets may need more data and careful tuning. Measure both.

**Why scale features for nearest neighbors?**  
Distance sums differences across coordinates; a large-unit feature can dominate even if unimportant.

**Is feature importance a causal explanation?**  
No. It is a model-specific summary and can change with correlated features or the explanation method.

**Can clustering discover the true groups?**  
It finds structure under a representation, metric and algorithm. Human meaning requires external validation.

## Practice

1. Compare logistic regression and a tree baseline on the same split using precision, recall and inference latency.
2. Show how an unscaled currency column can dominate Euclidean distance.
3. Compare shallow and deep trees on synthetic nonlinear data and plot train/validation errors.
4. Explain why PCA belongs inside a training pipeline.
5. Write a model card describing use, data coverage and limitations.

## Completion check

You can select candidate families from their assumptions, explain strengths and limits, and compare both predictive quality and operational cost without popularity-based reasoning.