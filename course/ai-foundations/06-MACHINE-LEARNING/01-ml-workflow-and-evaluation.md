# Machine learning workflow and trustworthy evaluation

## Learning outcomes

You should be able to frame a problem as a learning task, define what a prediction means, construct a defensible dataset split, build a baseline, choose metrics around the cost of errors, recognize leakage, and describe what would need monitoring after deployment.

## 1. What machine learning learns

In traditional software, a developer writes rules that map inputs to outputs. In supervised machine learning, we provide examples of inputs and desired outputs; a learning algorithm fits a model whose parameters capture patterns that may generalize to new examples.

A simple notation:
- x: input features, such as ticket text and metadata;
- y: target label, such as urgency;
- f_theta: model with parameters theta;
- y_hat = f_theta(x): model prediction;
- L(y, y_hat): loss function measuring a chosen kind of prediction error.

Training chooses theta to make aggregate training loss smaller. Generalization means useful behavior on relevant cases not used to fit the parameters. It is an empirical claim supported by evaluation, not guaranteed by the algorithm.

### Main learning settings

- **Supervised learning**: examples have labels; classification predicts categories, regression predicts numeric values.
- **Unsupervised learning**: no target labels are supplied; methods find structure such as clusters or compressed representations.
- **Self-supervised learning**: the data itself supplies a training signal, such as hiding part of an input and predicting it. Modern language-model pretraining is a common example.
- **Semi-supervised learning**: a small labeled set is combined with a larger unlabeled set.
- **Reinforcement learning**: an agent chooses actions and receives rewards over time; delayed consequences make credit assignment difficult.
- **Online learning**: model updates as data arrives, requiring safeguards against drift, feedback loops and harmful updates.

The names describe how a learning signal is obtained. They do not tell you whether a model is accurate, fair or safe.

## 2. Frame the problem before selecting a model

Write a one-paragraph problem statement:
1. Who uses the prediction and what decision follows?
2. What information is available at decision time?
3. What exactly is the target and how is it labeled?
4. What errors matter, and who bears their cost?
5. What baseline would be useful?
6. What latency, throughput, privacy and explanation constraints apply?
7. What evidence would justify launch or rollback?

For example, “classify a support ticket” is incomplete. Is the output routing category, urgency, or response suggestion? Is urgency known at intake or only after an incident? Do humans confirm labels? Are tickets from the same customer allowed in both train and test? These choices define the task more than the classifier does.

**Unit of analysis** is the entity receiving one prediction: request, customer, document, time window or conversation. Splitting and metrics should respect this unit.

## 3. Data: collection, labels and quality

A row is not automatically an example. Define:
- feature schema and types;
- label definition and source;
- collection period and population;
- missing-value meaning;
- duplicate and near-duplicate policy;
- inclusion/exclusion criteria;
- privacy, retention and access controls;
- lineage and version for reproducibility.

Labels may be noisy, delayed, subjective or biased. An “expert” label can still encode inconsistent standards. Track disagreement, adjudication and label changes. A model may learn how a label was assigned rather than the phenomenon the team thinks it represents.

Use descriptive summaries before fitting: counts, missingness, ranges, category frequencies, time trends, duplicates, label balance and subgroup coverage. Plot distributions and inspect examples under approved privacy controls. Do not copy sensitive raw examples into notebooks or logs without authorization and minimization.

## 4. Train, validation and test sets

- **Training set**: used to fit model parameters.
- **Validation set**: used to select hyperparameters, thresholds, features and model variants.
- **Test set**: a final held-out estimate used after decisions are substantially fixed.

A test set is not a permanent oracle. Repeated experimentation against it leaks information through human decisions. If the project is exploratory, rotate evaluation sets or collect a later prospective sample.

The split must simulate deployment:
- **Random split**: reasonable only when examples are independent and deployment resembles random draws from the same population.
- **Group split**: keep all records from one person, company, device or source together when repeated identities would leak.
- **Temporal split**: train on the past and test on later data for future prediction.
- **Geographic/site split**: test whether behavior transfers to unseen regions or organizations.
- **Stratified split**: preserve approximate class proportions; it does not solve group or time leakage by itself.

For a time-based task, avoid random splitting across time. If repeated records for one user appear on both sides, a model may memorize that user's behavior and appear to generalize.

## 5. Leakage: the silent evaluation failure

Leakage means training or selection uses information that would not legitimately be available for a new prediction, or the held-out set influences decisions. It can create an impressive but fictional score.

Examples:
- fitting a scaler, imputer, vocabulary or feature selector on the full dataset before splitting;
- using a post-outcome field (refund status) to predict an earlier event;
- splitting chunks of one document into both train and test;
- placing near-identical customer examples on both sides;
- selecting a threshold after inspecting test labels;
- using label-derived aggregates that include the current row;
- tuning repeatedly to the same “final” benchmark.

Fit every learned preprocessing step on training data only. A pipeline keeps transformation fitting inside each training fold:

~~~python
from sklearn.impute import SimpleImputer
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler

model = make_pipeline(
    SimpleImputer(strategy="median"),
    StandardScaler(),
    LogisticRegression(max_iter=1000, class_weight="balanced"),
)
model.fit(X_train, y_train)
predictions = model.predict(X_test)
~~~

The example assumes numeric features and a classification target. Text, categorical columns, groups and time require an appropriate transformer and splitter. The key principle is that the pipeline is fitted on training data, then applied unchanged to held-out data.

## 6. Baselines before complexity

A baseline is a simple reference point. Common baselines:
- predict the most frequent class;
- predict a historical rate or last-known value;
- linear/logistic regression;
- a small decision tree;
- keyword or business rule system;
- retrieval from existing cases.

A baseline helps answer whether the complicated model adds measurable value. A useful baseline also exposes bad labels and split bugs. If a “sophisticated” model barely improves upon majority prediction, investigate the data and task before adding more layers.

## 7. Metrics and threshold choice

Start with a confusion matrix for classification:
- TP: predicted positive, actually positive;
- FP: predicted positive, actually negative;
- TN: predicted negative, actually negative;
- FN: predicted negative, actually positive.

Accuracy=(TP+TN)/(TP+FP+TN+FN). Precision=TP/(TP+FP). Recall=TP/(TP+FN). Specificity=TN/(TN+FP). F1=2×precision×recall/(precision+recall), undefined if both denominators create no positive predictions unless a convention is chosen.

For regression, common metrics include MAE (average absolute error), MSE (average squared error), RMSE (square root of MSE), and R². Metrics differ in scale, sensitivity to outliers and interpretation. Never report a metric without specifying the evaluated sample, target scale and split method.

Many classifiers output a score or probability. A threshold converts it to a decision. Lowering the threshold generally increases recall and may increase false positives; raising it generally increases precision and may miss more positives. Choose a threshold using validation data and an explicit cost or service constraint. Do not treat 0.5 as inherently correct.

For rare classes, precision-recall curves can show trade-offs more clearly than ROC curves. For ranking, assess whether relevant items appear near the top (for example precision@k or recall@k). For probabilistic decisions, evaluate calibration. For generative systems, exact-match accuracy alone is rarely enough; assess correctness, grounding, refusal behavior, latency, cost and human utility.

## 8. Bias-variance and overfitting

A model underfits when it cannot capture important patterns even on training data. A model overfits when it fits training examples but performs poorly on new cases. More complexity can lower training error while worsening generalization.

The bias-variance framing describes sources of expected prediction error: overly restrictive assumptions create bias; sensitivity to the particular sample creates variance. More data, regularization, cross-validation, early stopping and suitable model complexity can help, but no technique repairs mislabeled or unrepresentative data automatically.

Inspect learning curves and error slices:
- training and validation performance together across model sizes or data sizes;
- errors by time, input length, class, geography or user group when lawful and meaningful;
- representative false positives and false negatives;
- confidence versus correctness.

A single average hides important failure clusters.

## 9. Cross-validation and hyperparameter tuning

K-fold cross-validation divides training data into folds, repeatedly trains on some folds and evaluates on the held-out fold. It can use limited data more efficiently when observations meet the split assumptions. Use group-aware or time-aware methods when needed. For small rare-positive samples, estimates may have wide uncertainty even across folds.

Hyperparameters are choices outside the fitted coefficients (tree depth, regularization strength, learning rate). Tune on training/validation procedures only. Searching many combinations increases the chance of selecting noise. Record configurations and use a final untouched test or later evaluation window.

## 10. Class imbalance and cost-sensitive decisions

Imbalance means classes occur at different rates. It is not always a problem: a rare class can still be easy to detect. More often, default training and accuracy obscure important errors.

Possible tools include class weights, carefully chosen sampling, threshold adjustment, anomaly detection, and gathering more positive examples. Resampling must occur within each training fold; doing it before splitting can leak duplicates or distort test prevalence. Preserve realistic class prevalence in final evaluation unless the chosen metric explicitly accounts for sampling.

Build a cost matrix with stakeholders: cost of false alarm, cost of miss, review capacity, and consequences of delay. Optimize a decision rule around real constraints, not around a convenient metric alone.

## 11. Distribution shift and drift

- **Covariate shift**: input distribution P(x) changes.
- **Label shift**: class prevalence P(y) changes.
- **Concept drift**: relationship P(y|x) changes.
- **Data quality drift**: schemas, missingness or instrumentation change.
- **Feedback loop**: predictions alter what gets observed or labeled later.

Monitor input and output summaries, quality checks, error metrics when labels arrive, subgroup behavior, and service performance. A change alert is a reason to investigate, not proof that a model is broken. Plan rollback, fallback, re-evaluation and retraining criteria before launch.

## 12. From notebook to model artifact

A reproducible experiment records:
- data source/version and split definition;
- code commit and dependency versions;
- preprocessing and model configuration;
- random seeds where meaningful;
- metrics, uncertainty and error analysis;
- model artifact checksum and schema;
- known limitations and intended use.

A saved estimator is executable behavior from a trusted source. Loading an untrusted serialized model can execute unsafe payloads depending on the serialization method. Treat artifacts as supply-chain inputs: verify provenance and access.

## Questions and answers

**Q: Do I split before or after preprocessing?**  
A: Split first, then fit learned preprocessing on training data only. Use a pipeline so each cross-validation fold independently fits its transformations.

**Q: Is a validation set optional?**  
A: For any meaningful model selection, you need data or a procedure that plays the validation role. Tiny data may need cross-validation, but the uncertainty and split assumptions still matter.

**Q: If test performance is poor, can I change the model and test again?**  
A: You can explore, but that test set is now informing choices. Treat it as development data and obtain a fresh final evaluation sample or later time window.

**Q: Should every AI project use deep learning?**  
A: No. Use the simplest approach that meets measured quality and operating needs. For small structured datasets, a tree or linear model may be stronger, cheaper and easier to explain.

**Q: Does cross-validation prove deployment performance?**  
A: No. It estimates performance under its sampling and split assumptions. Deployment can differ in time, users, instrumentation, incentives and data quality.

## Practice

1. Frame a ticket-priority prediction task by defining the prediction time, label, unit, cost of FP/FN and likely leakage fields.
2. Create a majority-class baseline and compare it with logistic regression on a group-aware split.
3. Draw a timeline showing a temporal split; list which transformations must be fitted using the past only.
4. Find a threshold using validation data under a minimum-recall constraint; report the resulting precision and review volume.
5. Write a monitoring plan with one data-quality metric, one delayed quality metric and one service metric.

## Completion check

You can defend a dataset split, recognize leakage, compare a model against a baseline, choose and explain metrics, analyze error slices, and describe how evaluation limitations affect launch confidence.