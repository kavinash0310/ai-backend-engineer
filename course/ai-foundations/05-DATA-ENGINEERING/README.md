# Data Engineering for AI

A model learns only from the data pipeline it receives. Data engineering covers source contracts, cleaning, validation, transformation, lineage, quality, storage and repeatability.

## Required sequence

- Advanced SQL, indexes, constraints and transactions (reuse [CS database/SQL notes](https://github.com/kavinash0310/cs-fundamentals/blob/main/subjects/database-systems-sql.md)).
- NumPy/Pandas and analysis; handle missingness, duplicates, types, outliers and class distribution.
- Train/validation/test split before fitting preprocessing; preserve temporal/group boundaries.
- ETL/ELT, batch/stream pipelines, schema evolution, data quality and reproducibility.
- Feature and document pipelines, metadata, provenance, deletion and access controls.

**Completion evidence:** a versioned data card and pipeline that can be rerun, validates inputs, records transformations, and proves no test leakage. RAG ingestion reuses many of these principles.


## Detailed lesson

- [Data engineering for AI](01-data-engineering-for-ai.md) — SQL concepts, OLTP/analytics, contracts, quality, leakage, lineage and streaming.
