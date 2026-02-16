Insert data into a Spanner Graph using DML.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Manage Spanner Graph data](/spanner/docs/graph/insert-update-delete-data)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void InsertDataWithDml(google::cloud::spanner::Client client) {
  using ::google::cloud::StatusOr;
  namespace spanner = ::google::cloud::spanner;

  std::int64_t rows_inserted;
  auto commit_result = client.Commit(
      [&client, &rows_inserted](
          spanner::Transaction txn) -> StatusOr<spanner::Mutations> {
        auto insert =
            client.ExecuteDml(std::move(txn), spanner::SqlStatement(R"""(
          INSERT INTO Account (id, create_time, is_blocked)
          VALUES
          (1, CAST('2000-08-10 08:18:48.463959-07:52' AS TIMESTAMP), false),
          (2, CAST('2000-08-12 07:13:16.463959-03:41' AS TIMESTAMP), true)
        )"""));
        if (!insert) return std::move(insert).status();
        rows_inserted = insert->RowsModified();
        return spanner::Mutations{};
      });
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << "Rows inserted into Account: " << rows_inserted << "\n";

  commit_result = client.Commit(
      [&client, &rows_inserted](
          spanner::Transaction txn) -> StatusOr<spanner::Mutations> {
        auto insert =
            client.ExecuteDml(std::move(txn), spanner::SqlStatement(R"""(
          INSERT INTO AccountTransferAccount (id, to_id, create_time, amount)
          VALUES
          (1, 2, CAST('2000-09-11 03:11:18.463959-06:36' AS TIMESTAMP), 100),
          (1, 1, CAST('2000-09-12 04:09:34.463959-05:12' AS TIMESTAMP), 200)
        )"""));
        if (!insert) return std::move(insert).status();
        rows_inserted = insert->RowsModified();
        return spanner::Mutations{};
      });
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << "Rows inserted into AccountTransferAccount: " << rows_inserted
            << "\n";

  std::cout << "Insert was successful [spanner_insert_graph_data_with_dml]\n";
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

func insertGraphDataWithDml(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 // Execute a ReadWriteTransaction to insert values into the 'Account' table
 // underpinning 'Account' nodes in 'FinGraph'. The function run by ReadWriteTransaction
 // executes an 'INSERT' SQL DML statement. Graph queries run after this
 // transaction is committed will observe the effects of the new 'Account's
 // added to the graph.
 _, err1 := client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `INSERT INTO Account (id, create_time, is_blocked)
                 VALUES
                     (1, CAST('2000-08-10 08:18:48.463959-07:52' AS TIMESTAMP), false),
                     (2, CAST('2000-08-12 07:13:16.463959-03:41' AS TIMESTAMP), true)`,
     }
     rowCount, err := txn.Update(ctx, stmt)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%d Account record(s) inserted.\n", rowCount)
     return err
 })

 if err1 != nil {
     return err1
 }

 // Execute a ReadWriteTransaction to insert values into the 'AccountTransferAccount'
 // table underpinning 'AccountTransferAccount' edges in 'FinGraph'. The function run
 // by ReadWriteTransaction executes an 'INSERT' SQL DML statement.
 // Graph queries run after this transaction is committed will observe the effects
 // of the edges added to the graph.
 _, err2 := client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `INSERT INTO AccountTransferAccount (id, to_id, create_time, amount)
                 VALUES
                     (1, 2, CAST('2000-09-11 03:11:18.463959-06:36' AS TIMESTAMP), 100),
                     (1, 1, CAST('2000-09-12 04:09:34.463959-05:12' AS TIMESTAMP), 200)`,
     }
     rowCount, err := txn.Update(ctx, stmt)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%d AccountTransferAccount record(s) inserted.\n", rowCount)
     return err
 })

 return err2
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void insertUsingDml(DatabaseClient dbClient) {
  dbClient
      .readWriteTransaction()
      .run(
          transaction -> {
            String sql =
                "INSERT INTO Account (id, create_time, is_blocked) "
                    + "  VALUES"
                    + "    (1, CAST('2000-08-10 08:18:48.463959-07:52' AS TIMESTAMP), false),"
                    + "    (2, CAST('2000-08-12 07:13:16.463959-03:41' AS TIMESTAMP), true)";
            long rowCount = transaction.executeUpdate(Statement.of(sql));
            System.out.printf("%d record(s) inserted into Account.\n", rowCount);
            return null;
          });

  dbClient
      .readWriteTransaction()
      .run(
          transaction -> {
            String sql =
                "INSERT INTO AccountTransferAccount (id, to_id, create_time, amount) "
                    + "  VALUES"
                    + "    (1, 2, CAST('2000-09-11 03:11:18.463959-06:36' AS TIMESTAMP), 100),"
                    + "    (1, 1, CAST('2000-09-12 04:09:34.463959-05:12' AS TIMESTAMP), 200) ";
            long rowCount = transaction.executeUpdate(Statement.of(sql));
            System.out.printf("%d record(s) inserted into AccountTransferAccount.\n", rowCount);
            return null;
          });
}
```

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def insert_data_with_dml(instance_id, database_id):
    """Inserts sample data into the given database using a DML statement."""

    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    def insert_accounts(transaction):
        row_ct = transaction.execute_update(
            "INSERT INTO Account (id, create_time, is_blocked) "
            "  VALUES"
            "    (1, CAST('2000-08-10 08:18:48.463959-07:52' AS TIMESTAMP), false),"
            "    (2, CAST('2000-08-12 07:13:16.463959-03:41' AS TIMESTAMP), true)"
        )

        print("{} record(s) inserted into Account.".format(row_ct))

    def insert_transfers(transaction):
        row_ct = transaction.execute_update(
            "INSERT INTO AccountTransferAccount (id, to_id, create_time, amount) "
            "  VALUES"
            "    (1, 2, CAST('2000-09-11 03:11:18.463959-06:36' AS TIMESTAMP), 100),"
            "    (1, 1, CAST('2000-09-12 04:09:34.463959-05:12' AS TIMESTAMP), 200) "
        )

        print("{} record(s) inserted into AccountTransferAccount.".format(row_ct))

    database.run_in_transaction(insert_accounts)
    database.run_in_transaction(insert_transfers)
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
