Delete Spanner Graph data using DML.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Manage Spanner Graph data](/spanner/docs/graph/insert-update-delete-data)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void DeleteDataWithDml(google::cloud::spanner::Client client) {
  using ::google::cloud::StatusOr;
  namespace spanner = ::google::cloud::spanner;

  auto commit_result = client.Commit([&client](spanner::Transaction txn)
                                         -> StatusOr<spanner::Mutations> {
    auto deleted = client.ExecuteDml(
        std::move(txn),
        spanner::SqlStatement(
            "DELETE FROM AccountTransferAccount WHERE id = 1 AND to_id = 2"));
    if (!deleted) return std::move(deleted).status();
    return spanner::Mutations{};
  });
  if (!commit_result) throw std::move(commit_result).status();

  commit_result = client.Commit(
      [&client](spanner::Transaction txn) -> StatusOr<spanner::Mutations> {
        auto deleted = client.ExecuteDml(
            std::move(txn),
            spanner::SqlStatement("DELETE FROM Account WHERE id = 2"));
        if (!deleted) return std::move(deleted).status();
        return spanner::Mutations{};
      });
  if (!commit_result) throw std::move(commit_result).status();

  std::cout << "Delete was successful [spanner_delete_graph_data_with_dml]\n";
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

func deleteGraphDataWithDml(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 // Execute a ReadWriteTransaction to update the 'AccountTransferAccount'
 // table underpinning 'AccountTransferAccount' edges in 'FinGraph'. The
 // function run by ReadWriteTransaction executes an 'DELETE' SQL DML
 // statement. This has the effect of deleting the 'AccountTransferAccount'
 // edge where the source 'id' is 1 and the destination 'id' is 2 from the graph.
 _, err1 := client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{SQL: `DELETE FROM AccountTransferAccount WHERE id = 1 AND to_id = 2`}
     rowCount, err := txn.Update(ctx, stmt)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%d AccountTransferAccount record(s) deleted.\n", rowCount)
     return nil
 })

 if err1 != nil {
     return err1
 }

 // Execute a ReadWriteTransaction to update the 'Account' table underpinning
 //'Account' nodes in 'FinGraph'. In 'FinGraph', nodes can only be deleted
 // after any edges referencing the nodes have been deleted first. The function
 // run by ReadWriteTransaction executes an 'DELETE' SQL DML statement. This has
 // the effect of deleting the 'Account' node whose 'id' is 1 from the graph.
 _, err2 := client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{SQL: `DELETE FROM Account WHERE id = 2`}
     rowCount, err := txn.Update(ctx, stmt)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%d Account record(s) deleted.\n", rowCount)
     return nil
 })

 return err2
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void deleteUsingDml(DatabaseClient dbClient) {
  dbClient
      .readWriteTransaction()
      .run(
          transaction -> {
            String sql = "DELETE FROM AccountTransferAccount WHERE id = 1 AND to_id = 2";
            long rowCount = transaction.executeUpdate(Statement.of(sql));
            System.out.printf("%d AccountTransferAccount record(s) deleted.\n", rowCount);
            return null;
          });

  dbClient
      .readWriteTransaction()
      .run(
          transaction -> {
            String sql = "DELETE FROM Account WHERE id = 2";
            long rowCount = transaction.executeUpdate(Statement.of(sql));
            System.out.printf("%d Account record(s) deleted.\n", rowCount);
            return null;
          });
}
```

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def delete_data_with_dml(instance_id, database_id):
    """Deletes sample data from the database using a DML statement."""

    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    def delete_transfers(transaction):
        row_ct = transaction.execute_update(
            "DELETE FROM AccountTransferAccount WHERE id = 1 AND to_id = 2"
        )

        print("{} AccountTransferAccount record(s) deleted.".format(row_ct))

    def delete_accounts(transaction):
        row_ct = transaction.execute_update("DELETE FROM Account WHERE id = 2")

        print("{} Account record(s) deleted.".format(row_ct))

    database.run_in_transaction(delete_transfers)
    database.run_in_transaction(delete_accounts)
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
