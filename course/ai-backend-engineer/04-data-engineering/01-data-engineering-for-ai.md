# Data engineering foundations for AI

## Why AI engineers need data engineering

Models learn from data and production systems depend on it. If extraction, identifiers, timestamps, schemas, access rules or updates are wrong, a more advanced model can amplify the error. Data engineering makes information discoverable, trustworthy, reproducible and usable at the right time.

## 1. Database basics

A relational database stores records in tables with defined columns and types. A primary key identifies a row; a foreign key references another row. Constraints enforce invariants close to storage. SQL expresses queries over rows and columns.

~~~sql
SELECT category, COUNT(*) AS ticket_count
FROM support_ticket
WHERE created_at >= DATE '2026-01-01'
GROUP BY category
ORDER BY ticket_count DESC;
~~~

Important concepts:
- **Normalization** reduces update anomalies by separating related facts.
- **Denormalization** duplicates or precomputes data for read efficiency; it adds consistency work.
- **Index** accelerates selected queries at storage/write cost.
- **Transaction** groups changes under atomicity, consistency, isolation and durability properties.
- **NULL** means unknown/missing, not zero or empty string; SQL comparisons use three-valued logic.

AI features often need joins across product, event, content and labeling systems. Avoid leaking test/user identity through an accidental join. Parameterize SQL; never concatenate user input into query strings. Use least-privilege database accounts and query budgets.

## 2. OLTP, analytics and warehouses

- **OLTP** systems support operational transactions with current records and many small reads/writes.
- **Analytics warehouse/lakehouse** systems support large scans and transformations across historical data.
- **Object storage/data lake** stores files and semi-structured or raw data; governance and schemas are still needed.
- **Feature store** manages reusable ML features in offline and sometimes online form, with attention to training-serving consistency.

Do not run unbounded model-training scans against a production transactional database without capacity and isolation review. Extract to appropriate analytics storage or use carefully controlled replicas/snapshots.

## 3. Batch, streaming and event time

A **batch pipeline** processes a bounded collection periodically. A **streaming pipeline** processes events continuously or in small windows. These designs differ in freshness, complexity, replay and late-arriving data behavior.

Important time concepts:
- **event time**: when the event happened in the source domain;
- **processing time**: when a pipeline handled it;
- **ingestion time**: when a system first received it;
- **watermark**: an estimate that events earlier than a time are unlikely to arrive.

A delayed event can change historical aggregates. Define whether data is corrected, reprocessed, or ignored after a cutoff.

## 4. Data contracts and schema evolution

A data contract defines schema, semantics, owner, quality expectations and compatibility. Schema validation catches missing or changed fields before a model consumes them. Additive optional fields may be backward compatible, but changing units or meaning is often a breaking change even if column name/type stay the same.

Record:
- schema/version;
- units, allowed ranges and null semantics;
- owner/source and update expectation;
- compatibility rules;
- sensitive-data classification;
- quality thresholds and failure behavior.

A field named **amount** is ambiguous if the currency, tax basis and time period are absent.

## 5. Data quality dimensions

Common dimensions:
- completeness: are required values present?
- validity: do values follow domain constraints?
- uniqueness: are duplicates expected or accidental?
- consistency: do sources agree?
- timeliness: is data fresh enough?
- representativeness: does it cover intended deployment cases?
- lineage: can you trace how this dataset was formed?

Quality checks should report counts and categories without exposing private values. Distinguish a failed pipeline from a valid zero-row result. Quarantine malformed data when safe; do not silently coerce everything.

## 6. ETL/ELT and reproducibility

ETL extracts, transforms then loads; ELT loads then transforms in destination systems. AI datasets commonly combine source snapshots, feature transformations, labels and split definitions. Save the exact transformation code/version and data snapshot or reproducible query.

A reproducible dataset build specifies:
1. source versions and time range;
2. joins and filters;
3. label generation;
4. deduplication and exclusion rules;
5. feature transformation;
6. train/validation/test split logic;
7. quality report and privacy basis.

A random seed alone does not make a changing source dataset reproducible.

## 7. Feature engineering and leakage

A feature must be available at prediction time and computed consistently online/offline. A user-churn model may use current subscription state but not a cancellation reason recorded after the event. A support classifier may use the first message but not an agent's final resolution note.

**Training-serving skew** happens when offline feature creation differs from online serving (timezone, default, normalization, missing-value handling, freshness). Use shared transformation logic or explicit parity checks. Monitor feature distributions and missingness.

## 8. Data versioning and lineage

Track source data, derived dataset, code, configuration, labels and model artifact. Data versioning may use immutable snapshots, manifests, table time travel or content-addressed files. Avoid copying sensitive data into developer machines for reproducibility; use approved access and synthetic or minimized samples where possible.

Model lineage answers: which data and code produced this model, which model served this decision, and which source records influenced a response? This matters for debugging, rollback, compliance and deletion requests.

## 9. Streaming and message delivery basics

Queues decouple producers from consumers. Delivery semantics can be at-most-once, at-least-once or effectively-once with idempotent processing and deduplication. Exactly-once claims depend on system boundaries and usually require careful transactional design.

An ingestion worker should:
- validate message schema;
- handle duplicate IDs idempotently;
- retry transient errors with bounds;
- route poison messages to a dead-letter path;
- expose lag and failure metrics;
- preserve ordering only where the domain needs it.

## Questions and answers

**Q: Is more training data always better?**  
A: No. More mislabeled, duplicated, biased or out-of-scope data can worsen results. Quality, coverage and labeling relevance matter.

**Q: What is training-serving skew?**  
A: The same logical feature is computed differently during training and production, so the model sees a different representation at inference time.

**Q: Should I store a training dataset in Git?**  
A: Usually not if it is large, sensitive or changing. Store code/manifests and use approved data versioning/storage with access controls and provenance.

**Q: Why not train directly from the production database?**  
A: It can burden transactional workloads, make results non-reproducible, expose sensitive data and mix changing records. Use reviewed snapshots or analytics infrastructure.

## Practice

1. Define a data contract for a ticket-priority training table, including label timestamp and null semantics.
2. Identify fields that would leak future knowledge into prediction.
3. Design quality checks for duplicate IDs, missing text and out-of-range timestamps.
4. Explain how a late event should affect a daily feature aggregate.
5. Describe how to reproduce a dataset used by a model six months later.

## Completion check

You can model a data pipeline, describe OLTP versus analytics systems, write a safe SQL query, define quality checks, avoid future leakage, and trace data lineage.