---
name: documents/docs.cloud.google.com/spanner/docs/transaction-timeout
uri: https://docs.cloud.google.com/spanner/docs/transaction-timeout
title: Configure transaction timeout
description: Configure transaction timeouts in Spanner to prevent transactions from holding locks for too long, improving concurrency.
data_source: docs.cloud.google.com
---

This page describes how to set a timeout for transactions by using the Spanner client libraries. The transaction fails with a `DEADLINE_EXCEEDED` error if the transaction can't finish within the given timeout value.

If the timeout error occurs during the execution of the [`Commit`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Commit) request, it's still possible that the transaction was committed.

## RPC timeouts and transaction retry loops

RPC timeouts and transaction timeouts limit different scopes of work:

- An RPC timeout limits how long a single request, such as a query, a read, or a `Commit` request, can run. A transaction timeout that is longer than an RPC timeout doesn't extend the RPC timeout. For more information, see [Configure custom timeouts and retries](https://docs.cloud.google.com/spanner/docs/custom-timeout-and-retry) .
- A transaction timeout limits the total time that the client library spends running a read-write transaction, including every attempt that the client library makes after Spanner aborts the transaction.

When Spanner aborts a read-write transaction, the client library reruns the transaction function. Short RPC timeouts don't limit how long the client library keeps retrying an aborted transaction. If you don't set a transaction timeout, the default retry behavior depends on the client library:

- **Java** : The transaction runner retries aborted transactions for up to 24 hours, with no limit on the number of attempts.
- **Go** : The client library retries aborted transactions until the transaction commits, or until the `context.Context` object that you pass to the transaction reaches its deadline or is canceled.
- **Node.js** : The client library stops retrying aborted transactions after 1 hour ( `3600000` milliseconds). To change this limit, set the `timeout` field in the `RunTransactionOptions` object that you pass to the `runTransaction` or `runTransactionAsync` method.
- **Python** : The client library stops retrying aborted transactions after 30 seconds. To change this limit, pass the `timeout_secs` argument to the `run_in_transaction` method.

In workloads with high lock contention, every retry attempt competes for the same locks, so a long retry loop can increase lock wait times for other transactions. To limit the total time that a transaction can run, set a transaction timeout as shown in the samples on this page.

## Set a transaction timeout

You can set a transaction timeout by using the Go, Java, Python, and Node.js client libraries.

### Go

```
import (
    "context"
    "errors"
    "fmt"
    "io"
    "time"

    "cloud.google.com/go/spanner"
    "google.golang.org/api/iterator"
    "google.golang.org/grpc/codes"
)

func transactionTimeout(w io.Writer, db string) error {
    ctx := context.Background()
    client, err := spanner.NewClient(ctx, db)
    if err != nil {
        return err
    }
    defer client.Close()

    // Create a context with a 60-second timeout and use this context to run a read/write transaction.
    // This context timeout will be applied to the entire transaction, and the transaction will fail
    // if it cannot finish within the specified timeout value. The Spanner client library applies the
    // (remainder of the) timeout to each statement that is executed in the transaction.
    ctx, cancel := context.WithTimeout(context.Background(), 60*time.Second)
    defer cancel()

    _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
        selectStmt := spanner.Statement{
            SQL: `SELECT SingerId, FirstName, LastName FROM Singers ORDER BY LastName, FirstName`,
        }
        // The context that is passed in to the transaction function should be used for each statement
        // is executed.
        iter := txn.Query(ctx, selectStmt)
        defer iter.Stop()
        for {
            row, err := iter.Next()
            if errors.Is(err, iterator.Done) {
                break
            }
            if err != nil {
                return err
            }
            var singerID int64
            var firstName, lastName string
            if err := row.Columns(&singerID, &firstName, &lastName); err != nil {
                return err
            }
            fmt.Fprintf(w, "%d %s %s\n", singerID, firstName, lastName)
        }
        stmt := spanner.Statement{
            SQL: `INSERT INTO Singers (SingerId, FirstName, LastName)
                    VALUES (38, 'George', 'Washington')`,
        }
        rowCount, err := txn.Update(ctx, stmt)
        if err != nil {
            return err
        }
        fmt.Fprintf(w, "%d record(s) inserted.\n", rowCount)
        return nil
    })
    // Check if an error was returned by the transaction.
    // The spanner.ErrCode(err) function will return codes.OK if err == nil.
    code := spanner.ErrCode(err)
    if code == codes.OK {
        fmt.Fprintf(w, "Transaction with timeout was executed successfully\n")
    } else if code == codes.DeadlineExceeded {
        fmt.Fprintf(w, "Transaction timed out\n")
    } else {
        fmt.Fprintf(w, "Transaction failed with error code %v\n", code)
    }
    return err
}
```

### Java

```
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.ResultSet;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.Statement;
import io.grpc.Context;
import io.grpc.Context.CancellableContext;
import io.grpc.Deadline;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * Sample showing how to set a timeout for an entire transaction for the Cloud Spanner Java client.
 */
class TransactionTimeoutExample {

  static void executeTransactionWithTimeout() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";

    executeTransactionWithTimeout(projectId, instanceId, databaseId, 60L, TimeUnit.SECONDS);
  }

  // Execute a read/write transaction with a timeout for the entire transaction.
  static void executeTransactionWithTimeout(
      String projectId,
      String instanceId,
      String databaseId,
      long timeoutValue,
      TimeUnit timeoutUnit) {
    try (Spanner spanner = SpannerOptions.newBuilder().setProjectId(projectId).build()
        .getService()) {
      DatabaseClient client =
          spanner.getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));
      // Create a gRPC context with a deadline and with cancellation.
      // gRPC context deadlines require the use of a scheduled executor.
      ScheduledExecutorService executor = Executors.newSingleThreadScheduledExecutor();
      try (CancellableContext context =
          Context.current()
              .withDeadline(Deadline.after(timeoutValue, timeoutUnit), executor)
              .withCancellation()) {
        context.run(
            () -> {
              client
                  .readWriteTransaction()
                  .run(
                      transaction -> {
                        try (ResultSet resultSet =
                            transaction.executeQuery(
                                Statement.of(
                                    "SELECT SingerId, FirstName, LastName\n"
                                        + "FROM Singers\n"
                                        + "ORDER BY LastName, FirstName"))) {
                          while (resultSet.next()) {
                            System.out.printf(
                                "%d %s %s\n",
                                resultSet.getLong("SingerId"),
                                resultSet.getString("FirstName"),
                                resultSet.getString("LastName"));
                          }
                        }
                        String sql =
                            "INSERT INTO Singers (SingerId, FirstName, LastName)\n"
                                + "VALUES (20, 'George', 'Washington')";
                        long rowCount = transaction.executeUpdate(Statement.of(sql));
                        System.out.printf("%d record inserted.%n", rowCount);
                        return null;
                      });
            });
      }
    }
  }
}
```

### Node.js

```
/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

async function executeTransactionWithTimeout() {
  // Gets a reference to a Cloud Spanner instance and database.
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);

  const options = {
    timeout: 60000, // 60 seconds timeout
  };

  try {
    await database.runTransactionAsync(options, async tx => {
      const [results] = await tx.run(
        'SELECT SingerId, FirstName, LastName FROM Singers ORDER BY LastName, FirstName',
      );
      results.forEach(result => {
        console.log(
          `${result[0].name}: ${result[0].value.value}, ${result[1].name}: ${result[1].value}, ${result[2].name}: ${result[2].value}`,
        );
      });
      const sql =
        "INSERT INTO Singers (SingerId, FirstName, LastName) VALUES (100, 'George', 'Washington')";
      const [rowCount] = await tx.runUpdate(sql);
      console.log(`${rowCount} record inserted.`);
      await tx.commit();
    });
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    await database.close();
  }
}
executeTransactionWithTimeout();
```

### Python

```
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

def read_then_write(transaction):
    # Read records.
    results = transaction.execute_sql(
        "SELECT SingerId, FirstName, LastName FROM Singers ORDER BY LastName, FirstName"
    )
    for result in results:
        print("SingerId: {}, FirstName: {}, LastName: {}".format(*result))

    # Insert a record.
    row_ct = transaction.execute_update(
        "INSERT INTO Singers (SingerId, FirstName, LastName) "
        " VALUES (100, 'George', 'Washington')"
    )
    print("{} record(s) inserted.".format(row_ct))

# configure transaction timeout to 60 seconds
database.run_in_transaction(read_then_write, timeout_secs=60)
```

### Set a timeout for asynchronous Java transactions

The Java sample on this page uses a gRPC `Context` object with a deadline to set the transaction timeout. The gRPC `Context` object is attached to the current thread.

When you run a transaction asynchronously by using the `AsyncRunner.runAsync()` method, the client library runs the transaction on the `Executor` object that you provide. The client library doesn't propagate the calling thread's `Context` object, including its deadline, to that executor. As a result, the transaction runs without the timeout and can keep retrying aborted attempts for up to 24 hours.

To apply the timeout to an asynchronous transaction, do one of the following:

- Wrap the executor by calling the `Context.currentContextExecutor(executor)` method while the `Context` object that has the deadline is current, and pass the wrapped executor to the `runAsync()` method.
- Use the `AsyncTransactionManager` or `TransactionManager` interface instead of the `AsyncRunner` interface. With a transaction manager, you write the retry loop yourself and can stop retrying when your timeout expires.
