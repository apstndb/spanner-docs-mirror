Update data in a Spanner Graph using a graph query.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Manage Spanner Graph data](/spanner/docs/graph/insert-update-delete-data)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void UpdateDataWithGraphQueryInDml(google::cloud::spanner::Client client) {
  using ::google::cloud::StatusOr;
  namespace spanner = ::google::cloud::spanner;
  auto commit_result = client.Commit(
      [&client](spanner::Transaction txn) -> StatusOr<spanner::Mutations> {
        auto update =
            client.ExecuteDml(std::move(txn), spanner::SqlStatement(R"""(
              UPDATE Account SET is_blocked = true
              WHERE id IN {
                GRAPH FinGraph
                MATCH (a:Account WHERE a.id = 1)-[:TRANSFERS]->{1,2}(b:Account)
                RETURN b.id})"""));
        if (!update) return std::move(update).status();
        return spanner::Mutations{};
      });
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << "Update was successful "
            << "[spanner_update_graph_data_with_graph_query_in_dml]\n";
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

func updateGraphDataWithGraphQueryInDml(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 // Execute a ReadWriteTransaction to update the 'Account' table underpinning
 // 'Account' nodes in 'FinGraph'. The function run by ReadWriteTransaction
 // executes an 'UPDATE' SQL DML statement. Graph queries run after this
 // transaction is committed will observe the effects of the updates to 'Account's
 //
 // The update is performed for all 'Account's whose 'id' is returned by
 // the graph query in the 'IN' subquery, i.e., all 'Account's that have
 // received transfers directly or via one intermediary from an 'Account'
 // whose 'id' is 1.
 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `UPDATE Account SET is_blocked = true 
               WHERE id IN {
                 GRAPH FinGraph 
                 MATCH (a:Account WHERE a.id = 1)-[:TRANSFERS]->{1,2}(b:Account)
                 RETURN b.id}`,
     }
     rowCount, err := txn.Update(ctx, stmt)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%d Account record(s) updated.\n", rowCount)
     return err
 })

 return err
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void updateUsingGraphQueryInDml(DatabaseClient dbClient) {
  dbClient
      .readWriteTransaction()
      .run(
          transaction -> {
            String sql =
                "UPDATE Account SET is_blocked = true "
                    + "WHERE id IN {"
                    + "  GRAPH FinGraph"
                    + "  MATCH (a:Account WHERE a.id = 1)-[:TRANSFERS]->{1,2}(b:Account)"
                    + "  RETURN b.id}";
            long rowCount = transaction.executeUpdate(Statement.of(sql));
            System.out.printf("%d Account record(s) updated.\n", rowCount);
            return null;
          });
}
```

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def update_data_with_graph_query_in_dml(instance_id, database_id):
    """Updates sample data from the database using a DML statement."""

    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    def update_accounts(transaction):
        row_ct = transaction.execute_update(
            "UPDATE Account SET is_blocked = true "
            "WHERE id IN {"
            "  GRAPH FinGraph"
            "  MATCH (a:Account WHERE a.id = 1)-[:TRANSFERS]->{1,2}(b:Account)"
            "  RETURN b.id}"
        )

        print("{} Account record(s) updated.".format(row_ct))

    database.run_in_transaction(update_accounts)
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
