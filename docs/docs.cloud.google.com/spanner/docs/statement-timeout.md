This page describes how to set a timeout for a single statement execution using the Spanner client libraries. This can be used to override the default timeout configuration of the client library. The statement fails with a `  DEADLINE_EXCEEDED  ` error if the statement cannot finish within the given timeout value.

**Note:** It is possible that the timeout occurs after the statement has finished executing on Spanner, but before the response reaches the client. In that case, it's possible that the statement was committed to Spanner successfully.

These samples show how to set a timeout for a single statement execution in the Cloud Spanner client library.

### Go

``` go
import (
 "context"
 "fmt"
 "io"
 "time"

 "cloud.google.com/go/spanner"
 "google.golang.org/grpc/codes"
)

func setStatementTimeout(w io.Writer, db string) error {
 client, err := spanner.NewClient(context.Background(), db)
 if err != nil {
     return err
 }
 defer client.Close()

 _, err = client.ReadWriteTransaction(context.Background(),
     func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
         // Create a context with a 60-second timeout and apply this timeout to the insert statement.
         ctxWithTimeout, cancel := context.WithTimeout(context.Background(), 60*time.Second)
         defer cancel()
         stmt := spanner.Statement{
             SQL: `INSERT Singers (SingerId, FirstName, LastName)
                 VALUES (39, 'George', 'Washington')`,
         }
         rowCount, err := txn.Update(ctxWithTimeout, stmt)
         // Get the error code from the error. This function returns codes.OK if err == nil.
         code := spanner.ErrCode(err)
         if code == codes.DeadlineExceeded {
             fmt.Fprintf(w, "Insert statement timed out.\n")
         } else if code == codes.OK {
             fmt.Fprintf(w, "%d record(s) inserted.\n", rowCount)
         } else {
             fmt.Fprintf(w, "Insert statement failed with error %v\n", err)
         }
         return err
     })
 if err != nil {
     return err
 }
 return nil
}
```

### Java

``` java
static void executeSqlWithTimeout() {
  // TODO(developer): Replace these variables before running the sample.
  String projectId = "my-project";
  String instanceId = "my-instance";
  String databaseId = "my-database";

  try (Spanner spanner =
      SpannerOptions.newBuilder().setProjectId(projectId).build().getService()) {
    DatabaseClient client =
        spanner.getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));
    executeSqlWithTimeout(client);
  }
}

static void executeSqlWithTimeout(DatabaseClient client) {
  CallContextConfigurator configurator = new CallContextConfigurator() {
    public <ReqT, RespT> ApiCallContext configure(ApiCallContext context, ReqT request,
        MethodDescriptor<ReqT, RespT> method) {
      // DML uses the ExecuteSql RPC.
      if (method == SpannerGrpc.getExecuteSqlMethod()) {
        // NOTE: You can use a GrpcCallContext to set a custom timeout for a single RPC
        // invocation. This timeout can however ONLY BE SHORTER than the default timeout
        // for the RPC. If you set a timeout that is longer than the default timeout, then
        // the default timeout will be used.
        return GrpcCallContext.createDefault()
            .withCallOptions(CallOptions.DEFAULT.withDeadlineAfter(60L, TimeUnit.SECONDS));
      }
      // Return null to indicate that the default should be used for other methods.
      return null;
    }
  };
  // Create a context that uses the custom call configuration.
  Context context =
      Context.current().withValue(SpannerOptions.CALL_CONTEXT_CONFIGURATOR_KEY, configurator);
  // Run the transaction in the custom context.
  context.run(() ->
      client.readWriteTransaction().<long[]>run(transaction -> {
        String sql = "INSERT INTO Singers (SingerId, FirstName, LastName)\n"
            + "VALUES (20, 'George', 'Washington')";
        long rowCount = transaction.executeUpdate(Statement.of(sql));
        System.out.printf("%d record inserted.%n", rowCount);
        return null;
      })
  );
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

async function executeSqlWithTimeout() {
  // Gets a reference to a Cloud Spanner instance and database.
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);

  try {
    await database.runTransactionAsync(async tx => {
      // NOTE: You can use gaxOptions to set a custom timeout for a single RPC
      // invocation. This timeout can however ONLY BE SHORTER than the default timeout
      // for the RPC. If you set a timeout that is longer than the default timeout, then
      // the default timeout will be used.
      const query = {
        sql: "INSERT INTO Singers (SingerId, FirstName, LastName) VALUES (110, 'George', 'Washington')",
        gaxOptions: {
          timeout: 60000, // 60 seconds timeout
        },
      };
      const results = await tx.run(query);
      console.log(`${results[1].rowCountExact} record inserted.`);
      await tx.commit();
    });
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    await database.close();
  }
}
executeSqlWithTimeout();
```

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

def write(transaction):
    # Insert a record and configure the statement timeout to 60 seconds
    # This timeout can however ONLY BE SHORTER than the default timeout
    # for the RPC. If you set a timeout that is longer than the default timeout,
    # then the default timeout will be used.
    row_ct = transaction.execute_update(
        "INSERT INTO Singers (SingerId, FirstName, LastName) "
        " VALUES (110, 'George', 'Washington')",
        timeout=60,
    )
    print("{} record(s) inserted.".format(row_ct))

database.run_in_transaction(write)
```
