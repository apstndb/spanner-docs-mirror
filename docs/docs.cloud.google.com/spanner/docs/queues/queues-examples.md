---
name: documents/docs.cloud.google.com/spanner/docs/queues/queues-examples
uri: https://docs.cloud.google.com/spanner/docs/queues/queues-examples
title: Spanner queues scenarios and examples
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

This document provides architectural patterns and code examples for common messaging scenarios using Spanner queues. You can use these patterns to trigger asynchronous work after transactions commit, schedule delayed or recurring tasks, manage large message payloads with out-of-band storage, coordinate multi-event workflows, and checkpoint or extend leases for long-running background jobs.

## Exactly-once processing and at-most-once acknowledgment

The various considerations and solutions for exactly-once processing and at-most-once acknowledgment are described in more detail on the [Exactly-once processing and at-most-once acknowledgment](https://docs.cloud.google.com/spanner/docs/queues/queues-at-most-once) page.

## Perform work after a transaction commits

To perform work after a transaction commits, send a message to the queue within the same transaction.

For example, a new user signup triggers a welcome email:

### GoogleSQL

```
-- Inside your application transaction:
-- 1. Insert into Users table
INSERT INTO Users (UserId, UserName) VALUES (124, 'New User');

-- 2. Send message to queue to trigger email
INSERT INTO UserTasks (UserId, MessageId, Payload)
VALUES (
  124,
  'welcome-email-id',
  b'{"type": "welcome", "email": "user@example.com"}'
);
```

### PostgreSQL

```
-- Inside your application transaction:
-- 1. Insert into users table
INSERT INTO users (userid, username) VALUES (124, 'New User');

-- 2. Send message to queue to trigger email
INSERT INTO usertasks (userid, messageid, payload)
VALUES (
  124,
  'welcome-email-id',
  CAST('{"type": "welcome", "email": "user@example.com"}' AS bytea)
);
```

After the transaction commits, the receiver for `UserTasks` streams the message, sends the email, and acknowledges the message:

### GoogleSQL

```
-- 1. In the receiver process, stream messages from the queue
SELECT
  UserId,
  MessageId,
  Payload,
  DeliverTime,
  SpannerLeaseExpirationTimestamp,
  SpannerLeaseToken
FROM RECEIVE_UserTasks(max_duration=>'20m');

-- 2. After sending the welcome email, acknowledge the message
DELETE FROM UserTasks
WHERE UserId = 124 AND MessageId = 'welcome-email-id';
```

### PostgreSQL

```
-- 1. In the receiver process, stream messages from the queue
SELECT
  userid,
  messageid,
  payload,
  deliver_time,
  spanner_lease_expiration_timestamp,
  spanner_lease_token
FROM spanner.receive_usertasks(
    max_batch_size=>NULL, priority=>NULL, max_duration=>'20m');

-- 2. After sending the welcome email, acknowledge the message
DELETE FROM usertasks
WHERE userid = 124 AND messageid = 'welcome-email-id';
```

## Handle long-running work

If you have work that might take longer than the default lease (greater than 10 seconds), periodically call `SELECT * FROM RENEWLEASE_ `` QUEUE_NAME `` ()` .

For example, generating a report:

1.  The receiver gets a message from `RECEIVE_ReportQueue()` .
2.  Start report generation.
3.  Every 5 seconds, call `SELECT * FROM RENEWLEASE_ReportQueue([leaseToken])` in a separate thread or routine.
4.  Upon completion, acknowledge the message and store the report.

Alternatively, if you have long-running work that requires at-most-once processing, or a long lease time, do the following:

1.  Acknowledge ( `DELETE` or `ACK` ) the current queue message on arrival. In the same transaction, re-enqueue a new queue message with a delivery timestamp in the future, beyond the time that processing takes.
2.  Proceed with processing, and acknowledge the newly enqueued message when done.

The advantages of this approach are that there is no need to continuously extend the lease, and the message is not redelivered until the future time arrives (which covers crashes). If the initial acknowledgment succeeds, it achieves at-most-once processing.

## Checkpoint long-running work

Spanner queues can manage tasks lasting minutes to hours, not just quick jobs. For these long-running tasks, use the following approach:

1.  **Store metadata externally:** Use out-of-band storage to hold the details and state of the task.
2.  **Checkpoint regularly:** To recover from crashes without losing much progress, the task should periodically save its state.
3.  **Use the recommended checkpointing pattern:** The best way to checkpoint is to atomically acknowledge ( `ACK` ) the current queue message and send a new message scheduled for future delivery. This new message contains or points to the updated state, which prevents immediate redelivery to another worker.

This pattern reduces duplicate work even if full checkpointing isn't possible, although the task restarts from the beginning after a crash in that scenario.

## Schedule work for a specific time in the future

To schedule work for a specific time in the future, set the `DeliverTime` column when inserting the message.

For example, a trial expiration reminder:

### GoogleSQL

```
-- 1. Insert into Users table
INSERT INTO Users (UserId, UserName) VALUES (125, 'Trial User');

-- 2. Send message to queue with a future delivery time
INSERT INTO UserTasks (UserId, MessageId, Payload, DeliverTime)
VALUES (125, 'trial-expire-reminder', b'{"type": "reminder"}', TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 29 DAY));
```

### PostgreSQL

```
-- 1. Insert into users table
INSERT INTO users (userid, username) VALUES (125, 'Trial User');

-- 2. Send message to queue with a future delivery time
INSERT INTO usertasks (userid, messageid, payload, deliver_time)
VALUES (125, 'trial-expire-reminder', CAST('{"type": "reminder"}' AS bytea), CURRENT_TIMESTAMP + INTERVAL '29 DAY');
```

## Handle large message payloads

If your message payload is large, use the out-of-band storage pattern. Store the large payload in a separate table and put a reference to it in the queue message.

For example, image processing:

### GoogleSQL

```
-- Schema
CREATE TABLE ImageUploads (
  UserId    INT64 NOT NULL,
  ImageId   STRING(36) NOT NULL,
  ImageData BYTES(MAX),
  Status    STRING(MAX) -- PENDING, PROCESSING, DONE
) PRIMARY KEY (UserId, ImageId),
  INTERLEAVE IN PARENT Users;

CREATE QUEUE ImageProcessingQueue (
  UserId    INT64 NOT NULL,
  ImageId   STRING(36) NOT NULL,
  Payload   BYTES(1) NOT NULL -- Payload can be minimal
) PRIMARY KEY (UserId, ImageId),
  INTERLEAVE IN PARENT ImageUploads ON DELETE CASCADE;

-- Application Logic
-- 1. Upload image, insert into ImageUploads with Status 'PENDING'
-- 2. Send message to ImageProcessingQueue
INSERT INTO ImageProcessingQueue (UserId, ImageId, Payload) VALUES (123, 'image-uuid-1', b'');

-- Receiver for ImageProcessingQueue:
-- 1. Receives message (UserId, ImageId).
-- 2. Reads ImageData from ImageUploads.
-- 3. Processes image.
-- 4. Updates ImageUploads Status to 'DONE'.
-- 5. ACKs the queue message.
```

### PostgreSQL

```
-- Schema
CREATE TABLE imageuploads (
  userid    bigint NOT NULL,
  imageid   varchar(36) NOT NULL,
  imagedata bytea,
  status    varchar, -- PENDING, PROCESSING, DONE
  PRIMARY KEY (userid, imageid)
) INTERLEAVE IN PARENT users;

CREATE QUEUE imageprocessingqueue (
  userid    bigint NOT NULL,
  imageid   varchar(36) NOT NULL,
  payload   bytea NOT NULL, -- Payload can be minimal
  PRIMARY KEY (userid, imageid)
) INTERLEAVE IN PARENT imageuploads ON DELETE CASCADE;

-- Application Logic
-- 1. Upload image, insert into imageuploads with status 'PENDING'
-- 2. Send message to imageprocessingqueue
INSERT INTO imageprocessingqueue (userid, imageid, payload) VALUES (123, 'image-uuid-1', CAST('' AS bytea));

-- Receiver for imageprocessingqueue:
-- 1. Receives message (userid, imageid).
-- 2. Reads imagedata from imageuploads.
-- 3. Processes image.
-- 4. Updates imageuploads status to 'DONE'.
-- 5. ACKs the queue message.
```

## Wait for multiple events before proceeding

To wait for multiple events before proceeding (such as a join operation), use a table to track state and a queue to trigger checks.

For example, order fulfillment requiring inventory and payment:

1.  Create an `Orders` table with `InventoryStatus` and `PaymentStatus` .
2.  When inventory is confirmed, update `Orders` and send a message to `OrderCheckQueue` .
3.  When payment is confirmed, update `Orders` and send a message to `OrderCheckQueue` .
4.  The receiver for `OrderCheckQueue` checks the `Orders` table. If both statuses are confirmed, it proceeds with shipping and acknowledges the message. If not, it might re-queue for a later check or execute other logic.

## Perform an action periodically

To perform an action periodically, use the periodic scheduling pattern. The receiver acknowledges the message and sends a new one scheduled for the next interval.

For example, hourly data aggregation:

### GoogleSQL

```
-- Inside your application transaction:
-- 1. Acknowledge current message
DELETE FROM AggregationQueue
WHERE TaskType = 'hourly-aggregator' AND MessageId = 'current-uuid'
ASSERT_ROWS_MODIFIED 1;

-- 2. Schedule next run 1 hour in the future
INSERT INTO AggregationQueue (TaskType, MessageId, Payload, DeliverTime)
VALUES ('hourly-aggregator', 'next-uuid', b'', TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR));
```

### PostgreSQL

```
-- Inside your application transaction:
-- 1. Acknowledge current message
DELETE FROM aggregationqueue
WHERE tasktype = 'hourly-aggregator' AND messageid = 'current-uuid'
ASSERT_ROWS_MODIFIED 1;

-- 2. Schedule next run 1 hour in the future
INSERT INTO aggregationqueue (tasktype, messageid, payload, deliver_time)
VALUES ('hourly-aggregator', 'next-uuid', CAST('' AS bytea), CURRENT_TIMESTAMP + INTERVAL '1 HOUR');
```

Alternatively, use the client library `Ack` and `Send` mutations. These examples assume you have a `Message` object encapsulating the key and payload:

### Java

```
// Receiver logic for AggregationQueue
public void process(DatabaseClient dbClient, Message msg) {
  // ... do aggregation ...

  // ACK current message and schedule next run (1 hour from now)
  Instant nextRun = Instant.now().plus(Duration.ofHours(1));
  Mutation ackMutation =
      Mutation.newAckBuilder("AggregationQueue")
          .setKey(msg.getKey()) // Ack
          .build();
  Mutation sendMutation =
      Mutation.newSendBuilder("AggregationQueue")
          .setKey(Key.of("hourly-aggregator", "next-uuid"))
          .setPayload(Value.bytes(ByteArray.copyFrom("")))
          .setDeliveryTime(nextRun) // Schedule next
          .build();
  dbClient.write(Arrays.asList(ackMutation, sendMutation));
}
```

### Go

```
// Receiver logic for AggregationQueue
func process(msg) {
    // ... do aggregation ...

    // ACK current message and schedule next run
    nextRun := time.Now().Add(1 * time.Hour)
    _, err := client.Apply(ctx, []*spanner.Mutation{
        spanner.Ack("AggregationQueue", msg.Key), // Ack
        spanner.Send("AggregationQueue",
            spanner.Key{"hourly-aggregator", "next-uuid"},
            []byte(""),
            spanner.WithDeliveryTime(nextRun), // Schedule next
        ),
    })
    // ... handle err ...
}
```

### Python

```
# Receiver logic for AggregationQueue
def process(database: spanner.Database, msg: Message):
  # ... do aggregation ...
  # ACK current message and schedule next run (1 hour from now)
  next_run = datetime.datetime.now(datetime.timezone.utc) + datetime.timedelta(
      hours=1
  )
  with database.batch() as batch:
    batch.ack(
        queue="AggregationQueue",
        key=msg.key,  # Ack
    )
    batch.send(
        queue="AggregationQueue",
        key=("hourly-aggregator", "next-uuid"),
        payload=b"",
        deliver_time=next_run,  # Schedule next
    )
```

### Node.js

```
/**
 * Receiver logic for AggregationQueue
 * @param {import('@google-cloud/spanner').Database} database
 * @param { { key: Array<string|number>, payload: Buffer } } msg
 */
async function process(database, msg) {
  // ... do aggregation ...
  // ACK current message and schedule next run (1 hour from now)
  const nextRun = new Date(Date.now() + 60 * 60 * 1000);
  await database.runTransactionAsync(async (transaction) => {
    // Ack current message
    transaction.queueAck('AggregationQueue', msg.key);
    // Schedule next run
    transaction.queueSend(
      'AggregationQueue',
      ['hourly-aggregator', 'next-uuid'],
      {
        payload: Buffer.from(''),
        deliverTime: nextRun,
      }
    );
    await transaction.commit();
  });
}
```

## Batch messages using temporal batching

Spanner queues deliver messages with minimal latency. However, when high volumes of messages arrive continuously from independent clients, processing each message individually can create high transaction overhead. Trying to query or scan the queue table manually to batch messages can introduce range-locking contention, elevated abort rates, and extra read costs.

To achieve high-throughput batching without contention, apply the **temporal batching** pattern. Senders align the `DeliverTime` of messages to a discrete time window in the future (for example, rounding up to the nearest 10-second boundary). Because independent senders compute an identical future timestamp, Spanner tends to group the messages from the same split together and delivers them in a single batch if `max_batch_size` in the `RECEIVE_ `` QUEUE_NAME `` ()` table-valued function allows, or multiple batches if `max_batch_size` is smaller than the number of messages to be delivered.

The delivery timestamp can be calculated with this formula:

\$\$ \text{DeliverTime} = \text{RoundDown}(\text{CurrentTime}, \text{FixedDelay}) + \text{FixedDelay} \$\$

For example, with a 10-second window, messages enqueued between 09:05:00 and 09:05:09.999 all receive a `DeliverTime` of 09:05:10:

### GoogleSQL

```
-- Calculate delivery time rounded to the next 10-second interval.
-- Use DIV(..., 10) * 10 to perform the RoundDown in SQL:
INSERT INTO OrderProcessingQueue (OrderId, DeliverTime, Payload)
VALUES (
  'order-101',
  TIMESTAMP_SECONDS(DIV(UNIX_SECONDS(CURRENT_TIMESTAMP()), 10) * 10 + 10),
  b'{"item": "book", "qty": 1}'
);
```

### PostgreSQL

```
-- Calculate delivery time rounded to the next 10-second interval:
INSERT INTO orderprocessingqueue (orderid, deliver_time, payload)
VALUES (
  'order-101',
  to_timestamp((floor(extract(epoch from CURRENT_TIMESTAMP) / 10) * 10) + 10),
  CAST('{"item": "book", "qty": 1}' AS bytea)
);
```

Receivers then pull these co-timed messages in batches using `max_batch_size` :

### GoogleSQL

```
SELECT OrderId, Payload, DeliverTime, SpannerLeaseToken
FROM RECEIVE_OrderProcessingQueue(max_duration=>'20m', max_batch_size=>50);
```

### PostgreSQL

```
SELECT orderid, payload, deliver_time, spanner_lease_token
FROM spanner.receive_orderprocessingqueue(
    max_batch_size=>50, priority=>NULL, max_duration=>'20m');
```

If you send millions of messages at the same time, aligning all messages to the exact same second can cause sudden processing spikes. To distribute the work evenly and still batch messages by entity, add an offset to the calculation based on a unique identifier (such as `TIMESTAMP_ADD(deliver_time, INTERVAL MOD(ABS(FARM_FINGERPRINT(CAST(UserId AS STRING))), 10) SECOND)` ).

## Decouple data modification from processing (dirty flag pattern)

In high-throughput transactional applications, executing complex recomputation, search indexing, or cache invalidation directly inside user-facing transactions can slow down the user experience by increasing latency and causing lock contention on shared rows.

The **dirty flag pattern** decouples data modifications from asynchronous processing. When a transaction modifies a table, it writes a lightweight "dirty bit" message into a queue within the same transaction. A background worker subsequently consumes the message and performs the expensive processing asynchronously.

For example, when a customer makes a profile or settings change:

### GoogleSQL

```
-- Inside user profile update transaction:
-- 1. Update the primary entity table
UPDATE UserProfiles
SET FullName = 'Jane Doe', UpdatedAt = CURRENT_TIMESTAMP()
WHERE UserId = 456;

-- 2. Send lightweight dirty flag message to the queue.
-- It is recommended to interleave the queue in the primary UserProfiles
-- table for better transaction performance.
INSERT INTO UserDirtyQueue (UserId, TaskType, CommitTimestamp, Payload)
VALUES (456, 'reindex-user-profile', CURRENT_TIMESTAMP(), b'');
```

### PostgreSQL

```
-- Inside user profile update transaction:
-- 1. Update the primary entity table
UPDATE userprofiles
SET fullname = 'Jane Doe', updatedat = CURRENT_TIMESTAMP
WHERE userid = 456;

-- 2. Send lightweight dirty flag message to the queue
INSERT INTO userdirtyqueue (userid, tasktype, committimestamp, payload)
VALUES (456, 'reindex-user-profile', CURRENT_TIMESTAMP, CAST('' AS bytea));
```

The background receiver for `UserDirtyQueue` receives the `UserId` , reads the fresh profile row outside the user's critical path, and recomputes the search index or updates external caches. Batching can be applied to this pattern as well if multiple updates against the same user are sent to the queue within a short period of time. In that case, the receiver TVF can specify a `max_batch_size` greater than 1 to receive multiple messages from the same batch.

## Monitor worker health and detect timeouts

You can use scheduled queue messages to build a fault-tolerant heartbeat and health-monitoring system for fleets of worker nodes or microservice instances.

To implement health checking:

1.  **Register worker on startup:** When a worker initializes, it inserts a heartbeat message into a health-check queue with a future `DeliverTime` set to its failure deadline (for example, 60 seconds).
2.  **Send periodic heartbeats:** While healthy, the worker periodically (for example, every 10 seconds) refreshes its heartbeat message by advancing the `DeliverTime` another 60 seconds into the future.
3.  **Detect failures:** If the worker crashes or loses network connectivity, heartbeat refreshes stop. After 60 seconds, the delivery timestamp matures ( `DeliverTime <= CURRENT_TIMESTAMP()` ), and Spanner delivers the message to an alerting receiver, which initiates failover or task reassignment.

**Important:** Spanner queues do not support `UPDATE` DML statements. Therefore, to refresh the heartbeat timestamp, you must delete the existing message and insert a replacement with the new `DeliverTime` within a single transaction, or apply client library `Ack` and `Send` mutations. Ensure that the queue's primary key is `WorkerId` alone (rather than `(WorkerId, MessageId)` ) so that only one heartbeat message exists per worker at any given time.

### GoogleSQL

```
-- Inside the worker heartbeat transaction (executed every 10 seconds):
-- 1. Acknowledge the existing heartbeat message
DELETE FROM WorkerHealthQueue
WHERE WorkerId = 'worker-node-42' ASSERT_ROWS_MODIFIED 1;

-- 2. Send replacement heartbeat with refreshed 60-second deadline
INSERT INTO WorkerHealthQueue (WorkerId, DeliverTime, Payload)
VALUES (
  'worker-node-42',
  TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 60 SECOND),
  b'{"status": "healthy", "active_jobs": 3}'
);
```

### PostgreSQL

```
-- Inside the worker heartbeat transaction (executed every 10 seconds):
-- 1. Acknowledge the existing heartbeat message
DELETE FROM workerhealthqueue
WHERE workerid = 'worker-node-42' ASSERT_ROWS_MODIFIED 1;

-- 2. Send replacement heartbeat with refreshed 60-second deadline
INSERT INTO workerhealthqueue (workerid, deliver_time, payload)
VALUES (
  'worker-node-42',
  CURRENT_TIMESTAMP + INTERVAL '60 SECOND',
  CAST('{"status": "healthy", "active_jobs": 3}' AS bytea)
);
```

When using client library mutations, ensure `Ack` precedes `Send` in the mutation slice, as in this Go example:

```
// Worker heartbeat loop
func sendHeartbeat(ctx context.Context, client *spanner.Client, workerID string) error {
    newDeadline := time.Now().Add(60 * time.Second)
    _, err := client.Apply(ctx, []*spanner.Mutation{
        spanner.Ack("WorkerHealthQueue", spanner.Key{workerID}),
        spanner.Send(
            "WorkerHealthQueue",
            spanner.Key{workerID},
            []byte(`{"status":"healthy"}`),
            spanner.WithDeliveryTime(newDeadline),
        ),
    })
    return err
}
```

When a worker shuts down gracefully, it explicitly deletes its heartbeat message so no false alert is triggered:

```
DELETE FROM WorkerHealthQueue WHERE WorkerId = 'worker-node-42' ASSERT_ROWS_MODIFIED 1;
```

## Handle multiple task types in a single queue

Spanner instances have [limits](https://docs.cloud.google.com/spanner/quotas#queue-limits) on the total number of queues. Creating a separate queue for every small asynchronous operation can quickly hit this limit and requires managing many concurrent receiver queries.

To consolidate operations, combine different task types into a single queue, which is called a **polymorphic queue** . There are two strategies used to create a polymorphic queue.

### Strategy 1: Include a type column in the primary key

### GoogleSQL

```
CREATE QUEUE ApplicationTasks (
  TaskType   STRING(50) NOT NULL,
  TaskId     STRING(36) NOT NULL,
  Payload    BYTES(MAX) NOT NULL,
) PRIMARY KEY (TaskType, TaskId);

-- Enqueue an email task
INSERT INTO ApplicationTasks (TaskType, TaskId, Payload)
VALUES ('SEND_EMAIL', 'task-uuid-1', b'{"to": "user@example.com", "template": "welcome"}');

-- Enqueue an image thumbnail task
INSERT INTO ApplicationTasks (TaskType, TaskId, Payload)
VALUES ('GENERATE_THUMBNAIL', 'task-uuid-2', b'{"image_id": "img-789", "size": "small"}');
```

### PostgreSQL

```
CREATE QUEUE applicationtasks (
  tasktype   varchar(50) NOT NULL,
  taskid     varchar(36) NOT NULL,
  payload    bytea NOT NULL,
  PRIMARY KEY (tasktype, taskid)
);

-- Enqueue an email task
INSERT INTO applicationtasks (tasktype, taskid, payload)
VALUES ('SEND_EMAIL', 'task-uuid-1', CAST('{"to": "user@example.com", "template": "welcome"}' AS bytea));

-- Enqueue an image thumbnail task
INSERT INTO applicationtasks (tasktype, taskid, payload)
VALUES ('GENERATE_THUMBNAIL', 'task-uuid-2', CAST('{"image_id": "img-789", "size": "small"}' AS bytea));
```

The receiver inspects `TaskType` and dispatches the payload to the corresponding handler.

### Strategy 2: Polymorphic payload structure

Alternatively, use a JSON payload containing an action or type discriminator field:

```
{
  "action": "SYNC_INVENTORY",
  "data": { "item_id": 987, "delta": -1 }
}
```

> **Caution:** avoid combining ultra-fast, latency-sensitive tasks with slow, long-running jobs in the same queue. If long-running tasks occupy receiver workers, they can starve fast tasks and introduce processing latency.

## Implement custom retry delays

Spanner queues automatically retry failed or unacknowledged messages with built-in exponential backoff. However, in scenarios where a message fails due to a known reason with a known duration, or an external rate limit (such as an HTTP 429 response specifying a `Retry-After` header), relying on automatic backoff can cause premature retry attempts that waste CPU resources.

To implement a custom retry delay:

1.  Catch the specific transient failure in your message processor.
2.  Acknowledge the current message to satisfy the current delivery attempt.
3.  In the same transaction, send a replacement message with an explicit `DeliverTime` set to the chosen future retry time (in the following example, 5 minutes later).

### GoogleSQL

```
-- Inside failure-handling transaction:
-- 1. Acknowledge the failed message
DELETE FROM OutboundNotificationQueue
WHERE NotificationId = 'notif-555' ASSERT_ROWS_MODIFIED 1;

-- 2. Reschedule delivery 5 minutes in the future
INSERT INTO OutboundNotificationQueue (NotificationId, DeliverTime, Payload)
VALUES (
  'notif-555',
  TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 5 MINUTE),
  b'{"recipient": "user@example.com", "retry_count": 2}'
);
```

### PostgreSQL

```
-- Inside failure-handling transaction:
-- 1. Acknowledge the failed message
DELETE FROM outboundnotificationqueue
WHERE notificationid = 'notif-555' ASSERT_ROWS_MODIFIED 1;

-- 2. Reschedule delivery 5 minutes in the future
INSERT INTO outboundnotificationqueue (notificationid, deliver_time, payload)
VALUES (
  'notif-555',
  CURRENT_TIMESTAMP + INTERVAL '5 MINUTE',
  CAST('{"recipient": "user@example.com", "retry_count": 2}' AS bytea)
);
```

This approach allows your application to precisely manage backoff schedules and avoid saturating external APIs during downstream recovery periods.

## What's next

- Learn how to [use Spanner queues, including best practices and monitoring](https://docs.cloud.google.com/spanner/docs/queues/queues-using) .
- Learn about [exactly-once processing and at-most-once acknowledgment](https://docs.cloud.google.com/spanner/docs/queues/queues-at-most-once) .
- Configure access control with [fine-grained access control for queues](https://docs.cloud.google.com/spanner/docs/fgac-queues) .
