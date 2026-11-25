---
name: documents/docs.cloud.google.com/spanner/docs/queues/queues-in-order
uri: https://docs.cloud.google.com/spanner/docs/queues/queues-in-order
title: Process messages in order per entity
description: Implement design patterns to process messages in order per entity using Spanner queues.
data_source: docs.cloud.google.com
---

[Spanner queues](https://docs.cloud.google.com/spanner/docs/queues/queues-overview) deliver messages at least once and are designed for parallel processing across multiple receivers. When multiple messages for the same entity (such as a customer account, an order, or a device) are sent to a queue, different workers can pull and process them at the same time. Even if one message is sent first, a network delay or retry can slow down the first worker while a second worker finishes a later message.

If every message across the entire queue were processed in strict global order, one slow worker could stall the entire queue (a phenomenon known as head-of-line blocking). However, most applications don't need every message across the entire database to wait in a single global line. Instead, you only need tasks for the *same* entity to run in order, while tasks for different entities run in parallel. You can use the following patterns to process messages in order per entity based on your workload.

## Chain messages for sequential workflows

When a background job for an entity consists of dependent sequential steps (for example, `step-1` followed by `step-2` and `step-3` ), and external requests don't add competing tasks to that workflow in parallel, you want each step to start only after the previous step completes.

To enforce step-by-step ordering, enqueue only the first step initially. When a worker finishes processing the current step, it acknowledges the current message ( `DELETE ...` [`ASSERT_ROWS_MODIFIED`](https://docs.cloud.google.com/spanner/docs/queues/queues-at-most-once#assert_rows_modified) `1` ) and inserts the message for the next step in the same [read-write transaction](https://docs.cloud.google.com/spanner/docs/transactions#read-write_transactions) .

Because the message for the next step isn't created until the current step is completed and acknowledged in the same transaction, two steps for the same workflow never exist in the queue at the same time and can't be processed out of order by concurrent workers.

For example, when a worker finishes `step-1` of an onboarding workflow and schedules `step-2` :

### GoogleSQL

```
-- Inside the worker transaction after finishing step-1:
-- 1. Acknowledge the current step
DELETE FROM OnboardingQueue
WHERE CustomerId = 456 AND StepName = 'step-1'
ASSERT_ROWS_MODIFIED 1;

-- 2. Enqueue the next step in the sequence
INSERT INTO OnboardingQueue (CustomerId, StepName, DeliverTime, Payload)
VALUES (456, 'step-2', CURRENT_TIMESTAMP(), b'{"stage": "provision_resources"}');
```

### PostgreSQL

```
-- Inside the worker transaction after finishing step-1:
-- 1. Acknowledge the current step
DELETE FROM onboardingqueue
WHERE customerid = 456 AND stepname = 'step-1'
ASSERT_ROWS_MODIFIED 1;

-- 2. Enqueue the next step in the sequence
INSERT INTO onboardingqueue (customerid, stepname, deliver_time, payload)
VALUES (456, 'step-2', CURRENT_TIMESTAMP, CAST('{"stage": "provision_resources"}' AS bytea));
```

## Use an ordered work table and a wake-up queue

However, when you need all tasks for a particular entity to run in commit order, even while external requests, webhooks, or user actions add new tasks as earlier tasks are still processing, this pattern supports workloads that fit within the write capacity of a single [split](https://docs.cloud.google.com/spanner/docs/schema-and-data-model#database-splits) .

To process concurrent tasks in order per entity, extend the [dirty flag pattern](https://docs.cloud.google.com/spanner/docs/queues/queues-examples#decouple_data_modification_from_processing_dirty_flag_pattern) by storing ordered tasks in an interleaved table and using the queue only for lightweight wake-up notifications.

First, interleave an ordered work table (using [commit timestamps](https://docs.cloud.google.com/spanner/docs/commit-timestamp) ) and a wake-up queue in a parent entity table that tracks the active claim. To avoid hotspots across splits, choose a primary key (such as a UUID or a [bit-reversed sequence](https://docs.cloud.google.com/spanner/docs/sequence-tasks#create-sequence) ) that distributes entities evenly:

### GoogleSQL

```
CREATE SEQUENCE CustomerIdSeq OPTIONS (sequence_kind = 'bit_reversed_positive');

CREATE TABLE Customers (
  CustomerId          INT64 NOT NULL DEFAULT (GET_NEXT_SEQUENCE_VALUE(SEQUENCE CustomerIdSeq)),
  CustomerName        STRING(1024),
  IsProcessing        BOOL NOT NULL DEFAULT (FALSE),
  ActiveClaimId       STRING(36),
  ProcessingExpiresAt TIMESTAMP,
) PRIMARY KEY (CustomerId);

CREATE TABLE CustomerWorkItems (
  CustomerId   INT64 NOT NULL,
  CreatedAt    TIMESTAMP NOT NULL DEFAULT (PENDING_COMMIT_TIMESTAMP())
    OPTIONS (allow_commit_timestamp = true),
  BatchIndex   INT64 NOT NULL DEFAULT (0),
  WorkType     STRING(128) NOT NULL,
  WorkPayload  JSON NOT NULL,
) PRIMARY KEY (CustomerId, CreatedAt, BatchIndex),
  INTERLEAVE IN PARENT Customers ON DELETE CASCADE;

CREATE QUEUE CustomerDirtyQueue (
  CustomerId   INT64 NOT NULL,
  MessageId    STRING(36) NOT NULL,
  Payload      BYTES(MAX) NOT NULL,
) PRIMARY KEY (CustomerId, MessageId),
  INTERLEAVE IN PARENT Customers ON DELETE CASCADE;
```

### PostgreSQL

```
CREATE SEQUENCE customeridseq BIT_REVERSED_POSITIVE;

CREATE TABLE customers (
  customerid          bigint NOT NULL DEFAULT nextval('customeridseq'),
  customername        varchar(1024),
  isprocessing        boolean NOT NULL DEFAULT false,
  activeclaimid       varchar(36),
  processingexpiresat timestamptz,
  PRIMARY KEY (customerid)
);

CREATE TABLE customerworkitems (
  customerid  bigint NOT NULL,
  createdat   spanner.commit_timestamp NOT NULL DEFAULT spanner.pending_commit_timestamp(),
  batchindex  bigint NOT NULL DEFAULT 0,
  worktype    varchar(128) NOT NULL,
  workpayload jsonb NOT NULL,
  PRIMARY KEY (customerid, createdat, batchindex)
) INTERLEAVE IN PARENT customers ON DELETE CASCADE;

CREATE QUEUE customerdirtyqueue (
  customerid  bigint NOT NULL,
  messageid   varchar(36) NOT NULL,
  payload     bytea NOT NULL,
  PRIMARY KEY (customerid, messageid)
) INTERLEAVE IN PARENT customers ON DELETE CASCADE;
```

With this schema in place, senders and workers coordinate using the following workflow:

1.  **Insert ordered tasks and a wake-up message:** In the sender transaction, insert the tasks into `CustomerWorkItems` (using `PENDING_COMMIT_TIMESTAMP()` and a zero-based `BatchIndex` to order multiple tasks within the same transaction) and insert a single wake-up message with an empty payload ( `b''` ) into `CustomerDirtyQueue` .
2.  **Claim, process, and release the entity in three steps:**
    - **Step 1 (Claim the entity in Transaction 1):** When a worker receives a wake-up message from `RECEIVE_CustomerDirtyQueue` , it runs a short read-write transaction:
      1.  Check `Customers` and acknowledge the incoming wake-up message ( `DELETE ... ASSERT_ROWS_MODIFIED 1` ). If another worker already holds an active claim ( `IsProcessing = TRUE` and `ProcessingExpiresAt > CURRENT_TIMESTAMP()` ), commit Transaction 1 and exit. The active worker checks for remaining tasks before releasing its claim.
      2.  Otherwise, read the oldest batch of tasks (for example, `LIMIT 25` ) from `CustomerWorkItems` ordered by `CreatedAt ASC, BatchIndex ASC` , and delete any remaining expired fallback message (whose `MessageId` is the prior `ActiveClaimId` ).
      3.  If tasks are returned, set `IsProcessing = TRUE` , `ActiveClaimId` , and `ProcessingExpiresAt` (such as 10 seconds in the future) on `Customers` , and insert a fallback message ( `MessageId = ActiveClaimId` ) into `CustomerDirtyQueue` scheduled for `ProcessingExpiresAt` . If no tasks remain, reset `IsProcessing = FALSE` , `ActiveClaimId = NULL` , and `ProcessingExpiresAt = NULL` .
    - **Step 2 (Process tasks outside a transaction):** After Transaction 1 commits, the worker runs the tasks sequentially in application code. If a task fails, the worker stops the batch early and still runs Step 3 so completed tasks are deleted before retrying the failed task.
    - **Step 3 (Delete completed tasks and release the entity in Transaction 2):** The worker opens a second short read-write transaction, deletes its fallback message ( `ActiveClaimId` ) from `CustomerDirtyQueue` with `ASSERT_ROWS_MODIFIED 1` , deletes the completed tasks from `CustomerWorkItems` , inserts a new wake-up message if more tasks remain in `CustomerWorkItems` , and resets `IsProcessing = FALSE` , `ActiveClaimId = NULL` , and `ProcessingExpiresAt = NULL` on `Customers` .

This pattern preserves strict per-entity ordering and high throughput for the following reasons:

- **Commit-timestamp ordering:** Storing tasks in `CustomerWorkItems` keyed by `(CustomerId, CreatedAt, BatchIndex)` orders every task by its transaction commit timestamp, even when multiple senders write concurrently.
- **Single-split locality:** [Interleaving](https://docs.cloud.google.com/spanner/docs/schema-and-data-model#create-interleaved-tables) `CustomerWorkItems` and `CustomerDirtyQueue` in `Customers` keeps an entity's claim state, tasks, and wake-up messages in the same split. Keeping `CustomerDirtyQueue.Payload` empty ( `b''` ) also minimizes queue storage and in-memory delivery overhead.
- **Short-lived database locks:** Holding a read-write transaction open during Step 2 locks the `Customers` row and blocks concurrent senders. Splitting the claim (Transaction 1) and completion (Transaction 2) into two short transactions ensures that only one worker processes an entity at a time without holding database locks while application code runs.
- **Crash recovery and fencing:** Immediately acknowledging the initial wake-up message in Transaction 1 and scheduling a fallback message ( `ActiveClaimId` ) at `ProcessingExpiresAt` replaces the fixed 10-second `RECEIVE` message lease with a configurable entity claim. If the worker crashes during Step 2, the fallback message matures and wakes up another worker. If a slow worker finishes after its claim expired and another worker already took over, deleting `ActiveClaimId` with `ASSERT_ROWS_MODIFIED 1` in Transaction 2 fails with `OUT_OF_RANGE` . Letting that error propagate (or explicitly rolling back) aborts Transaction 2 before those writes can commit.

The following statements insert two ordered tasks and a wake-up message in a single sender transaction:

### GoogleSQL

```
-- 1. Insert ordered tasks for the entity using a 0-based BatchIndex
INSERT INTO CustomerWorkItems (CustomerId, CreatedAt, BatchIndex, WorkType, WorkPayload)
VALUES
  (456, PENDING_COMMIT_TIMESTAMP(), 0, 'ORDER_CREATED', JSON '{"order_id": 1001}'),
  (456, PENDING_COMMIT_TIMESTAMP(), 1, 'PAYMENT_CAPTURED', JSON '{"order_id": 1001}');

-- 2. Send a single lightweight wake-up message to the interleaved queue
INSERT INTO CustomerDirtyQueue (CustomerId, MessageId, DeliverTime, Payload)
VALUES (456, 'msg-uuid-1', CURRENT_TIMESTAMP(), b'');
```

### PostgreSQL

```
-- 1. Insert ordered tasks for the entity using a 0-based batchindex
INSERT INTO customerworkitems (customerid, createdat, batchindex, worktype, workpayload)
VALUES
  (456, spanner.pending_commit_timestamp(), 0, 'ORDER_CREATED', '{"order_id": 1001}'::jsonb),
  (456, spanner.pending_commit_timestamp(), 1, 'PAYMENT_CAPTURED', '{"order_id": 1001}'::jsonb);

-- 2. Send a single lightweight wake-up message to the interleaved queue
INSERT INTO customerdirtyqueue (customerid, messageid, deliver_time, payload)
VALUES (456, 'msg-uuid-1', CURRENT_TIMESTAMP, CAST('' AS bytea));
```

The following SQL statements show how a worker receives wake-up messages, claims the entity in Transaction 1, and releases the entity in Transaction 2:

### GoogleSQL

```
-- Stream wake-up messages from the queue:
SELECT CustomerId, MessageId, Payload, DeliverTime, SpannerLeaseToken
FROM RECEIVE_CustomerDirtyQueue(max_duration=>'20m', max_batch_size=>10);

-- Transaction 1 (claim the entity):
-- 1. Check whether another worker holds an active claim on the entity
SELECT
  IsProcessing,
  ActiveClaimId,
  IFNULL(ProcessingExpiresAt > CURRENT_TIMESTAMP(), FALSE) AS HasActiveClaim
FROM Customers
WHERE CustomerId = 456;

-- Always acknowledge the incoming wake-up message in Transaction 1.
-- Let OUT_OF_RANGE propagate to roll back the transaction so duplicate workers exit.
-- If IsProcessing = TRUE and HasActiveClaim = TRUE, commit after this DELETE and exit.
DELETE FROM CustomerDirtyQueue
WHERE CustomerId = 456 AND MessageId = 'msg-uuid-1'
ASSERT_ROWS_MODIFIED 1;

-- 2. Only if no active claim exists, read the next ordered batch of work items
SELECT CreatedAt, BatchIndex, WorkType, WorkPayload
FROM CustomerWorkItems
WHERE CustomerId = 456
ORDER BY CreatedAt ASC, BatchIndex ASC
LIMIT 25;

-- 3. Always clean up any remaining expired fallback message
--    (when ActiveClaimId IS NOT NULL AND ActiveClaimId != 'msg-uuid-1'):
DELETE FROM CustomerDirtyQueue
WHERE CustomerId = 456 AND MessageId = 'expired-claim-uuid';

-- 4. If step 2 returns work items, schedule a 10-second fallback message
--    and mark the entity as busy. (If step 2 returns no rows, clear any
--    expired claim fields on Customers instead.)
INSERT INTO CustomerDirtyQueue (CustomerId, MessageId, DeliverTime, Payload)
VALUES (456, 'claim-uuid-2', TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 10 SECOND), b'');

UPDATE Customers
SET IsProcessing = TRUE,
    ActiveClaimId = 'claim-uuid-2',
    ProcessingExpiresAt = TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 10 SECOND)
WHERE CustomerId = 456;

-- Transaction 2 (after processing the batch in order outside a transaction):
-- 1. Verify the claim is still active by deleting the fallback message.
--    Let OUT_OF_RANGE propagate to roll back the transaction if another worker took over.
DELETE FROM CustomerDirtyQueue
WHERE CustomerId = 456 AND MessageId = 'claim-uuid-2'
ASSERT_ROWS_MODIFIED 1;

-- 2. Delete the completed work items
DELETE FROM CustomerWorkItems
WHERE CustomerId = 456
  AND CreatedAt = TIMESTAMP '2026-05-01T12:00:00.123456Z'
  AND BatchIndex IN (0, 1);

-- 3. Check whether any unprocessed work items remain:
SELECT 1 FROM CustomerWorkItems
WHERE CustomerId = 456
LIMIT 1;

-- Run this INSERT only if the preceding SELECT returned a row:
INSERT INTO CustomerDirtyQueue (CustomerId, MessageId, DeliverTime, Payload)
VALUES (456, 'continuation-uuid-3', CURRENT_TIMESTAMP(), b'');

-- 4. Release the entity claim
UPDATE Customers
SET IsProcessing = FALSE, ActiveClaimId = NULL, ProcessingExpiresAt = NULL
WHERE CustomerId = 456 AND ActiveClaimId = 'claim-uuid-2';
```

### PostgreSQL

```
-- Stream wake-up messages from the queue:
SELECT customerid, messageid, payload, deliver_time, spanner_lease_token
FROM spanner.receive_customerdirtyqueue(
    max_batch_size=>10, priority=>NULL, max_duration=>'20m');

-- Transaction 1 (claim the entity):
-- 1. Check whether another worker holds an active claim on the entity
SELECT
  isprocessing,
  activeclaimid,
  COALESCE(processingexpiresat > CURRENT_TIMESTAMP, false) AS hasactiveclaim
FROM customers
WHERE customerid = 456;

-- Always acknowledge the incoming wake-up message in Transaction 1.
-- Let OUT_OF_RANGE propagate to roll back the transaction so duplicate workers exit.
-- If isprocessing = true and hasactiveclaim = true, commit after this DELETE and exit.
DELETE FROM customerdirtyqueue
WHERE customerid = 456 AND messageid = 'msg-uuid-1'
ASSERT_ROWS_MODIFIED 1;

-- 2. Only if no active claim exists, read the next ordered batch of work items
SELECT createdat, batchindex, worktype, workpayload
FROM customerworkitems
WHERE customerid = 456
ORDER BY createdat ASC, batchindex ASC
LIMIT 25;

-- 3. Always clean up any remaining expired fallback message
--    (when activeclaimid IS NOT NULL AND activeclaimid != 'msg-uuid-1'):
DELETE FROM customerdirtyqueue
WHERE customerid = 456 AND messageid = 'expired-claim-uuid';

-- 4. If step 2 returns work items, schedule a 10-second fallback message
--    and mark the entity as busy. (If step 2 returns no rows, clear any
--    expired claim fields on customers instead.)
INSERT INTO customerdirtyqueue (customerid, messageid, deliver_time, payload)
VALUES (456, 'claim-uuid-2', CURRENT_TIMESTAMP + INTERVAL '10 SECOND', CAST('' AS bytea));

UPDATE customers
SET isprocessing = true,
    activeclaimid = 'claim-uuid-2',
    processingexpiresat = CURRENT_TIMESTAMP + INTERVAL '10 SECOND'
WHERE customerid = 456;

-- Transaction 2 (after processing the batch in order outside a transaction):
-- 1. Verify the claim is still active by deleting the fallback message.
--    Let OUT_OF_RANGE propagate to roll back the transaction if another worker took over.
DELETE FROM customerdirtyqueue
WHERE customerid = 456 AND messageid = 'claim-uuid-2'
ASSERT_ROWS_MODIFIED 1;

-- 2. Delete the completed work items
DELETE FROM customerworkitems
WHERE customerid = 456
  AND createdat = '2026-05-01T12:00:00.123456Z'::timestamptz
  AND batchindex IN (0, 1);

-- 3. Check whether any unprocessed work items remain:
SELECT 1 FROM customerworkitems
WHERE customerid = 456
LIMIT 1;

-- Run this INSERT only if the preceding SELECT returned a row:
INSERT INTO customerdirtyqueue (customerid, messageid, deliver_time, payload)
VALUES (456, 'continuation-uuid-3', CURRENT_TIMESTAMP, CAST('' AS bytea));

-- 4. Release the entity claim
UPDATE customers
SET isprocessing = false, activeclaimid = NULL, processingexpiresat = NULL
WHERE customerid = 456 AND activeclaimid = 'claim-uuid-2';
```

### Deduplicate bursty wake-up messages with temporal batching

When an entity receives frequent bursts of updates within a few seconds, sending a separate wake-up message on every transaction creates unnecessary queue writes and redundant wake-up deliveries.

To coalesce wake-up messages at write time, combine [temporal batching](https://docs.cloud.google.com/spanner/docs/queues/queues-examples#batch_messages_using_temporal_batching) with a deterministic `MessageId` :

- Set `MessageId` to the time window's bucket ID (for example, a 5-second bucket).
- Round `DeliverTime` up to the next time window plus a per-entity offset computed with [`FARM_FINGERPRINT`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/hash_functions#farm_fingerprint) (or [`spanner.farm_fingerprint`](https://docs.cloud.google.com/spanner/docs/reference/postgresql/functions#hash-functions) ).
- Insert the wake-up message using [`INSERT OR IGNORE`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/dml-syntax#insert-ignore) in GoogleSQL or `INSERT ... SELECT ... FROM ... WHERE NOT EXISTS` in PostgreSQL.

Every transaction for the same entity within that 5-second window computes the same `MessageId` and `DeliverTime` . All transactions insert their tasks into `CustomerWorkItems` , but only the first transaction in the window writes a wake-up message to `CustomerDirtyQueue` . Adding a per-entity offset spreads delivery times across the 5-second interval so different entities don't all trigger at the exact same second.

For example, the following sender transaction coalesces wake-up messages into 5-second buckets:

### GoogleSQL

```
-- 1. Insert the ordered work item
INSERT INTO CustomerWorkItems (CustomerId, CreatedAt, BatchIndex, WorkType, WorkPayload)
VALUES (456, PENDING_COMMIT_TIMESTAMP(), 0, 'METRIC_RECORDED', JSON '{"v": 42}');

-- 2. Coalesce wake-up messages into a 5-second window with a per-customer offset
INSERT OR IGNORE INTO CustomerDirtyQueue (CustomerId, MessageId, DeliverTime, Payload)
VALUES (
  456,
  CONCAT('bucket-', CAST(DIV(UNIX_SECONDS(CURRENT_TIMESTAMP()), 5) * 5 AS STRING)),
  TIMESTAMP_SECONDS(
    DIV(UNIX_SECONDS(CURRENT_TIMESTAMP()), 5) * 5 + 5
    + MOD(ABS(FARM_FINGERPRINT(CAST(456 AS STRING))), 5)
  ),
  b''
);
```

### PostgreSQL

```
-- 1. Insert the ordered work item
INSERT INTO customerworkitems (customerid, createdat, batchindex, worktype, workpayload)
VALUES (456, spanner.pending_commit_timestamp(), 0, 'METRIC_RECORDED', '{"v": 42}'::jsonb);

-- 2. Coalesce wake-up messages into a 5-second window with a per-customer offset
INSERT INTO customerdirtyqueue (customerid, messageid, deliver_time, payload)
SELECT
  456,
  'bucket-' || bucket.start_sec::text,
  to_timestamp(
    bucket.start_sec + 5
    + (abs(spanner.farm_fingerprint(456::text)) % 5)
  ),
  CAST('' AS bytea)
FROM (
  SELECT (floor(EXTRACT(EPOCH FROM CURRENT_TIMESTAMP))::bigint / 5) * 5 AS start_sec
) AS bucket
WHERE NOT EXISTS (
  SELECT 1 FROM customerdirtyqueue
  WHERE customerid = 456
    AND messageid = 'bucket-' || bucket.start_sec::text
);
```

### Reduce lock contention from concurrent wake-up messages

When multiple wake-up messages for the same busy entity mature at the same time, several workers can start Transaction 1 simultaneously and contend for read-write locks on the same `Customers` row.

To reduce lock contention on hot entities, perform a lock-free [single read](https://docs.cloud.google.com/spanner/docs/reads#single_read_methods) ( `client.Single()` ) before starting Transaction 1 to check whether `IsProcessing = TRUE` and `ProcessingExpiresAt > CURRENT_TIMESTAMP()` . If the entity is busy, sleep for a brief random delay (such as 10 to 50 milliseconds) before running Transaction 1 to re-check `Customers` and acknowledge the duplicate wake-up message.

A single read checks the entity's claim status without acquiring read-write locks, and the random delay staggers the subsequent read-write transactions so workers don't contend on the `Customers` row at the same millisecond. Always re-check `IsProcessing` inside Transaction 1 before deleting the wake-up message so that if the active worker finishes right after the single read, the wake-up message isn't deleted while new tasks remain unprocessed.

For example, the following Go function applies a random delay when an entity is already busy:

```
func maybeJitterBeforeClaim(ctx context.Context, client *spanner.Client, customerID int64) error {
    iter := client.Single().Query(ctx, spanner.Statement{
        SQL: `SELECT IsProcessing AND IFNULL(ProcessingExpiresAt > CURRENT_TIMESTAMP(), FALSE) AS IsBusy
              FROM Customers WHERE CustomerId = @cid`,
        Params: map[string]interface{}{"cid": customerID},
    })
    defer iter.Stop()
    row, err := iter.Next()
    if errors.Is(err, iterator.Done) {
        return nil
    }
    if err != nil {
        return err
    }
    var isBusy bool
    if err := row.Columns(&isBusy); err != nil {
        return err
    }
    if isBusy {
        time.Sleep(time.Duration(10+rand.Intn(40)) * time.Millisecond)
    }
    return nil
}
```

### Handle long-running batches and external calls

When a batch of tasks takes longer to process than the initial 10-second `ProcessingExpiresAt` window, or calls external APIs (such as payment gateways or webhooks) that aren't rolled back when a database transaction aborts, you need to prevent duplicate external actions and premature claim expiration.

To protect external calls and extend entity claims for slow batches:

1.  **Pass a deterministic idempotency key to external APIs:** Combine `CustomerId` , `CreatedAt` , and `BatchIndex` (for example, `456:2026-05-01T12:00:00.123456Z:0` ) and pass that key to downstream services during Step 2.
2.  **Extend the entity claim with a periodic heartbeat:** Either set a longer initial `ProcessingExpiresAt` and fallback `DeliverTime` (such as 30 or 60 seconds) in Transaction 1, or run a background heartbeat transaction before the claim expires (for example, every 3 to 4 seconds on a 10-second claim window). Each heartbeat transaction deletes the current fallback message ( `ActiveClaimId` ) with `ASSERT_ROWS_MODIFIED 1` , inserts a new fallback message with a refreshed future `DeliverTime` , and updates `ActiveClaimId` and `ProcessingExpiresAt` on `Customers` . Stop the heartbeat before starting Transaction 2.

Because Step 2 runs outside a database transaction and queue delivery is at-least-once, a worker crash after an external API call causes the next worker to retry the task. Passing a deterministic idempotency key lets the external service ignore duplicate requests. For more information, see [Transactional outbox](https://docs.cloud.google.com/spanner/docs/queues/queues-at-most-once#transactional_outbox) . Periodically replacing the fallback message and extending `ProcessingExpiresAt` keeps the entity claim active while the worker is healthy, while still allowing another worker to take over if the worker crashes.

For example, the following heartbeat transaction extends an active entity claim by 10 seconds:

### GoogleSQL

```
-- Run every 3 to 4 seconds on a 10-second claim window before starting Transaction 2:
DELETE FROM CustomerDirtyQueue
WHERE CustomerId = 456 AND MessageId = 'claim-uuid-2'
ASSERT_ROWS_MODIFIED 1;

INSERT INTO CustomerDirtyQueue (CustomerId, MessageId, DeliverTime, Payload)
VALUES (456, 'claim-uuid-3', TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 10 SECOND), b'');

UPDATE Customers
SET ActiveClaimId = 'claim-uuid-3',
    ProcessingExpiresAt = TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 10 SECOND)
WHERE CustomerId = 456 AND ActiveClaimId = 'claim-uuid-2';
```

### PostgreSQL

```
-- Run every 3 to 4 seconds on a 10-second claim window before starting Transaction 2:
DELETE FROM customerdirtyqueue
WHERE customerid = 456 AND messageid = 'claim-uuid-2'
ASSERT_ROWS_MODIFIED 1;

INSERT INTO customerdirtyqueue (customerid, messageid, deliver_time, payload)
VALUES (456, 'claim-uuid-3', CURRENT_TIMESTAMP + INTERVAL '10 SECOND', CAST('' AS bytea));

UPDATE customers
SET activeclaimid = 'claim-uuid-3',
    processingexpiresat = CURRENT_TIMESTAMP + INTERVAL '10 SECOND'
WHERE customerid = 456 AND activeclaimid = 'claim-uuid-2';
```

## Scale high-volume entities with sharding and stale reads

When a single entity receives more writes than one Spanner split can handle, writing every task into a single `Customers` split creates a [write hotspot](https://docs.cloud.google.com/spanner/docs/introspection/hot-split-statistics) , even if you still need to process tasks for that entity in commit order.

To scale write throughput for a high-volume entity while preserving global ordering, distribute writes across `N` shards and merge them asynchronously with a periodic sweeper worker using [stale reads](https://docs.cloud.google.com/spanner/docs/reads#perform-stale-read) :

- **Shard the work and deduplication tables:** Create a `CustomerShards` parent table with `PRIMARY KEY (ShardId, CustomerId)` ( `ShardId` from `0` to `N - 1` ) and interleave a sharded work table keyed by `(ShardId, CustomerId, CreatedAt, BatchIndex)` . Making `ShardId` the leading key column spreads writes across `N` splits. To prevent duplicate task rows when a retried transaction receives a new `PENDING_COMMIT_TIMESTAMP()` , also interleave a batch deduplication table keyed by `(ShardId, CustomerId, BatchId)` with a [TTL](https://docs.cloud.google.com/spanner/docs/ttl/working-with-ttl#create) policy.
- **Route each producer batch to one shard without queue writes:** In each producer transaction, compute `ShardId` from a hash of a unique `BatchId` (such as `FARM_FINGERPRINT` ) and insert into both the deduplication table and the sharded work table without inserting into a queue. Hashing `BatchId` keeps each producer transaction within a single split and eliminates per-write queue overhead.
- **Sweep shards in order using a stale read and a progress cursor:** Maintain an active claim and a `(LastSweptAt, LastBatchIndex, LastShardId)` cursor in a separate `CustomerSweeperState` table with an interleaved wake-up queue containing one recurring sweeper message per entity. When triggered, the worker claims the entity (using the same claim and fallback-message pattern as [Use an ordered work table and a wake-up queue](https://docs.cloud.google.com/spanner/docs/queues/queues-in-order#use_an_ordered_work_table_and_a_wake-up_queue) ), queries across all `N` shards after the cursor using a 10-second [exact staleness](https://docs.cloud.google.com/spanner/docs/timestamp-bounds#exact_staleness) read ordered by `CreatedAt ASC, BatchIndex ASC, ShardId ASC` , processes the batch in order, and commits a completion transaction that deletes the processed rows, advances the cursor, and schedules the next sweep. Using a stale read avoids blocking on in-flight producer writes across splits, while the cursor skips rows that were processed and deleted within the staleness window.

## What's next

- Learn how to [use Spanner queues, including best practices and monitoring](https://docs.cloud.google.com/spanner/docs/queues/queues-using) .
- Explore more [Spanner queues scenarios and examples](https://docs.cloud.google.com/spanner/docs/queues/queues-examples) .
- Learn about [exactly-once processing and at-most-once acknowledgment](https://docs.cloud.google.com/spanner/docs/queues/queues-at-most-once) .
- Configure access control with [fine-grained access control for queues](https://docs.cloud.google.com/spanner/docs/fgac-queues) .
