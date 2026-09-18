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

### PostgreSQL

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

After the transaction commits, the receiver for `UserTasks` streams the message, sends the email, and acknowledges the message:

### GoogleSQL

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

### PostgreSQL

    -- 1. In the receiver process, stream messages from the queue
    SELECT
      userid,
      messageid,
      payload,
      deliver_time,
      spanner_lease_expiration_timestamp,
      spanner_lease_token
    FROM spanner.receive_usertasks(NULL, NULL, '20m');
    
    -- 2. After sending the welcome email, acknowledge the message
    DELETE FROM usertasks
    WHERE userid = 124 AND messageid = 'welcome-email-id';

## Handle long-running work

If you have work that might take longer than the default lease (greater than 10 seconds), periodically call `SELECT * FROM RENEWLEASE_ QUEUE_NAME ()` .

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

    -- 1. Insert into Users table
    INSERT INTO Users (UserId, UserName) VALUES (125, 'Trial User');
    
    -- 2. Send message to queue with a future delivery time
    INSERT INTO UserTasks (UserId, MessageId, Payload, DeliverTime)
    VALUES (125, 'trial-expire-reminder', b'{"type": "reminder"}', TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 29 DAY));

### PostgreSQL

    -- 1. Insert into users table
    INSERT INTO users (userid, username) VALUES (125, 'Trial User');
    
    -- 2. Send message to queue with a future delivery time
    INSERT INTO usertasks (userid, messageid, payload, deliver_time)
    VALUES (125, 'trial-expire-reminder', CAST('{"type": "reminder"}' AS bytea), CURRENT_TIMESTAMP + INTERVAL '29 DAY');

## Handle large message payloads

If your message payload is large, use the out-of-band storage pattern. Store the large payload in a separate table and put a reference to it in the queue message.

For example, image processing:

### GoogleSQL

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

### PostgreSQL

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

    -- Inside your application transaction:
    -- 1. Acknowledge current message
    DELETE FROM AggregationQueue
    WHERE TaskType = 'hourly-aggregator' AND MessageId = 'current-uuid'
    ASSERT_ROWS_MODIFIED 1;
    
    -- 2. Schedule next run 1 hour in the future
    INSERT INTO AggregationQueue (TaskType, MessageId, Payload, DeliverTime)
    VALUES ('hourly-aggregator', 'next-uuid', b'', TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR));

### PostgreSQL

    -- Inside your application transaction:
    -- 1. Acknowledge current message
    DELETE FROM aggregationqueue
    WHERE tasktype = 'hourly-aggregator' AND messageid = 'current-uuid'
    ASSERT_ROWS_MODIFIED 1;
    
    -- 2. Schedule next run 1 hour in the future
    INSERT INTO aggregationqueue (tasktype, messageid, payload, deliver_time)
    VALUES ('hourly-aggregator', 'next-uuid', CAST('' AS bytea), CURRENT_TIMESTAMP + INTERVAL '1 HOUR');

Alternatively, use the client library `Ack` and `Send` mutations. These examples assume you have a `Message` object encapsulating the key and payload:

### Java

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

### Go

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

### Python

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

### Node.js

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

## What's next

  - Learn how to [use Spanner queues, including best practices and monitoring](https://docs.cloud.google.com/spanner/docs/queues/queues-using) .
  - Learn about [exactly-once processing and at-most-once acknowledgment](https://docs.cloud.google.com/spanner/docs/queues/queues-at-most-once) .
  - Configure access control with [fine-grained access control for queues](https://docs.cloud.google.com/spanner/docs/fgac-queues) .
