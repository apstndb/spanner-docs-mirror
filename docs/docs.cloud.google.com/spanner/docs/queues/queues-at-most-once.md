---
name: documents/docs.cloud.google.com/spanner/docs/queues/queues-at-most-once
uri: https://docs.cloud.google.com/spanner/docs/queues/queues-at-most-once
title: Exactly-once processing and at-most-once acknowledgment
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

Building reliable message queues and microservices across distributed cloud databases like Spanner involves unique challenges. This document describes the core concepts and design patterns used to execute exactly-once processing and at-most-once acknowledgment in Spanner queues.

## Core concepts and the idempotency dilemma

"Idempotency" means that no matter how many times an operation is executed, the result is always the same. When building message queues or work delivery systems, there are three primary delivery methods that address idempotency:

  - **At-least-once:** Messages are guaranteed to be delivered and processed. If network failures occur, a message might be delivered and processed multiple times.
  - **At-most-once:** Messages are delivered at most once. Duplicate processing is prevented, but if a failure occurs, the message might be lost or dropped.
  - **Exactly-once:** Every message is processed exactly once—neither lost nor duplicated.

Spanner queues automatically provide **at-least-once delivery** and **at-most-once acknowledgment** . However, you can use the design patterns and examples described in the following sections to programmatically reduce duplicate processing and redelivery.

### The unknown commit status problem

In a single-machine database, transactions either succeed or fail. In a distributed cloud database, transactions can be lost as data is replicated across multiple physical data centers.

When your application processes a message and commits an acknowledgment such as `DELETE FROM MessageQueue WHERE message_id = '123' ASSERT_ROWS_MODIFIED 1` , the request travels across multiple network hops:

    [Your App] <--(gRPC)--> [Spanner API] <--(Google network)--> [Spanner Storage]

When the Spanner storage leader receives the commit, it synchronizes across storage replicas. After consensus is reached and data is written to disk, the transaction successfully commits in the database.

However, a transient network failure (such as a proxy reboot or a network disconnect) might occur after the data commits to disk, but before the success confirmation reaches your application. If this happens, your application receives a network timeout error ( `DEADLINE_EXCEEDED` or `UNAVAILABLE` ).

Because the connection broke out-of-band, your application has no way of knowing whether the connection broke before or after the commit succeeded. This is known as the "unknown commit status" problem.

### Why automatic retries can be dangerous

If your application intercepts a network timeout error and retries the exact same transaction without deduplication, it risks executing the business logic twice. For example, if the transaction involved charging a customer's credit card or shipping an item, retrying a successful (but unconfirmed) transaction results in a duplicate charge or shipment.

To protect your users, official Google Cloud client libraries never automatically retry transactions that fail with unknown commit errors. They throw an explicit error alerting you that the transaction outcome is unknown, leaving it up to your application code to handle the retry safely using idempotency patterns.

## Real-world strategies for exactly-once processing

To safely retry transactions and reduce duplicate processing and message redelivery, implement one of the following primary design patterns.

### Assert rows modified

If all of your business logic (such as updating other tables) and the queue message acknowledgment occur in a single Spanner transaction, you can use `ASSERT_ROWS_MODIFIED` on your `DELETE` statement. This is the simplest strategy for exactly-once processing because it doesn't require a separate deduplication table.

By appending `ASSERT_ROWS_MODIFIED 1` to your statement, the acknowledgment only succeeds if exactly one message was deleted. If a network failure occurs and your application retries the transaction, the message no longer exists in the queue. The retry deletes 0 rows and the statement fails with `OUT_OF_RANGE` .

The `OUT_OF_RANGE` error is permanent: the row is already gone, so re-executing the statement will always modify 0 rows and fail again. It is also statement-level: only the `DELETE` fails. The transaction stays open, and any writes you have already made in it are still buffered and will be committed if you proceed. Spanner does not abort or roll back the transaction for you. To get the intended exactly-once protection, you must ensure the transaction is rolled back:

  - **Let the error propagate:** If you use a runner-based API (like `ReadWriteTransaction` in Go), let the `OUT_OF_RANGE` error propagate out of your transaction function. The client library will abandon the transaction and roll it back, ensuring no business-logic writes are committed.
  - **Do not catch and continue:** Never catch the assertion error inside the transaction function and continue. If you do, Spanner will still commit your other statements, causing the exact duplicate update this pattern is meant to prevent.
  - **Manual rollbacks:** If you manage transactions manually (for example, with `ReadWriteStmtBasedTransaction` in Go, or the REST/gRPC APIs), you must explicitly call the corresponding rollback method when the assertion fails.

### Transactional outbox

If your business logic cannot occur within a single Spanner transaction (such as multi-step workflows or updates spanning multiple systems), or if message processing requires calling external services (such as sending an SMS or processing a payment), you cannot atomically couple the business logic with the queue acknowledgment.

In these scenarios, never call external APIs or execute non-transactional operations directly inside a Spanner transaction block. If Spanner retries or aborts the transaction, your application might execute that business logic multiple times.

Your application should implement its own idempotency strategy, but you can use the following patterns to reduce duplicate work:

  - **Fast operations (completes within the 10-second default lease):** Execute the business logic or external API call when the message is received. Acknowledge the message only after the operation succeeds. If the operation fails, or if the worker crashes before acknowledging, don't acknowledge the message; the lease expires and Spanner automatically delivers the message for retry.

  - **Long-running operations (takes longer than 10 seconds):** Acknowledge the message when it is received, and within the same transaction, resend a new message scheduled for future delivery (using a delivery delay greater than the estimated execution time). Alternatively, periodically extend the message lease using the `RENEWLEASE_ QUEUE_NAME ()` TVF. When the business logic or external call succeeds, acknowledge the newly enqueued or extended message.

## Multiplexed session idempotency

Spanner sessions can be [multiplexed](https://docs.cloud.google.com/spanner/docs/sessions#multiplexed_sessions) . Multiplexed sessions track transaction states in memory across shared sessions. This protects against client-side network failures by allowing client libraries to reconnect and resolve unknown commit statuses more reliably.

However, multiplexed sessions cannot eliminate unknown commit outcomes if a frontend server crashes. If the specific server hosting your session's transaction table reboots immediately after a commit, the in-memory state is lost, returning an unknown transaction outcome error upon reconnection. To address this risk, implement the [idempotency patterns](https://docs.cloud.google.com/spanner/docs/queues/queues-at-most-once#strategies) on this page.

## What's next

  - Learn how to [use Spanner queues, including best practices and monitoring](https://docs.cloud.google.com/spanner/docs/queues/queues-using) .
  - Explore more [Spanner queues scenarios and examples](https://docs.cloud.google.com/spanner/docs/queues/queues-examples) .
  - Configure access control with [fine-grained access control for queues](https://docs.cloud.google.com/spanner/docs/fgac-queues) .
