**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This document shows you how to manage data in a graph by inserting, updating, and deleting nodes and edges. Spanner Graph maps data from tables to graph nodes and edges. To mutate data in a graph, you must mutate data in the corresponding input tables. You can use the Google Cloud console, the Google Cloud CLI, or Spanner client libraries to manage graph data.

## Set up your Spanner Graph

Before you can manage data in Spanner Graph, you must set up a graph by doing the following:

1.  [Create a Spanner instance](/spanner/docs/graph/set-up#create-instance) .

2.  [Create a database](/spanner/docs/graph/set-up#create-database) in your Spanner instance.

3.  [Insert graph data](/spanner/docs/graph/set-up#insert-graph-data) into your database.

The examples in these sections use the instance and database that you created when you set up your Spanner Graph with the previous steps.

## Insert nodes or edges

To insert nodes or edges into node or edge tables, use [the Google Cloud console](/spanner/docs/modify-data) , the [gcloud CLI](/sdk/gcloud/reference/spanner) , or the [Spanner client libraries](/spanner/docs/reference/libraries) .

In the Google Cloud console and in the gcloud CLI, you can use GoogleSQL [Data Manipulation Language](/spanner/docs/dml-tasks#use-dml) (DML). In the Spanner client library, you can use DML or [Mutation APIs](/spanner/docs/modify-mutation-api) .

Before you insert an edge, make sure that the source and destination nodes connected by the edge exist. If you insert an edge when the source or destination node connected by the edge doesn't exist, you might get referential integrity violation errors. For more information, see [Missing source node violates INTERLEAVE IN relationship](/spanner/docs/graph/troubleshoot#missing-source-node-violates-foreign-key-constraint) and [Missing destination node violates foreign key constraint](/spanner/docs/graph/troubleshoot#missing-destination-node-violates-foreign-key-constraint) .

The following examples insert `  Account  ` nodes and `  Transfer  ` edges into the database you created in [Set up your Spanner Graph](#before-you-begin) .

### Console

In the Google Cloud console, run the following DML statement. For more information, see [Run statements in the Google Cloud console](/spanner/docs/dml-tasks#console) .

``` text
-- Insert 2 Account nodes.
INSERT INTO Account (id, create_time, is_blocked)
VALUES (1, CAST('2000-08-10 08:18:48.463959-07:52' AS TIMESTAMP), false);
INSERT INTO Account (id, create_time, is_blocked)
VALUES (2, CAST('2000-08-12 07:13:16.463959-03:41' AS TIMESTAMP), true);

-- Insert 2 Transfer edges.
INSERT INTO AccountTransferAccount (id, to_id, create_time, amount)
VALUES (1, 2, CAST('2000-09-11 03:11:18.463959-06:36' AS TIMESTAMP), 100);
INSERT INTO AccountTransferAccount (id, to_id, create_time, amount)
VALUES (1, 1, CAST('2000-09-12 04:09:34.463959-05:12' AS TIMESTAMP), 200);
```

### gcloud

In the gcloud CLI, run the following commands. For more information, see [Execute statements with the gcloud CLI](/spanner/docs/dml-tasks#gcloud-dml) .

``` text
gcloud spanner databases execute-sql  DATABASE-NAME --instance=INSTANCE-NAME \
    --sql="INSERT INTO Account (id, create_time, is_blocked) VALUES (1, CAST('2000-08-10 08:18:48.463959-07:52' AS TIMESTAMP), false)"
gcloud spanner databases execute-sql  DATABASE-NAME --instance=INSTANCE-NAME \
    --sql="INSERT INTO Account (id, create_time, is_blocked) VALUES (2, CAST('2000-08-12 07:13:16.463959-03:41'  AS TIMESTAMP), true)"
gcloud spanner databases execute-sql  DATABASE-NAME --instance=INSTANCE-NAME \
    --sql="INSERT INTO AccountTransferAccount (id, to_id, create_time, amount) VALUES (1, 2, CAST('2000-09-11 03:11:18.463959-06:36' AS TIMESTAMP), 100)"
gcloud spanner databases execute-sql  DATABASE-NAME --instance=INSTANCE-NAME \
    --sql="INSERT INTO AccountTransferAccount (id, to_id, create_time, amount) VALUES (1, 1, CAST('2000-09-12 04:09:34.463959-05:12' AS TIMESTAMP), 200)"
```

Replace the following:

  - DATABASE\_NAME : the name of your database.
  - INSTANCE\_NAME : the name of your instance.

### Client libraries

### Python

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

### Java

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

### Go

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

### C++

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

## Update nodes or edges

To update existing nodes or edges, use the [Google Cloud console](/spanner/docs/modify-data) , the [gcloud CLI](/sdk/gcloud/reference/spanner) , or the [Spanner client libraries](/spanner/docs/reference/libraries) .

You can update existing nodes or edges using a GoogleSQL [Data Manipulation Language](/spanner/docs/dml-tasks#use-dml) (DML) statement, or Spanner Graph queries with a DML statement. If you use a Spanner client library, you can also use [Mutation APIs](/spanner/docs/modify-mutation-api) .

### Update nodes or edges with DML

The following examples use DML to update the `  Account  ` node and `  Transfer  ` edge that you added in [Insert nodes or edges](#insert-nodes-or-edges) .

### Console

In the Google Cloud console, run the following DML statement. For more information, see [Run statements in the Google Cloud console](/spanner/docs/dml-tasks#console) .

``` text
-- Update Account node
UPDATE Account SET is_blocked = false WHERE id = 2;

-- Update Transfer edge
UPDATE AccountTransferAccount
SET amount = 300
WHERE id = 1 AND to_id = 2;
```

### gcloud

1.  [Execute statements with the gcloud CLI](/spanner/docs/dml-tasks#gcloud-dml) .
2.  In the gcloud CLI, run the following commands:

<!-- end list -->

``` text
gcloud spanner databases execute-sql  DATABASE-NAME --instance=INSTANCE-NAME \
    --sql="UPDATE Account SET is_blocked = false WHERE id = 2"
gcloud spanner databases execute-sql  DATABASE-NAME --instance=INSTANCE-NAME \
    --sql="UPDATE AccountTransferAccount SET amount = 300 WHERE id = 1 AND to_id = 2"
```

Replace the following:

  - DATABASE\_NAME : the name of your database.
  - INSTANCE\_NAME : the name of your instance.

### Client libraries

### Python

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

### Java

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

### Go

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

### C++

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

### Update nodes or edges with graph queries and DML

The following examples use Spanner Graph queries with DML to update the `  Account  ` node and `  Transfer  ` edge that you added in [Insert nodes or edges](#insert-nodes-or-edges) .

### Console

In the Google Cloud console, run the following DML statement. For more information, see [Run statements in the Google Cloud console](/spanner/docs/dml-tasks#console) .

``` text
-- Use Graph pattern matching to identify Account nodes to update:
UPDATE Account SET is_blocked = false
WHERE id IN {
  GRAPH FinGraph
  MATCH (a:Account WHERE a.id = 1)-[:Transfers]->{1,2}(b:Account)
  RETURN b.id
}
```

### gcloud

In the gcloud CLI, run the following commands. For more information, see [Execute statements with the gcloud CLI](/spanner/docs/dml-tasks#gcloud-dml) .

``` text
gcloud spanner databases execute-sql DATABASE-NAME --instance=INSTANCE_NAME \
    --sql="UPDATE Account SET is_blocked = false"
gcloud spanner databases execute-sql  DATABASE-NAME --instance=INSTANCE-NAME \
    --sql="UPDATE AccountTransferAccount SET amount = 300 WHERE id = 1 AND to_id = 2"
    --sql=" WHERE id IN { GRAPH FinGraph MATCH (a:Account WHERE a.id = 1)-[:Transfers]->{1,2}(b:Account) RETURN b.id }"
```

Replace the following:

  - DATABASE\_NAME : the name of your database.
  - INSTANCE\_NAME : the name of your instance.

### Client libraries

### Python

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

### Java

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

### Go

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

### C++

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

## Delete nodes or edges

To delete existing nodes or edges, use [the Google Cloud console](/spanner/docs/modify-data) , the [gcloud CLI](/sdk/gcloud/reference/spanner) , or the [Spanner client libraries](/spanner/docs/reference/libraries) .

In the the Google Cloud console and the gcloud CLI, you use GoogleSQL [Data Manipulation Language](/spanner/docs/dml-tasks#use-dml) (DML) to delete. In the Spanner client library, you can use DML or [Mutation APIs](/spanner/docs/modify-mutation-api) to delete nodes or edges.

To prevent referential integrity violation errors, make sure no edges refer to a node when you delete the node. For more information, see [Orphaned outgoing edge violates parent-child relationship](/spanner/docs/graph/troubleshoot#orphaned-outgoing-edge-violates-parent-child-relationship) and [Orphaned incoming edge violates parent-child relationship](/spanner/docs/graph/troubleshoot#orphaned-incoming-edge-violates-parent-child-relationship) .

The following examples delete a `  Transfer  ` edge and an `  Account  ` node from the graph.

### Console

In the Google Cloud console, run the following DML statement. For more information, see [Run statements in the Google Cloud console](/spanner/docs/dml-tasks#console) .

``` text
-- Delete Transfer edge
DELETE FROM AccountTransferAccount
WHERE id = 1 AND to_id = 2;

-- Delete Account node
DELETE FROM Account WHERE id = 2;
```

### gcloud

In the gcloud CLI, run the following commands. For more information, see [Execute statements with the gcloud CLI](/spanner/docs/dml-tasks#gcloud-dml) .

``` text
gcloud spanner databases execute-sql  DATABASE-NAME --instance=INSTANCE-NAME \
    --sql="DELETE FROM AccountTransferAccount WHERE id = 1 AND to_id = 2"
gcloud spanner databases execute-sql  DATABASE-NAME --instance=INSTANCE-NAME \
    --sql="DELETE FROM Account WHERE id = 2"
```

Replace the following:

  - DATABASE\_NAME : the name of your database.
  - INSTANCE\_NAME : the name of your instance.

### Client libraries

### Python

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

### Java

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

### Go

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

### C++

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

You can combine Spanner Graph queries with your DML statement, as shown in the following example:

``` text
  -- Use Graph pattern matching to identify Account nodes to delete:
  DELETE FROM AccountTransferAccount
  WHERE id IN {
    GRAPH FinGraph
    MATCH (a:Account WHERE a.id = 1)-[:Transfers]->(b:Account)
    RETURN b.id
  }
```

## Automated and bulk data operations

In addition to using DML to insert, update, and delete individual nodes and edges, you can also use the following methods to manage your Spanner Graph data:

  - You can automate deletion of edges in a graph by using the [ON DELETE CASCADE](/spanner/docs/graph/best-practices-designing-schema#on-delete-cascade) action.

  - You can automate deletion of nodes and edges in the graph using a [TTL](/spanner/docs/ttl) policy. For more information, see [TTL on nodes and edges](/spanner/docs/graph/best-practices-designing-schema#ttl-on-nodes-and-edges) .

  - Efficiently bulk update and delete nodes and edges in the graph using [partitioned DML](/spanner/docs/dml-partitioned) .

## What's next

  - Read the [Spanner Graph queries overview](/spanner/docs/graph/queries-overview) .
  - Learn [best practices for tuning Spanner Graph queries](/spanner/docs/graph/best-practices-tuning-queries) .
