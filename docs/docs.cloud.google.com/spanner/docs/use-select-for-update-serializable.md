**Important:** Don't use locks to ensure exclusive access to a resource outside of Spanner. For more information, see [Unsupported use cases](#unsupported-use-cases) .

This page describes how to use the `  FOR UPDATE  ` clause in serializable isolation.

The locking mechanism of the `  FOR UPDATE  ` clause is different for [repeatable read](/spanner/docs/isolation-levels#repeatable-read) and [serializable isolation](/spanner/docs/isolation-levels#serializable) . Using serializable isolation, when you use the `  SELECT  ` query to scan a table, adding a `  FOR UPDATE  ` clause enables exclusive locks at the intersection of the row-and-column granularity level, otherwise known as cell-level. The lock remains in place for the lifetime of the read-write transaction. During this time, the `  FOR UPDATE  ` clause prevents other transactions from modifying the locked cells until the current transaction completes.

To learn how to use the `  FOR UPDATE  ` clause, see the [GoogleSQL](/spanner/docs/reference/standard-sql/query-syntax#for_update_clause) and [PostgreSQL](/spanner/docs/reference/postgresql/query-syntax#for_update_clause) `  FOR UPDATE  ` reference guides.

## Why use the `     FOR UPDATE    ` clause

In databases with less strict isolation levels, the `  FOR UPDATE  ` clause might be necessary to ensure that a concurrent transaction doesn't update data between reading the data and committing the transaction. Since Spanner enforces serializability by default, it's guaranteed that the transaction only commits successfully if the data accessed within the transaction isn't stale at commit time. Therefore, the `  FOR UPDATE  ` clause isn't necessary to ensure transaction correctness in Spanner.

However, in use cases with high write contention, such as when multiple transactions are concurrently reading and writing to the same data, the simultaneous transactions might cause an increase in aborts. This is because when multiple, simultaneous transactions acquire shared locks, and then try to upgrade to exclusive locks, the transactions cause a deadlock. The deadlock permanently blocks the transactions because each is waiting for the other to release the resource that it needs. In order to make progress, Spanner aborts all but one of the transactions to resolve the deadlock. For more information, see [Locking](/spanner/docs/transactions#locking) .

A transaction that uses the `  FOR UPDATE  ` clause acquires the exclusive lock proactively and proceeds to execute, while other transactions wait their turn for the lock. Although Spanner might still limit throughput because the conflicting transactions can only be performed one at a time, but because Spanner is only making progress on one transaction, it saves time that would otherwise be spent aborting and retrying transactions.

Therefore, if reducing the number of aborted transactions in a simultaneous write request scenario is important, then you can use the `  FOR UPDATE  ` clause to reduce the overall number of aborts and increase workload execution efficiency.

## Comparison to the `     LOCK_SCANNED_RANGES    ` hint

The `  FOR UPDATE  ` clause serves a similar function as the [`  LOCK_SCANNED_RANGES=exclusive  `](/spanner/docs/reference/standard-sql/query-syntax#statement_hints) hint.

There are two key differences:

  - If you use the `  LOCK_SCANNED_RANGES  ` hint, the transaction acquires exclusive locks on the scanned ranges for the entire statement. You can't acquire exclusive locks on a subquery. Using the lock hint might result in acquiring more locks than necessary and contributing to lock contention in the workload. The following example shows how to use a lock hint:
    
    ``` text
    @{lock_scanned_ranges=exclusive}
    SELECT s.SingerId, s.FullName FROM Singers AS s
    JOIN (SELECT SingerId FROM Albums WHERE MarketingBudget > 100000)
    AS a ON a.SingerId = s.SingerId;
    ```
    
    On the other hand, you can use the `  FOR UPDATE  ` clause in a subquery as shown in the following example:
    
    ``` text
    SELECT s.SingerId, s.FullName FROM Singers AS s
    JOIN (SELECT SingerId FROM Albums WHERE MarketingBudget > 100000)
    FOR UPDATE AS a ON a.SingerId = s.SingerId;
    ```

  - You can use the `  LOCK_SCANNED_RANGES  ` hint in DML statements whereas you can only use the `  FOR UPDATE  ` clause in `  SELECT  ` statements.

## Lock semantics

To reduce simultaneous write requests and the cost of transactions being aborted as a result of deadlock, Spanner locks data at the cell-level if possible. The cell-level is the most granular level of data within a table - a data point at the intersection of a row and a column. When using the `  FOR UPDATE  ` clause, Spanner locks specific cells that are scanned by the `  SELECT  ` query.

In the following example, the `  MarketingBudget  ` cell in the `  SingerId = 1  ` and `  AlbumId = 1  ` row is exclusively locked in the `  Albums  ` table, preventing concurrent transactions from modifying that cell until this transaction is committed or rolled back. However, concurrent transactions can still update the `  AlbumTitle  ` cell in that row.

``` text
SELECT MarketingBudget
FROM Albums
WHERE SingerId = 1 and AlbumId = 1
FOR UPDATE;
```

### Concurrent transactions might block on reading locked data

When one transaction has acquired exclusive locks on a scanned range, concurrent transactions might block reading that data. Spanner enforces [serializability](/spanner/docs/transactions#serializability_and_external_consistency) so data can only be read if it is guaranteed to be unchanged by another transaction within the lifetime of the transaction. Concurrent transactions that attempt to read already locked data might have to wait until the transaction holding the locks is committed, rolled back, or times out.

In the following example, `  Transaction 1  ` locks the `  MarketingBudget  ` cells for `  1 <= AlbumId < 5  ` .

``` text
-- Transaction 1
SELECT MarketingBudget
FROM Albums
WHERE SingerId = 1 and AlbumId >= 1 and AlbumId < 5
FOR UPDATE;
```

`  Transaction 2  ` , which is attempting to read the `  MarketingBudget  ` for `  AlbumId = 1  ` , is blocked until `  Transaction 1  ` either commits or is rolled back.

``` text
-- Transaction 2
SELECT MarketingBudget
FROM Albums
WHERE SingerId = 1 and AlbumId = 1;

-- Blocked by Transaction 1
```

Similarly, a transaction attempting to lock a scanned range with `  FOR UPDATE  ` is blocked by a concurrent transaction that locks an overlapping scanned range.

`  Transaction 3  ` in the following example is also blocked since `  Transaction 1  ` has locked the `  MarketingBudget  ` cells for `  3 <= AlbumId < 5  ` , which is the overlapping scanned range with `  Transaction 3  ` .

``` text
-- Transaction 3
SELECT MarketingBudget
FROM Albums
WHERE SingerId = 1 and AlbumId >= 3 and AlbumId < 10
FOR UPDATE;

-- Blocked by Transaction 1
```

#### Read an index

A concurrent read might not be blocked if the query that locked the scanned range locks the rows in the base table, but the concurrent transaction reads from an index.

The following `  Transaction 1  ` locks the `  SingerId  ` and `  SingerInfo  ` cells for `  SingerId = 1  ` .

``` text
-- Transaction 1
SELECT SingerId, SingerInfo
FROM Singers
WHERE SingerId = 1
FOR UPDATE;
```

The read-only `  Transaction 2  ` isn't blocked by the locks acquired in `  Transaction 1  ` , because it queries an index table.

``` text
-- Transaction 2
SELECT SingerId FROM Singers;
```

### Concurrent transactions doesn't block DML operations on already locked data

When one transaction has acquired locks on a range of cells with an exclusive lock hint, concurrent transactions attempting to perform a write without reading the data first on the locked cells can proceed. The transaction blocks on the commit until the transaction holding the locks commits or rolls back.

The following `  Transaction 1  ` locks the `  MarketingBudget  ` cells for `  1 <= AlbumId < 5  ` .

``` text
-- Transaction 1
SELECT MarketingBudget
FROM Albums
WHERE SingerId = 1 and AlbumId >= 1 and AlbumId < 5
FOR UPDATE;
```

If `  Transaction 2  ` attempts to update the `  Albums  ` table, it is blocked from doing so until `  Transaction 1  ` commits or rolls back.

``` text
-- Transaction 2
UPDATE Albums
SET MarketingBudget = 200000
WHERE SingerId = 1 and AlbumId = 1;

> Query OK, 1 rows affected

COMMIT;

-- Blocked by Transaction 1
```

### Existing rows and gaps are locked when a scanned range is locked

When one transaction has acquired exclusive locks on a scanned range, concurrent transactions can't insert data in the gaps within that range.

The following `  Transaction 1  ` locks the `  MarketingBudget  ` cells for `  1 <= AlbumId < 10  ` .

``` text
-- Transaction 1
SELECT MarketingBudget
FROM Albums
WHERE SingerId = 1 and AlbumId >= 1 and AlbumId < 10
FOR UPDATE;
```

If `  Transaction 2  ` attempts to insert a row for `  AlbumId = 9  ` that doesn't exist yet, it is blocked from doing so until `  Transaction 1  ` commits or rolls back.

``` text
-- Transaction 2
INSERT INTO Albums (SingerId, AlbumId, AlbumTitle, MarketingBudget)
VALUES (1, 9, "Hello hello!", 10000);

> Query OK, 1 rows affected

COMMIT;

-- Blocked by Transaction 1
```

### Lock acquisition caveats

The lock semantics described provide general guidance but aren't a guarantee on exactly how locks might be acquired when Spanner runs a transaction that uses the `  FOR UPDATE  ` clause. Spanner's query optimization mechanisms might also affect which locks are acquired. The clause prevents other transactions from modifying the locked cells until the current transaction completes.

## Query syntax

This section provides guidance on query syntax when using the `  FOR UPDATE  ` clause.

The most common usage is in a top-level `  SELECT  ` statement. For example:

``` text
SELECT SingerId, SingerInfo
FROM Singers WHERE SingerID = 5
FOR UPDATE;
```

This sample demonstrates how to use the `  FOR UPDATE  ` clause in a `  SELECT  ` statement to exclusively lock the `  SingerId  ` and `  SingerInfo  ` cells of `  WHERE SingerID = 5  ` .

### Use in WITH statements

The `  FOR UPDATE  ` clause doesn't acquire locks for the `  WITH  ` statement when you specify `  FOR UPDATE  ` in the outer-level query of the `  WITH  ` statement.

In the following query, no locks are acquired by the `  Singers  ` table, because the intent to lock isn't propagated to the common table expressions (CTE) query.

``` text
WITH s AS (SELECT SingerId, SingerInfo FROM Singers WHERE SingerID > 5)
SELECT * FROM s
FOR UPDATE;
```

If the `  FOR UPDATE  ` clause is specified in the CTE query, the scanned range of the CTE query acquires the locks.

In the following example, the `  SingerId  ` and `  SingerInfo  ` cells for the rows where `  SingerId > 5  ` are locked.

``` text
WITH s AS
  (SELECT SingerId, SingerInfo FROM Singers WHERE SingerId > 5 FOR UPDATE)
SELECT * FROM s;
```

### Use in subqueries

You can use the `  FOR UPDATE  ` clause in an outer-level query that has one or more subqueries. Locks are acquired by the top-level query and within subqueries, except in [expression subqueries](/spanner/docs/reference/standard-sql/subqueries#expression_subquery_concepts) .

The following query locks the `  SingerId  ` and `  SingerInfo  ` cells for rows where `  SingerId > 5.  `

``` text
(SELECT SingerId, SingerInfo FROM Singers WHERE SingerId > 5) AS t
FOR UPDATE;
```

The following query doesn't lock any cells in the `  Albums  ` table because it is within an expression subquery. The `  SingerId  ` and `  SingerInfo  ` cells for the rows returned by the expression subquery are locked.

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
  - **In combination with the `  LOCK_SCANNED_RANGES  ` hint:** You can't use both the `  FOR UPDATE  ` clause and the `  LOCK_SCANNED_RANGES  ` hint in the same query, or Spanner returns an error.
  - **In full-text search queries:** You can't use the `  FOR UPDATE  ` clause in queries using [full-text search](/spanner/docs/full-text-search) indexes.
  - **In read-only transactions:** The `  FOR UPDATE  ` clause is only valid in queries running within read-write transactions.
  - **Within DDL statements:** You can't use the `  FOR UPDATE  ` clause in queries within DDL statements, which are stored for later execution. For example, you can't use the `  FOR UPDATE  ` clause [when defining a view](/spanner/docs/reference/standard-sql/data-definition-language#create-view) . If locking is required, the `  FOR UPDATE  ` clause can be specified when querying the view.

## What's Next

  - Learn how to use the `  FOR UPDATE  ` clause in [GoogleSQL](/spanner/docs/reference/standard-sql/query-syntax#for_update_clause) and [PostgreSQL](/spanner/docs/reference/postgresql/query-syntax#for_update_clause) .
  - Learn how to [Use SELECT FOR UPDATE in repeatable read isolation](/spanner/docs/use-select-for-update-repeatable-read) .
  - Learn about the [`  LOCK_SCANNED_RANGES  ` hint](/spanner/docs/reference/standard-sql/query-syntax#hints) .
  - Learn about [Locking](/spanner/docs/transactions#locking) in Spanner.
  - Learn about Spanner [serializability](/spanner/docs/transactions#serializability_and_external_consistency) .
