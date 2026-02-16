This page describes how to set a timeout for transactions using the Spanner client libraries. The transaction fails with a `  DEADLINE_EXCEEDED  ` error if the transaction can't finish within the given timeout value.

You can set timeout values for transactions and for [RPC request statements](/spanner/docs/custom-timeout-and-retry) . Setting a longer timeout value for the transaction than the timeout value for the statement that is executed in the transaction doesn't increase the timeout for the statement, which is constrained by its own timeout value.

Also, if the timeout error occurs during the execution of the [`  Commit  `](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Commit) request, it's still possible that the transaction was committed.

You can set a transaction timeout using the Go, Java, Python, and Node.js client libraries.

### Go

``` go
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

``` java
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

``` javascript
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

``` python
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
