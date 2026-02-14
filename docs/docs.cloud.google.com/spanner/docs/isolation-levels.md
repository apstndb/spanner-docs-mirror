**Preview — Repeatable read isolation**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

This page introduces different isolation levels and explains how they work in Spanner.

*Isolation level* is a database property that defines which data is visible to concurrent transactions. Spanner supports two of the isolation levels defined in the ANSI/ISO SQL standard: *serializable* and *repeatable read* . When you create a transaction, you need to choose the most appropriate isolation level for the transaction. The chosen isolation level lets individual transactions prioritize various factors such as latency, abort rate, and whether the application is susceptible to the effects of data anomalies. The best choice depends on the specific demands of the workload.

## Serializable isolation

Serializable isolation is the default isolation level in Spanner. Under serializable isolation, Spanner provides you with the strictest concurrency-control guarantees for transactions, which is called [external consistency](/spanner/docs/true-time-external-consistency) . Spanner behaves as if all transactions were executed sequentially, even though Spanner actually runs them across multiple servers (and possibly in multiple datacenters) for higher performance and availability than single server databases. In addition, if one transaction completes before another transaction starts to commit, Spanner guarantees that clients always see the results of transactions in sequential order. Intuitively, Spanner is similar to a single-machine database.

The trade-off is that Spanner might abort transactions if a workload has high read-write contention, where many transactions read data that another transaction is updating, due to the fundamental nature of serializable transactions. However, this is a good default for an operational database. It helps you to avoid tricky timing issues that usually only arise with high concurrency. These issues are difficult to reproduce and troubleshoot. Therefore, serializable isolation provides the strongest protection against data anomalies. If a transaction needs to be retried, there might be an increase in latency due to transaction retries.

## Repeatable read isolation

In Spanner, repeatable read isolation is implemented using a technique commonly known as snapshot isolation. Repeatable read isolation in Spanner ensures that all read operations within a transaction see a consistent, or strong, snapshot of the database as it existed at the start of the transaction. It also guarantees that concurrent writes on the same data only succeed if there are no conflicts. This approach is beneficial in high read-write conflict scenarios where numerous transactions read data that other transactions might be modifying. By using a fixed snapshot, repeatable read avoids the performance impacts of the more restrictive serializable isolation level. Reads can execute without acquiring locks and without blocking concurrent writes, which results in fewer aborted transactions that might need to be retried due to potential serialization conflicts. In use cases where your clients already run everything in a read-write transaction, and it is difficult to redesign and use read-only transactions, you can use repeatable read isolation to improve the latency of your workloads.

Unlike serializable isolation, repeatable read might lead to data anomalies if your application relies on specific data relationships or constraints that aren't enforced by the database schema, especially when the order of operations matter. In such cases, a transaction might read data, make decisions based on that data, and then write changes that violate those application-specific constraints, even if the database schema constraints are still met. This happens because repeatable read isolation allows concurrent transactions to proceed without strict serialization. One potential anomaly is known as a *write skew* , which arises from a particular kind of concurrent update, where each update is independently accepted, but their combined effect violates application data integrity. For example, imagine there's a hospital system where at least one doctor needs to be on-call at all times, and doctors can request to be taken off-call for a shift. Under repeatable read isolation, if both Dr. Richards and Dr. Smith are scheduled to be on-call for the same shift and concurrently try to request to be taken off-call, each request succeeds in parallel. This is because both transactions read that there is at least one other doctor who is scheduled to be on-call at the start of the transaction, causing data anomaly if the transactions succeed. On the other hand, using serializable isolation prevents these transactions from violating the constraint because serializable transactions will detect potential data anomalies and abort the transaction. Thereby ensuring application consistency by accepting higher abort rates.

In the previous example, you can [use the `  SELECT FOR UPDATE  ` clause in repeatable read isolation](/spanner/docs/use-select-for-update-repeatable-read) . The `  SELECT…FOR UPDATE  ` clause verifies if the data it read at the chosen snapshot remains unchanged at commit time. Similarly, [DML statements](/spanner/docs/dml-tasks) and [mutations](/spanner/docs/modify-mutation-api) , that read data internally to ensure the integrity of the writes, also verify that the data remains unchanged at commit time.

For more information, see [Use repeatable read isolation](/spanner/docs/use-repeatable-read-isolation) .

### Example use case

The following example demonstrates the benefit of using repeatable read isolation to eliminate locking overhead. Both `  Transaction 1  ` and `  Transaction 2  ` run in repeatable read isolation.

`  Transaction 1  ` establishes a snapshot timestamp when the `  SELECT  ` statement runs.

### GoogleSQL

``` text
-- Transaction 1
BEGIN;

-- Snapshot established at T1
SELECT AlbumId, MarketingBudget
FROM Albums
WHERE SingerId = 1;

/*-----------+------------------*
| AlbumId    | MarketingBudget  |
+------------+------------------+
| 1          | 50000            |
| 2          | 100000           |
| 3          | 70000            |
| 4          | 80000            |
*------------+------------------*/
```

### PostgreSQL

``` text
-- Transaction 1
BEGIN;

-- Snapshot established at T1
SELECT albumid, marketingbudget
FROM albums
WHERE singerid = 1;

/*-----------+------------------*
| albumid    | marketingbudget  |
+------------+------------------+
| 1          | 50000            |
| 2          | 100000           |
| 3          | 70000            |
| 4          | 80000            |
*------------+------------------*/
```

Then, `  Transaction 2  ` establishes a snapshot timestamp after `  Transaction 1  ` begins but before it commits. Since `  Transaction 1  ` hasn't updated the data, the `  SELECT  ` query in `  Transaction 2  ` reads the same data as `  Transaction 1  ` .

### GoogleSQL

``` text
-- Transaction 2
BEGIN;

-- Snapshot established at T2 > T1
SELECT AlbumId, MarketingBudget
FROM Albums
WHERE SingerId = 1;

INSERT INTO Albums (SingerId, AlbumId, MarketingBudget) VALUES (1, 5, 50000);

COMMIT;
```

### PostgreSQL

``` text
-- Transaction 2
BEGIN;

-- Snapshot established at T2 > T1
SELECT albumid, marketingbudget
FROM albums
WHERE singerid = 1;

INSERT INTO albums (singerid, albumid, marketingbudget) VALUES (1, 5, 50000);

COMMIT;
```

`  Transaction 1  ` continues after `  Transaction 2  ` has committed.

### GoogleSQL

``` text
-- Transaction 1 continues
SELECT SUM(MarketingBudget) as UsedBudget
FROM Albums
WHERE SingerId = 1;

/*-----------*
| UsedBudget |
+------------+
| 300000     |
*------------*/
```

### PostgreSQL

``` text
-- Transaction 1 continues
SELECT SUM(marketingbudget) AS usedbudget
FROM albums
WHERE singerid = 1;

/*-----------*
| usedbudget |
+------------+
| 300000     |
*------------*/
```

The `  UsedBudget  ` value that Spanner returns is the sum of the budget read by `  Transaction 1  ` . This sum reflects only the data present at the `  T1  ` snapshot. It doesn't include the budget that `  Transaction 2  ` added, because `  Transaction 2  ` committed after `  Transaction 1  ` established snapshot `  T1  ` . Using repeatable read means that `  Transaction 1  ` didn't have to abort even though `  Transaction 2  ` modified the data read by `  Transaction 1  ` . However, the result Spanner returns might or might not be the intended outcome.

### Read-write conflicts and correctness

In the previous example, if the data queried by the `  SELECT  ` statements in `  Transaction 1  ` was used to make subsequent marketing budget decisions, there might be correctness issues.

For example, assume there is a total budget of `  400,000  ` . Based on the result from the `  SELECT  ` statement in `  Transaction 1  ` , we might think there is `  100,000  ` left in the budget and decide to allocate it all to `  AlbumId = 4  ` .

### GoogleSQL

``` text
-- Transaction 1 continues..
UPDATE Albums
SET MarketingBudget = MarketingBudget + 100000
WHERE SingerId = 1 AND AlbumId = 4;

COMMIT;
```

### PostgreSQL

``` text
-- Transaction 1 continues..
UPDATE albums
SET marketingbudget = marketingbudget + 100000
WHERE singerid = 1 AND albumid = 4;

COMMIT;
```

`  Transaction 1  ` commits successfully, even though `  Transaction 2  ` already allocated `  50,000  ` of the remaining `  100,000  ` budget to a new album `  AlbumId = 5  ` .

You can use the `  SELECT...FOR UPDATE  ` syntax to validate that certain reads of a transaction are unchanged during the lifetime of the transaction in order to guarantee the correctness of the transaction. In the following example using `  SELECT...FOR UPDATE  ` , `  Transaction 1  ` aborts at commit time.

### GoogleSQL

``` text
-- Transaction 1 continues..
SELECT SUM(MarketingBudget) AS TotalBudget
FROM Albums
WHERE SingerId = 1
FOR UPDATE;

/*-----------*
| TotalBudget |
+------------+
| 300000     |
*------------*/

COMMIT;
```

### PostgreSQL

``` text
-- Transaction 1 continues..
SELECT SUM(marketingbudget) AS totalbudget
FROM albums
WHERE singerid = 1
FOR UPDATE;

/*-------------*
 | totalbudget |
 +-------------+
 | 300000      |
 *-------------*/

COMMIT;
```

For more information, see [Use SELECT FOR UPDATE in repeatable read isolation](/spanner/docs/use-select-for-update-repeatable-read) .

### Write-write conflicts and correctness

By using repeatable read isolation level, concurrent writes on the same data only succeed if there are no conflicts.

In the following example, `  Transaction 1  ` establishes a snapshot timestamp at the first `  SELECT  ` statement.

### GoogleSQL

``` text
-- Transaction 1
BEGIN;

-- Snapshot established at T1
SELECT AlbumId, MarketingBudget
FROM Albums
WHERE SingerId = 1;
```

### PostgreSQL

``` text
-- Transaction 1
BEGIN;

-- Snapshot established at T1
SELECT albumid, marketingbudget
FROM albums
WHERE singerid = 1;
```

The following `  Transaction 2  ` reads the same data as `  Transaction 1  ` and inserts a new item. `  Transaction 2  ` successfully commits without waiting or aborting.

### GoogleSQL

``` text
-- Transaction 2
BEGIN;

-- Snapshot established at T2 (> T1)
SELECT AlbumId, MarketingBudget
FROM Albums
WHERE SingerId = 1;

INSERT INTO Albums (SingerId, AlbumId, MarketingBudget) VALUES (1, 5, 50000);

COMMIT;
```

### PostgreSQL

``` text
-- Transaction 2
BEGIN;

-- Snapshot established at T2 (> T1)
SELECT albumid, marketingbudget
FROM albums
WHERE singerid = 1;

INSERT INTO albums (singerid, albumid, marketingbudget) VALUES (1, 5, 50000);

COMMIT;
```

`  Transaction 1  ` continues after `  Transaction 2  ` has committed.

### GoogleSQL

``` text
-- Transaction 1 continues
INSERT INTO Albums (SingerId, AlbumId, MarketingBudget) VALUES (1, 5, 30000);
-- Transaction aborts
COMMIT;
```

### PostgreSQL

``` text
-- Transaction 1 continues
INSERT INTO albums (singerid, albumid, marketingbudget) VALUES (1, 5, 30000);
-- Transaction aborts
COMMIT;
```

`  Transaction 1  ` aborts since `  Transaction 2  ` already committed an insertion to the `  AlbumId = 5  ` row.

## What's next

  - Learn how to [Use repeatable read isolation level](/spanner/docs/use-repeatable-read-isolation) .

  - Learn how to [Use SELECT FOR UPDATE in repeatable read isolation](/spanner/docs/use-select-for-update-repeatable-read) .

  - Learn how to [Use SELECT FOR UPDATE in serializable isolation](/spanner/docs/use-select-for-update-serializable) .

  - Learn more about Spanner serializability and external consistency, see [TrueTime and external consistency](/spanner/docs/true-time-external-consistency) .
