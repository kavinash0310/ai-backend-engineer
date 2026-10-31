# PostgreSQL, SQL, and transactions

## Relational model from first principles

A relational database stores related facts in tables. A row is one record; a column is a field with a defined type. A primary key identifies a row. A foreign key links a row to another table and can prevent references to missing records. Constraints express rules such as required values, uniqueness, and valid ranges.

A simple AI application might have users, conversations, messages, documents, and jobs. Avoid storing every concept in one giant JSON field. Separate entities with clear ownership and use foreign keys. Store flexible provider metadata only when it truly varies; keep important searchable fields typed.

SQL retrieves and changes records. A parameterized query treats user input as data rather than executable SQL, helping prevent injection. Never build SQL by concatenating untrusted strings.

~~~sql
SELECT id, status, created_at
FROM jobs
WHERE owner_id = $1
ORDER BY created_at DESC
LIMIT $2;
~~~

The owner filter is part of authorization, not merely a convenience. A user-supplied object ID must not grant access to another user's record.

## Transactions and ACID

A transaction groups database operations into a unit. ACID describes atomicity, consistency, isolation, and durability. If creating a conversation and its first message must happen together, a transaction prevents one from persisting without the other.

Isolation controls how concurrent transactions observe each other's work. The default database behavior may not prevent every race, so invariants should be protected with constraints, locks, or appropriate isolation. Do not hold a transaction open while waiting for a slow LLM response; that consumes connections and locks for too long. Save job state, release the transaction, perform external work, then update status in a new transaction.

## Indexes and migrations

An index is an auxiliary structure that helps find rows faster, at the cost of storage and slower writes. Index columns used in common filters, joins, or ordering. Inspect query plans instead of indexing every column. A composite index order matters: an index on (owner_id, created_at) can support common owner-specific time queries.

A migration is a versioned schema change. Apply migrations in a controlled deploy step, design changes compatible with running application versions where possible, and backfill large tables incrementally. Back up data and know how to recover. A schema change can fail even when application code is correct.

Connection pools reuse database connections. Set pool sizes with server limits and expected concurrency in mind. An async application should use a database driver compatible with its execution model. Clean up sessions and transactions in every path.

## Worked request

To create an AI job, insert a job row with status queued and an owner ID in a short transaction. Enqueue its identifier. A worker claims the job, checks its current state, loads the authorized input, calls the model outside a database transaction, and writes result/status in a second transaction. A unique idempotency key or job constraint can prevent duplicate submissions.

Questions: Why not keep a database transaction open during model generation? The model can be slow and the transaction holds scarce resources. Why use a foreign key? It enforces the relationship even if application code has a bug. Why inspect query plans? Indexes have costs and queries may not use them.

Practice: design tables for users, documents, and asynchronous processing jobs. Add owner references, unique IDs, status constraints, and indexes for listing a user's recent jobs. Write a query that always filters by the authenticated owner.

Further reading: [PostgreSQL tutorial](https://www.postgresql.org/docs/current/tutorial.html) and [PostgreSQL application development](https://www.postgresql.org/docs/current/appdev.html).
