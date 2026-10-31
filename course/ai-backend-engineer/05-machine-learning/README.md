# Machine Learning

**Goal:** build sound prediction experiments and reason about what results mean outside a notebook.

## Lifecycle

Problem definition → data collection/permission → data audit → baseline → split strategy → preprocessing/features → model → validation/tuning → final untouched test → deployment → monitoring → retraining or retirement.

### Required subjects

Regression, classification, clustering and dimensionality reduction; linear/logistic models, trees/ensembles, nearest-neighbor, SVM and probabilistic baselines; metrics, calibration, thresholds, imbalance, cross-validation, data leakage, bias/variance, over/underfit, distribution shift, fairness and monitoring.

**Critical note:** choose the split before fitting preprocessing. Use grouped or time-based splits when samples are dependent. Never tune on the final test set.

Start with [ML Lifecycle and Evaluation](./01-ml-workflow-and-evaluation.md). For deeper subject volumes see [AI From Scratch ML](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/ml).


## Detailed lessons

- [ML workflow and evaluation](./01-ml-workflow-and-evaluation.md) — problem framing, splits, leakage, metrics, calibration, drift and reproducibility.
- [Classical model families](./02-classical-model-families.md) — linear/logistic, trees/ensembles, neighbors, Bayes, SVM, clustering and PCA.
- Continue with the companion [AI From Scratch ML course](https://github.com/kavinash0310/ai-from-scratch/tree/main/course/ml) for deeper worked theory and exercises.

- [Reinforcement learning](./03-reinforcement-learning.md) — MDPs, return, value/policy methods, Q-learning and safety.
