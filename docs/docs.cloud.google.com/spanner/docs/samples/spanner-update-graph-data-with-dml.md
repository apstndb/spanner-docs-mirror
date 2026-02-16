Update data in a Spanner Graph using DML.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Manage Spanner Graph data](/spanner/docs/graph/insert-update-delete-data)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void UpdateDataWithDml(google::cloud::spanner::Client client) {
  using ::google::cloud::StatusOr;
  namespace spanner = ::google::cloud::spanner;

  auto commit_result = client.Commit(
      [&client](spanner::Transaction txn) -> StatusOr<spanner::Mutations> {
        auto update = client.ExecuteDml(
            std::move(txn),
            spanner::SqlStatement(
                "UPDATE Account SET is_blocked = false WHERE id = 2"));
        if (!update) return std::move(update).status();
        return spanner::Mutations{};
      });
  if (!commit_result) throw std::move(commit_result).status();

  commit_result = client.Commit(
      [&client](spanner::Transaction txn) -> StatusOr<spanner::Mutations> {
        auto update =
            client.ExecuteDml(std::move(txn), spanner::SqlStatement(R"""(
          UPDATE AccountTransferAccount
            SET amount = 300 WHERE id = 1 AND to_id = 2)"""));
        if (!update) return std::move(update).status();
        return spanner::Mutations{};
      });
  if (!commit_result) throw std::move(commit_result).status();

  std::cout << "Update was successful [spanner_update_graph_data_with_dml]\n";
}
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
)

func updateGraphDataWithDml(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 // Execute a ReadWriteTransaction to update the 'Account' table underpinning
 // 'Account' nodes in 'FinGraph'. The function run by ReadWriteTransaction
 // executes an 'UPDATE' SQL DML statement. Graph queries run after this
 // transaction is committed will observe the effects of the update to 'Account'
 // with 'id' = 2.
 _, err1 := client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `UPDATE Account SET is_blocked = false WHERE id = 2`,
     }
     rowCount, err := txn.Update(ctx, stmt)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%d Account record(s) updated.\n", rowCount)
     return err
 })

 if err1 != nil {
     return err1
 }

 // Execute a ReadWriteTransaction to update the 'AccountTransferAccount' table
 // underpinning 'AccountTransferAccount' edges in 'FinGraph'. The function run
 // by ReadWriteTransaction executes an 'UPDATE' SQL DML statement.
 // Graph queries run after this transaction is committed will observe the effects
 // of the update to 'AccountTransferAccount' where the source of the transfer has
 // 'id' 1 and the destination has 'id' 2.
 _, err2 := client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `UPDATE AccountTransferAccount SET amount = 300 WHERE id = 1 AND to_id = 2`,
     }
     rowCount, err := txn.Update(ctx, stmt)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%d AccountTransferAccount record(s) updated.\n", rowCount)
     return err
 })

 return err2
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void updateUsingDml(DatabaseClient dbClient) {
  dbClient
      .readWriteTransaction()
      .run(
          transaction -> {
            String sql = "UPDATE Account SET is_blocked = false WHERE id = 2";
            long rowCount = transaction.executeUpdate(Statement.of(sql));
            System.out.printf("%d Account record(s) updated.\n", rowCount);
            return null;
          });

  dbClient
      .readWriteTransaction()
      .run(
          transaction -> {
            String sql =
                "UPDATE AccountTransferAccount SET amount = 300 WHERE id = 1 AND to_id = 2";
            long rowCount = transaction.executeUpdate(Statement.of(sql));
            System.out.printf("%d AccountTransferAccount record(s) updated.\n", rowCount);
            return null;
          });
}
```

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def update_data_with_dml(instance_id, database_id):
    """Updates sample data from the database using a DML statement."""

    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    def update_accounts(transaction):
        row_ct = transaction.execute_update(
            "UPDATE Account SET is_blocked = false WHERE id = 2"
        )

        print("{} Account record(s) updated.".format(row_ct))

    def update_transfers(transaction):
        row_ct = transaction.execute_update(
            "UPDATE AccountTransferAccount SET amount = 300 WHERE id = 1 AND to_id = 2"
        )

        print("{} AccountTransferAccount record(s) updated.".format(row_ct))

    database.run_in_transaction(update_accounts)
    database.run_in_transaction(update_transfers)
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
