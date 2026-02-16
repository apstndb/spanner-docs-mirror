This page describes how to use the `  FOR UPDATE  ` clause in [repeatable read isolation](/spanner/docs/isolation-levels#repeatable-read) .

The locking mechanism of the `  FOR UPDATE  ` clause is different for repeatable read and serializable isolation. Unlike in serializable isolation, the `  FOR UPDATE  ` clause doesn't acquire locks in repeatable read isolation. For more information about locks in `  FOR UPDATE  ` , see [Use SELECT FOR UPDATE in serializable isolation](/spanner/docs/use-select-for-update-serializable) .

To learn how to use the `  FOR UPDATE  ` clause, see the [GoogleSQL](/spanner/docs/reference/standard-sql/query-syntax#for_update_clause) and [PostgreSQL](/spanner/docs/reference/postgresql/query-syntax#for_update_clause) `  FOR UPDATE  ` reference guides.

## Why use the `     FOR UPDATE    ` clause

When a transaction runs with repeatable read isolation, the data queried by the `  SELECT  ` statement is always returned at the established snapshot timestamp for the transaction. If the transaction then makes updates based on the data that is queried, there might be correctness issues if a concurrent transaction also updates the queried data. For more information, see [Read-write conflicts and correctness](/spanner/docs/isolation-levels#read-write-conflicts-correctness) .

To ensure that data queried by the `  SELECT  ` statement is still valid when the transaction commits, you can use a `  FOR UPDATE  ` clause with repeatable read isolation. Using `  FOR UPDATE  ` guarantees transaction correctness despite read-write conflicts where data might have been modified by another transaction between the time it was read and modified.

## Query syntax

This section provides guidance on query syntax when using the `  FOR UPDATE  ` clause.

The most common usage is in a top-level `  SELECT  ` statement. For example:

``` text
SELECT SingerId, SingerInfo
FROM Singers WHERE SingerID = 5
FOR UPDATE;
```

The `  FOR UPDATE  ` clause ensures that the data queried by the `  SELECT  ` statement and `  SingerID = 5  ` is still valid when the transaction commits, preventing correctness issues that might arise if a concurrent transaction updates the queried data.

### Use in WITH statements

The `  FOR UPDATE  ` clause doesn't verify the ranges scanned within the `  WITH  ` statement when you specify `  FOR UPDATE  ` in the outer-level query of the `  WITH  ` statement.

In the following query, no scanned ranges are validated because the `  FOR UPDATE  ` isn't propagated to the common table expressions (CTE) query.

``` text
WITH s AS (SELECT SingerId, SingerInfo FROM Singers WHERE SingerID > 5)
SELECT * FROM s
FOR UPDATE;
```

If the `  FOR UPDATE  ` clause is specified in the CTE query, then the scanned range of the CTE query is validated.

In the following example, the `  SingerId  ` and `  SingerInfo  ` cells for the rows where `  SingerId > 5  ` are validated.

``` text
WITH s AS
  (SELECT SingerId, SingerInfo FROM Singers WHERE SingerId > 5 FOR UPDATE)
SELECT * FROM s;
```

### Use in subqueries

You can use the `  FOR UPDATE  ` clause in an outer-level query that has one or more subqueries. Ranges scanned by the top-level query and within subqueries are validated, except in [expression subqueries](/spanner/docs/reference/standard-sql/subqueries#expression_subquery_concepts) .

The following query validates the `  SingerId  ` and `  SingerInfo  ` cells for rows where `  SingerId > 5.  `

``` text
(SELECT SingerId, SingerInfo FROM Singers WHERE SingerId > 5) AS t
FOR UPDATE;
```

The following query doesn't validate any cells in the `  Albums  ` table because it's within an expression subquery. The `  SingerId  ` and `  SingerInfo  ` cells for the rows returned by the expression subquery are validated.

``` text
SELECT SingerId, SingerInfo
FROM Singers
WHERE SingerId = (SELECT SingerId FROM Albums WHERE MarketingBudget > 100000)
FOR UPDATE;
```

### Use to query views

You can use the `  FOR UPDATE  ` clause to query a view as shown in the following example:

``` text
CREATE VIEW SingerBio AS SELECT SingerId, FullName, SingerInfo FROM Singers;

SELECT * FROM SingerBio WHERE SingerId = 5 FOR UPDATE;
```

You can't use the `  FOR UPDATE  ` clause [when defining a view](/spanner/docs/reference/standard-sql/data-definition-language#create-view) .

## Unsupported use cases

The following `  FOR UPDATE  ` use cases are unsupported:

  - **As a mutual exclusion mechanism for running code outside of Spanner:** Don't use locking in Spanner to ensure exclusive access to a resource outside of Spanner. Transactions might be aborted by Spanner, for example, if a transaction is retried, whether explicitly by application code or implicitly by client code, such as the [Spanner JDBC driver](/spanner/docs/use-oss-jdbc) , it's only guaranteed that the locks are held during the attempt that was committed.
  - **In combination with the [`  LOCK_SCANNED_RANGES  ` hint](/spanner/docs/reference/standard-sql/query-syntax#hints) :** You can't use both the `  FOR UPDATE  ` clause and the `  LOCK_SCANNED_RANGES  ` hint in the same query, or Spanner returns an error. For more information, see [Comparison to the `  LOCK_SCANNED_RANGES  ` hint](/spanner/docs/use-select-for-update-serializable#comparison) .
  - **In full-text search queries:** You can't use the `  FOR UPDATE  ` clause in queries using [full-text search](/spanner/docs/full-text-search) indexes.
  - **In read-only transactions:** The `  FOR UPDATE  ` clause is only valid in queries executing within read-write transactions.
  - **Within DDL statements:** You can't use the `  FOR UPDATE  ` clause in queries within DDL statements, which are stored for later execution. For example, you can't use the `  FOR UPDATE  ` clause [when defining a view](/spanner/docs/reference/standard-sql/data-definition-language#create-view) .

## What's Next

  - Learn how to use the `  FOR UPDATE  ` clause in [GoogleSQL](/spanner/docs/reference/standard-sql/query-syntax#for_update_clause) and [PostgreSQL](/spanner/docs/reference/postgresql/query-syntax#for_update_clause) .
  - Learn how to [Use SELECT FOR UPDATE in serializable isolation](/spanner/docs/use-select-for-update-serializable) .
  - Learn about the [`  LOCK_SCANNED_RANGES  ` hint](/spanner/docs/reference/standard-sql/query-syntax#hints) .
  - Learn about [Locking](/spanner/docs/transactions#locking) in Spanner.
