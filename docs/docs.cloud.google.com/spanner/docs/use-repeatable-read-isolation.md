**Preview — Repeatable read isolation**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

This page describes how to use repeatable read isolation in Spanner.

Repeatable read is an [isolation level](/spanner/docs/isolation-levels) that ensures that all read operations within a transaction see a consistent snapshot of the database as it existed at the start of the transaction. In Spanner, this isolation level is implemented using a technique that is also commonly called snapshot isolation. This approach is beneficial in high read-write concurrency scenarios where numerous transactions read data that other transactions might be modifying. By using a fixed snapshot, repeatable read avoids the performance impacts of the more rigorous serializable isolation level. Reads can execute without acquiring locks and without blocking concurrent writes, which results in potentially fewer aborted transactions that might need to be retried due to serialization conflicts. For more information, see [Isolation level overview](/spanner/docs/isolation-levels) .

## Set the isolation level

You can set the isolation level on read-write transactions at the database client-level or the transaction-level using the following methods:

### Client libraries

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
 pb "cloud.google.com/go/spanner/apiv1/spannerpb"
)

func writeWithTransactionUsingIsolationLevel(w io.Writer, db string) error {
 ctx := context.Background()

 // The isolation level specified at the client-level will be applied
 // to all RW transactions.
 cfg := spanner.ClientConfig{
     TransactionOptions: spanner.TransactionOptions{
         IsolationLevel: pb.TransactionOptions_SERIALIZABLE,
     },
 }
 client, err := spanner.NewClientWithConfig(ctx, db, cfg)
 if err != nil {
     return fmt.Errorf("failed to create client: %w", err)
 }
 defer client.Close()

 // The isolation level specified at the transaction-level takes
 // precedence over the isolation level configured at the client-level.
 // REPEATABLE_READ is used here to demonstrate overriding the client-level setting.
 txnOpts := spanner.TransactionOptions{
     IsolationLevel: pb.TransactionOptions_REPEATABLE_READ,
 }

 _, err = client.ReadWriteTransactionWithOptions(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     // Read the current album title
     key := spanner.Key{1, 1}
     row, err := txn.ReadRow(ctx, "Albums", key, []string{"AlbumTitle"})
     if err != nil {
         return fmt.Errorf("failed to read album: %v", err)
     }
     var title string
     if err := row.Column(0, &title); err != nil {
         return fmt.Errorf("failed to get album title: %v", err)
     }
     fmt.Fprintf(w, "Current album title: %s\n", title)

     // Update the album title
     stmt := spanner.Statement{
         SQL: `UPDATE Albums
             SET AlbumTitle = @AlbumTitle
             WHERE SingerId = @SingerId AND AlbumId = @AlbumId`,
         Params: map[string]interface{}{
             "SingerId":   1,
             "AlbumId":    1,
             "AlbumTitle": "New Album Title",
         },
     }
     count, err := txn.Update(ctx, stmt)
     if err != nil {
         return fmt.Errorf("failed to update album: %v", err)
     }
     fmt.Fprintf(w, "Updated %d record(s).\n", count)
     return nil
 }, txnOpts)

 if err != nil {
     return fmt.Errorf("transaction failed: %v", err)
 }
 return nil
}
```

### Java

``` java
static void isolationLevelSetting(DatabaseId db) {
  // The isolation level specified at the client-level will be applied to all
  // RW transactions.
  DefaultReadWriteTransactionOptions transactionOptions =
      DefaultReadWriteTransactionOptions.newBuilder()
          .setIsolationLevel(IsolationLevel.SERIALIZABLE)
          .build();
  SpannerOptions options =
      SpannerOptions.newBuilder()
          .setDefaultTransactionOptions(transactionOptions)
          .build();
  Spanner spanner = options.getService();
  DatabaseClient dbClient = spanner.getDatabaseClient(db);
  dbClient
      // The isolation level specified at the transaction-level takes precedence
      // over the isolation level configured at the client-level.
      .readWriteTransaction(Options.isolationLevel(IsolationLevel.REPEATABLE_READ))
      .run(transaction -> {
        // Read an AlbumTitle.
        String selectSql =
            "SELECT AlbumTitle from Albums WHERE SingerId = 1 and AlbumId = 1";
        ResultSet resultSet = transaction.executeQuery(Statement.of(selectSql));
        String title = null;
        while (resultSet.next()) {
          title = resultSet.getString("AlbumTitle");
        }
        System.out.printf("Current album title: %s\n", title);

        // Update the title.
        String updateSql =
            "UPDATE Albums "
                + "SET AlbumTitle = 'New Album Title' "
                + "WHERE SingerId = 1 and AlbumId = 1";
        long rowCount = transaction.executeUpdate(Statement.of(updateSql));
        System.out.printf("%d record updated.\n", rowCount);
        return null;
      });
}
```

### Node.js

``` javascript
// Imports the Google Cloud Spanner client library
const {Spanner, protos} = require('@google-cloud/spanner');
// The isolation level specified at the client-level will be applied
// to all RW transactions.
const defaultTransactionOptions = {
  isolationLevel:
    protos.google.spanner.v1.TransactionOptions.IsolationLevel.SERIALIZABLE,
};

// Instantiates a client with defaultTransactionOptions
const spanner = new Spanner({
  projectId: projectId,
  defaultTransactionOptions,
});

function runTransactionWithIsolationLevel() {
  // Gets a reference to a Cloud Spanner instance and database
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);
  // The isolation level specified at the request level takes precedence over the isolation level configured at the client level.
  const isolationOptionsForTransaction = {
    isolationLevel:
      protos.google.spanner.v1.TransactionOptions.IsolationLevel
        .REPEATABLE_READ,
  };

  database.runTransaction(
    isolationOptionsForTransaction,
    async (err, transaction) => {
      if (err) {
        console.error(err);
        return;
      }
      try {
        const query =
          'SELECT AlbumTitle FROM Albums WHERE SingerId = 1 AND AlbumId = 1';
        const results = await transaction.run(query);
        // Gets first album's title
        const rows = results[0].map(row => row.toJSON());
        const albumTitle = rows[0].AlbumTitle;
        console.log(`previous album title ${albumTitle}`);

        const update =
          "UPDATE Albums SET AlbumTitle = 'New Album Title' WHERE SingerId = 1 AND AlbumId = 1";
        const [rowCount] = await transaction.runUpdate(update);
        console.log(
          `Successfully updated ${rowCount} record in Albums table.`,
        );
        await transaction.commit();
        console.log(
          'Successfully executed read-write transaction with isolationLevel option.',
        );
      } catch (err) {
        console.error('ERROR:', err);
        transaction.end();
      } finally {
        // Close the database when finished.
        await database.close();
      }
    },
  );
}
runTransactionWithIsolationLevel();
```

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"

# The isolation level specified at the client-level will be applied to all RW transactions.
isolation_options_for_client = TransactionOptions.IsolationLevel.SERIALIZABLE

spanner_client = spanner.Client(
    default_transaction_options=DefaultTransactionOptions(
        isolation_level=isolation_options_for_client
    )
)
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

# The isolation level specified at the request level takes precedence over the isolation level configured at the client level.
isolation_options_for_transaction = (
    TransactionOptions.IsolationLevel.REPEATABLE_READ
)

def update_albums_with_isolation(transaction):
    # Read an AlbumTitle.
    results = transaction.execute_sql(
        "SELECT AlbumTitle from Albums WHERE SingerId = 1 and AlbumId = 1"
    )
    for result in results:
        print("Current Album Title: {}".format(*result))

    # Update the AlbumTitle.
    row_ct = transaction.execute_update(
        "UPDATE Albums SET AlbumTitle = 'A New Title' WHERE SingerId = 1 and AlbumId = 1"
    )

    print("{} record(s) updated.".format(row_ct))

database.run_in_transaction(
    update_albums_with_isolation, isolation_level=isolation_options_for_transaction
)
```

### REST

You can use the [`  TransactionOptions.isolation_level  `](/spanner/docs/reference/rest/v1/TransactionOptions#isolationlevel) REST API to set the isolation level on read-write and read-only transactions at the transaction-level. The valid options are `  TransactionOptions.SERIALIZABLE  ` and `  TransactionOptions.REPEATABLE_READ  ` . By default, Spanner sets the isolation level to serializable isolation.

## Limitations

The following set of limitations exist in the repeatable read isolation Preview.

  - You might experience issues if your schema has [check constraints](/spanner/docs/check-constraint/how-to) .
      - There is a known issue that prevents check constraints from being validated, which can result in constraint violations when transactions commit. Therefore we don't recommend using repeatable read isolation in Preview if your schema has check constraints.
  - You might experience issues if concurrent schema changes occur in your database while transactions are executing.
      - If your DML statements use the [`  last_statement  ` option](/spanner/docs/dml-best-practices#use-last-statement) and a concurrent schema change occurs while the DML statement executes, it might internally retry and return an error stating that the DML was retried incorrectly after the `  last_statement  ` option was set. Retrying the transaction after the schema change applies resolves this issue.
      - If requests in a transaction experience a `  DEADLINE_EXCEEDED  ` error from the client, retry the transaction after the schema change applies to resolve the issue.

## Unsupported use cases

  - You can't set repeatable read isolation on partitioned DML transactions.
  - All read-only transactions already operate at a fixed snapshot and don't require locks, so setting repeatable read isolation in this transaction type doesn't change any behavior.
  - You can't set repeatable read isolation on read-only, single-use, and partition operations using the Spanner client libraries Spanner client libraries won't have the option to set the repeatable read isolation on read-only, single-use and partition query operations.

## What's next

  - Learn more about [isolation levels](/spanner/docs/isolation-levels) .

  - Learn how to [use SELECT FOR UPDATE in repeatable read isolation](/spanner/docs/use-select-for-update-repeatable-read) .

  - Learn more about Spanner serializability and external consistency, see [TrueTime and external consistency](/spanner/docs/true-time-external-consistency) .
