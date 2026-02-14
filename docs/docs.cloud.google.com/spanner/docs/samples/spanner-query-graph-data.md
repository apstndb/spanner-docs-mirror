Query data in a Spanner Graph.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Set up and query Spanner Graph](/spanner/docs/graph/set-up)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void QueryData(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  spanner::SqlStatement select(R"""(
    Graph FinGraph
    MATCH (a:Person)-[o:Owns]->()-[t:Transfers]->()<-[p:Owns]-(b:Person)
    RETURN a.name AS sender,
           b.name AS receiver,
           t.amount,
          t.create_time AS transfer_at
  )""");
  using RowType =
      std::tuple<std::string, std::string, double, spanner::Timestamp>;
  auto rows = client.ExecuteQuery(std::move(select));
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "sender: " << std::get<0>(*row) << "\t";
    std::cout << "receiver: " << std::get<1>(*row) << "\t";
    std::cout << "amount: " << std::get<2>(*row) << "\t";
    std::cout << "transfer_at: " << std::get<3>(*row) << "\n";
  }

  std::cout << "Query completed for [spanner_query_graph_data]\n";
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
 "time"

 "cloud.google.com/go/spanner"

 "google.golang.org/api/iterator"
)

func queryGraphData(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 // Execute a Spanner query statement comprising a graph query. Graph queries
 // are characterized by 'MATCH' statements describing node and edge
 // patterns.
 //
 // This statement finds entities ('Account's) owned by all 'Person's 'b' to
 // which transfers have been made by entities ('Account's) owned by any
 // 'Person' 'a' in the graph called 'FinGraph'. It then returns the names of
 // all such 'Person's 'a' and 'b', and the amount and time of the transfer.
 stmt := spanner.Statement{SQL: `Graph FinGraph 
      MATCH (a:Person)-[o:Owns]->()-[t:Transfers]->()<-[p:Owns]-(b:Person)
      RETURN a.name AS sender, b.name AS receiver, t.amount, t.create_time AS transfer_at`}
 iter := client.Single().Query(ctx, stmt)
 defer iter.Stop()

 // The results are returned in tabular form. Iterate over the
 // result rows and print them.
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var sender, receiver string
     var amount float64
     var transfer_at time.Time
     if err := row.Columns(&sender, &receiver, &amount, &transfer_at); err != nil {
         return err
     }
     fmt.Fprintf(w, "%s %s %f %s\n", sender, receiver, amount, transfer_at.Format(time.RFC3339))
 }
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void query(DatabaseClient dbClient) {
  try (ResultSet resultSet =
      dbClient
          .singleUse() // Execute a single query against Cloud Spanner.
          .executeQuery(
              Statement.of(
                  "Graph FinGraph MATCH"
                      + " (a:Person)-[o:Owns]->()-[t:Transfers]->()<-[p:Owns]-(b:Person)RETURN"
                      + " a.name AS sender, b.name AS receiver, t.amount, t.create_time AS"
                      + " transfer_at"))) {
    while (resultSet.next()) {
      System.out.printf(
          "%s %s %f %s\n",
          resultSet.getString(0),
          resultSet.getString(1),
          resultSet.getDouble(2),
          resultSet.getTimestamp(3));
    }
  }
}
```

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def query_data(instance_id, database_id):
    """Queries sample data from the database using GQL."""
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.snapshot() as snapshot:
        results = snapshot.execute_sql(
            """Graph FinGraph
            MATCH (a:Person)-[o:Owns]->()-[t:Transfers]->()<-[p:Owns]-(b:Person)
            RETURN a.name AS sender, b.name AS receiver, t.amount, t.create_time AS transfer_at"""
        )

        for row in results:
            print("sender: {}, receiver: {}, amount: {}, transfer_at: {}".format(*row))
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
