---
name: documents/docs.cloud.google.com/spanner/docs/use-select-for-update-repeatable-read
uri: https://docs.cloud.google.com/spanner/docs/use-select-for-update-repeatable-read
title: Use SELECT FOR UPDATE in repeatable read isolation
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

This page describes how to use the `FOR UPDATE` clause in [repeatable read isolation](https://docs.cloud.google.com/spanner/docs/isolation-levels#repeatable-read) .

In repeatable read isolation, whether the `FOR UPDATE` clause acquires locks depends on the concurrency control mode. For more information, see [Why use the `FOR UPDATE` clause](https://docs.cloud.google.com/spanner/docs/use-select-for-update-repeatable-read#why-use-for-update) .

> **Note:** Under [pessimistic concurrency control](https://docs.cloud.google.com/spanner/docs/concurrency-control#pessimistic_concurrency_control) in repeatable read isolation, `FOR UPDATE` acquires exclusive locks on scanned rows and cells during reads, which is similar to how `FOR UPDATE` behaves in many other databases. If you're migrating an application from another database, repeatable read isolation with pessimistic concurrency control is the option that's most semantically similar to the behavior that your application expects. Concurrent transactions that access the locked data wait until the lock is released at commit or rollback.

For more information about locks in `FOR UPDATE` , see [Use `SELECT FOR UPDATE` in serializable isolation](https://docs.cloud.google.com/spanner/docs/use-select-for-update-serializable) .

To learn how to use the `FOR UPDATE` clause, see the [GoogleSQL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/query-syntax#for_update_clause) and [PostgreSQL](https://docs.cloud.google.com/spanner/docs/reference/postgresql/query-syntax#for_update_clause) `FOR UPDATE` reference guides.

## Why use the `FOR UPDATE` clause

When a transaction runs with repeatable read isolation, the data queried by the `SELECT` statement is always returned at the established snapshot timestamp for the transaction. If the transaction then makes updates based on the data that is queried, there might be correctness issues if a concurrent transaction also updates the queried data. For more information, see [Read-write conflicts and correctness](https://docs.cloud.google.com/spanner/docs/isolation-levels#read-write-conflicts-correctness) .

To ensure that queried data remains valid at commit time, you can use the `FOR UPDATE` clause with repeatable read isolation. Using `FOR UPDATE` prevents read-write conflicts where another transaction updates the data after it is read:

- With [optimistic concurrency control](https://docs.cloud.google.com/spanner/docs/concurrency-control#optimistic_concurrency_control) (the default), Spanner validates that scanned ranges have not been modified before committing. If a conflict is detected, Spanner aborts the transaction.
- With [pessimistic concurrency control](https://docs.cloud.google.com/spanner/docs/concurrency-control#pessimistic_concurrency_control) , Spanner acquires exclusive locks during read execution, preventing concurrent transactions from modifying the locked data until the transaction commits.

## Query syntax

This section provides guidance on query syntax when using the `FOR UPDATE` clause.

The most common usage is in a top-level `SELECT` statement. For example:

```
SELECT SingerId, SingerInfo
FROM Singers WHERE SingerID = 5
FOR UPDATE;
```

The `FOR UPDATE` clause ensures that the data queried by the `SELECT` statement and `SingerID = 5` is still valid when the transaction commits, preventing correctness issues that might arise if a concurrent transaction updates the queried data.

### Use in WITH statements

The `FOR UPDATE` clause doesn't verify the ranges scanned within the `WITH` statement when you specify `FOR UPDATE` in the outer-level query of the `WITH` statement.

In the following query, no scanned ranges are validated because the `FOR UPDATE` isn't propagated to the common table expressions (CTE) query.

```
WITH s AS (SELECT SingerId, SingerInfo FROM Singers WHERE SingerID > 5)
SELECT * FROM s
FOR UPDATE;
```

If the `FOR UPDATE` clause is specified in the CTE query, then the scanned range of the CTE query is validated.

In the following example, the `SingerId` and `SingerInfo` cells for the rows where `SingerId > 5` are validated.

```
WITH s AS
  (SELECT SingerId, SingerInfo FROM Singers WHERE SingerId > 5 FOR UPDATE)
SELECT * FROM s;
```

### Use in subqueries

You can use the `FOR UPDATE` clause in an outer-level query that has one or more subqueries. Ranges scanned by the top-level query and within subqueries are validated, except in [expression subqueries](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/subqueries#expression_subquery_concepts) .

The following query validates the `SingerId` and `SingerInfo` cells for rows where `SingerId > 5.`

```
(SELECT SingerId, SingerInfo FROM Singers WHERE SingerId > 5) AS t
FOR UPDATE;
```

The following query doesn't validate any cells in the `Albums` table because it's within an expression subquery. The `SingerId` and `SingerInfo` cells for the rows returned by the expression subquery are validated.

```
SELECT SingerId, SingerInfo
FROM Singers
WHERE SingerId = (SELECT SingerId FROM Albums WHERE MarketingBudget > 100000)
FOR UPDATE;
```

### Use to query views

You can use the `FOR UPDATE` clause to query a view as shown in the following example:

```
CREATE VIEW SingerBio AS SELECT SingerId, FullName, SingerInfo FROM Singers;

SELECT * FROM SingerBio WHERE SingerId = 5 FOR UPDATE;
```

You can't use the `FOR UPDATE` clause [when defining a view](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#create-view) .

## Unsupported use cases

The following `FOR UPDATE` use cases are unsupported:

- **As a mutual exclusion mechanism for running code outside of Spanner:** Don't use locking in Spanner to ensure exclusive access to a resource outside of Spanner. Spanner might abort and retry a transaction, whether explicitly through application code or implicitly through client code such as the [Spanner JDBC driver](https://docs.cloud.google.com/spanner/docs/use-oss-jdbc) . For a transaction attempt that successfully commits, Spanner holds the locks for the entire duration of the attempt, up to the commit timestamp. For an attempt that aborts, Spanner releases the locks when it aborts the transaction, which can be before your client receives the abort. As a result, code outside of Spanner that relies on the locks might still be running after Spanner releases them.
- **In combination with the [`LOCK_SCANNED_RANGES` hint](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/query-syntax#hints) :** You can't use both the `FOR UPDATE` clause and the `LOCK_SCANNED_RANGES` hint in the same query, or Spanner returns an error. For more information, see [Comparison to the `LOCK_SCANNED_RANGES` hint](https://docs.cloud.google.com/spanner/docs/use-select-for-update-serializable#comparison) .
- **In full-text search queries:** You can't use the `FOR UPDATE` clause in queries using [full-text search](https://docs.cloud.google.com/spanner/docs/full-text-search) indexes.
- **In read-only transactions:** The `FOR UPDATE` clause is only valid in queries executing within read-write transactions.
- **Within DDL statements:** You can't use the `FOR UPDATE` clause in queries within DDL statements, which are stored for later execution. For example, you can't use the `FOR UPDATE` clause [when defining a view](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#create-view) .

## What's Next

- Learn how to use the `FOR UPDATE` clause in [GoogleSQL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/query-syntax#for_update_clause) and [PostgreSQL](https://docs.cloud.google.com/spanner/docs/reference/postgresql/query-syntax#for_update_clause) .
- Learn how to [use `SELECT FOR UPDATE` in serializable isolation](https://docs.cloud.google.com/spanner/docs/use-select-for-update-serializable) .
- Learn about the [`LOCK_SCANNED_RANGES` hint](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/query-syntax#hints) .
- Learn about [Locking](https://docs.cloud.google.com/spanner/docs/transactions#locking) in Spanner.
