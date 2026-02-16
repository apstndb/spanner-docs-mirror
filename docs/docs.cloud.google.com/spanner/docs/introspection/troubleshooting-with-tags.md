Spanner provides a set of built-in statistics tables to help you gain insight into your queries, reads, and transactions. To correlate statistics with your application code and to improve troubleshooting, you can add a tag (a free-form string) to Spanner read, query, and transaction operations in your application code. These tags are populated in statistics tables helping you to correlate and search based on tags.

Spanner supports two types of tags; ***request*** tags and ***transaction*** tags. As their names suggest, you can add transaction tags to transactions, and request tags to individual queries and reads APIs. You can set a transaction tag at the transaction scope and set individual request tags for each applicable API request within the transaction. Request tags and transaction tags that are set in the application code are populated in the columns of following statistics tables.

<table>
<thead>
<tr class="header">
<th>Statistics Table</th>
<th>Type of Tags populated in the statistics table</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/introspection/query-statistics#table_schema">TopN Query Statistics</a></td>
<td>Request tags</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/introspection/read-statistics#table_schema">TopN Read Statistics</a></td>
<td>Request tags</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/introspection/transaction-statistics#table_schema">TopN Transaction Statistics</a></td>
<td>Transaction tags</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/introspection/lock-statistics#table_schema">TopN Lock Statistics</a></td>
<td>Transaction tags</td>
</tr>
</tbody>
</table>

**Note:** Tags with the prefix `  sys_cloud_console_  ` are created by the Google Cloud console.

## Request tags

You can add an optional request tag to a query or a read request. Spanner groups statistics by request tag, which is visible in the `  REQUEST_TAG  ` field of both the [query statistics](/spanner/docs/introspection/query-statistics) and [read statistics](/spanner/docs/introspection/read-statistics) tables.

### When to use request tags

The following are some of the scenarios that benefit from using request tags.

  - **Finding the source of a problematic query or read:** Spanner collects statistics for reads and queries in built-in statistics tables. When you find the slow queries or high cpu consuming reads in the statistics table, if you have already assigned tags to those, then you can identify the source (application/microservice) that is calling these operations based on the information in the tag.
  - **Identifying reads or queries in statistics tables:** Assigning request tags helps to filter rows in the statistics table based on the tags that you are interested in.
  - **Finding if queries from a particular application or microservice are slow** : Request tags can help identify if queries from a particular application or microservice have higher latencies.
  - **Grouping statistics for a set of reads or queries:** You can use request tags to track, compare, and report performance across a set of similar reads or queries. For example, if multiple queries are accessing a table or a set of tables with the same access pattern, you can consider adding the same tag to all those queries to track them together.

### How to assign request tags

The following sample shows how to set request tags using the Spanner client libraries.

### C++

``` cpp
void SetRequestTag(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  spanner::SqlStatement select(
      "SELECT SingerId, AlbumId, AlbumTitle FROM Albums");
  using RowType = std::tuple<std::int64_t, std::int64_t, std::string>;

  auto opts = google::cloud::Options{}.set<spanner::RequestTagOption>(
      "app=concert,env=dev,action=select");
  auto rows = client.ExecuteQuery(std::move(select), std::move(opts));
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row)
              << " AlbumId: " << std::get<1>(*row)
              << " AlbumTitle: " << std::get<2>(*row) << "\n";
  }
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

public class RequestTagAsyncSample
{
    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
    }

    public async Task<List<Album>> RequestTagAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand(
            $"SELECT SingerId, AlbumId, AlbumTitle FROM Albums");
        // Sets the request tag to "app=concert,env=dev,action=select".
        // This request tag will only be set on this request.
        cmd.Tag = "app=concert,env=dev,action=select";

        var albums = new List<Album>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            var album = new Album
            {
                SingerId = reader.GetFieldValue<int>("SingerId"),
                AlbumId = reader.GetFieldValue<int>("AlbumId"),
                AlbumTitle = reader.GetFieldValue<string>("AlbumTitle")
            };
            albums.Add(album);
            Console.WriteLine($"SingerId: {album.SingerId}, AlbumId: {album.AlbumId}, AlbumTitle: {album.AlbumTitle}");
        }
        return albums;
    }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

// queryWithTag reads from a database with request tag set
func queryWithTag(w io.Writer, db string) error {
 // db = `projects/<project>/instances/<instance-id>/database/<database-id>`
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{SQL: `SELECT SingerId, AlbumId, AlbumTitle FROM Albums`}
 iter := client.Single().QueryWithOptions(ctx, stmt, spanner.QueryOptions{RequestTag: "app=concert,env=dev,action=select"})
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var singerID, albumID int64
     var albumTitle string
     if err := row.Columns(&singerID, &albumID, &albumTitle); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, albumTitle)
 }
}
```

### Java

``` java
static void setRequestTag(DatabaseClient databaseClient) {
  // Sets the request tag to "app=concert,env=dev,action=select".
  // This request tag will only be set on this request.
  try (ResultSet resultSet = databaseClient
      .singleUse()
      .executeQuery(
          Statement.of("SELECT SingerId, AlbumId, AlbumTitle FROM Albums"),
          Options.tag("app=concert,env=dev,action=select"))) {
    while (resultSet.next()) {
      System.out.printf(
          "SingerId: %d, AlbumId: %d, AlbumTitle: %s\n",
          resultSet.getLong(0),
          resultSet.getLong(1),
          resultSet.getString(2));
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

async function queryTags() {
  // Gets a reference to a Cloud Spanner instance and database.
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);

  // Execute a query with a request tag.
  const [albums] = await database.run({
    sql: 'SELECT SingerId, AlbumId, AlbumTitle FROM Albums',
    requestOptions: {requestTag: 'app=concert,env=dev,action=select'},
    json: true,
  });
  albums.forEach(album => {
    console.log(
      `SingerId: ${album.SingerId}, AlbumId: ${album.AlbumId}, AlbumTitle: ${album.AlbumTitle}`,
    );
  });
  await database.close();
}
queryTags();
```

### PHP

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Executes a read with a request tag.
 * Example:
 * ```
 * spanner_set_request_tag($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function set_request_tag(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $snapshot = $database->snapshot();
    $results = $snapshot->execute(
        'SELECT SingerId, AlbumId, AlbumTitle FROM Albums',
        [
            'requestOptions' => [
                'requestTag' => 'app=concert,env=dev,action=select'
            ]
        ]
    );
    foreach ($results as $row) {
        printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
            $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
    }
}
````

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT SingerId, AlbumId, AlbumTitle FROM Albums",
        request_options={"request_tag": "app=concert,env=dev,action=select"},
    )

    for row in results:
        print("SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row))
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client = spanner.client instance_id, database_id

client.execute(
  "SELECT SingerId, AlbumId, MarketingBudget FROM Albums",
  request_options: { tag: "app=concert,env=dev,action=select" }
).rows.each do |row|
  puts "#{row[:SingerId]} #{row[:AlbumId]} #{row[:MarketingBudget]}"
end
```

### How to view request tags in statistics table

The following query returns the query statistics over 10 minute intervals.

``` text
SELECT t.text,
       t.request_tag,
       t.execution_count,
       t.avg_latency_seconds,
       t.avg_rows,
       t.avg_bytes
FROM SPANNER_SYS.QUERY_STATS_TOP_10MINUTE AS t
LIMIT 3;
```

Let's take the following data as an example of the results we get back from our query.

<table>
<thead>
<tr class="header">
<th><strong>text</strong></th>
<th><strong>request_tag</strong></th>
<th><strong>execution_count</strong></th>
<th><strong>avg_latency_seconds</strong></th>
<th><strong>avg_rows</strong></th>
<th><strong>avg_bytes</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>SELECT SingerId, AlbumId, AlbumTitle FROM Albums</td>
<td>app=concert,env=dev,action=select</td>
<td>212</td>
<td>0.025</td>
<td>21</td>
<td>2365</td>
</tr>
<tr class="even">
<td>select * from orders;</td>
<td>app=catalogsearch,env=dev,action=list</td>
<td>55</td>
<td>0.02</td>
<td>16</td>
<td>33.35</td>
</tr>
<tr class="odd">
<td>SELECT SingerId, FirstName, LastName FROM Singers;</td>
<td>[empty string]</td>
<td>154</td>
<td>0.048</td>
<td>42</td>
<td>486.33</td>
</tr>
</tbody>
</table>

From this table of results, we can see that if you have assigned a `  REQUEST_TAG  ` for a query, then it gets populated in the statistics table. If there is no request tag assigned, it is displayed as an empty string.

For the tagged queries, the statistics are aggregated per tag (e.g. request tag `  app=concert,env=dev,action=select  ` has an average latency of **0.025** seconds). If there is no tag assigned then the statistics are aggregated per query (e.g. the query in the third row has an average latency of **0.048** seconds).

## Transaction tags

An optional transaction tag can be added to individual transactions. Spanner groups statistics by transaction tag, which is visible in the `  TRANSACTION_TAG  ` field of [transaction statistics](/spanner/docs/introspection/transaction-statistics) tables.

### When to use transaction tags

The following are some of the scenarios that benefit from using transaction tags.

  - **Finding the source of a problematic transaction:** Spanner collects statistics for read-write transactions in the transaction statistics table. When you find slow transactions in the transaction statistics table, if you have already assigned tags to them, then you can identify the source (application/microservice) that is calling these transactions based on the information in the tag.
  - **Identifying transactions in statistics tables:** Assigning transaction tags helps to filter rows in the transaction statistics table based on the tags that you are interested in. Without transaction tags, discovering what operations are represented by a statistic can be a cumbersome process. For example, for transaction statistics, you would have to examine the tables and columns involved in order to identify the untagged transaction.
  - **Finding if transactions from a particular application or microservice are slow** : Transaction tags can help identify if transactions from a particular application or microservice have higher latencies.
  - **Grouping statistics for a set of transactions:** You can use transaction tags to track, compare, and report performance for a set of similar transactions.
  - **Finding which transactions are accessing the columns involved in the lock conflict:** Transaction tags can help pinpoint individual transactions causing lock conflicts in the [Lock statistics](/spanner/docs/introspection/lock-statistics) tables.
  - **Streaming user change data out of Spanner using [change streams](/spanner/docs/change-streams/details) :** Change streams data records contain transaction tags for the transactions that modified the user data. This allows the reader of a change stream to associate changes with the transaction type based on tags.

### How to assign transaction tags

The following sample shows how to set transaction tags using the Spanner client libraries. When you use a client library you can set a transaction tag at the beginning of the transaction call which gets applied to all the individual operations inside that transaction.

### C++

``` cpp
void SetTransactionTag(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  using ::google::cloud::StatusOr;

  // Sets the transaction tag to "app=concert,env=dev". This will be
  // applied to all the individual operations inside this transaction.
  auto commit_options =
      google::cloud::Options{}.set<spanner::TransactionTagOption>(
          "app=concert,env=dev");
  auto commit = client.Commit(
      [&client](
          spanner::Transaction const& txn) -> StatusOr<spanner::Mutations> {
        spanner::SqlStatement update_statement(
            "UPDATE Venues SET Capacity = CAST(Capacity/4 AS INT64)"
            "  WHERE OutdoorVenue = false");
        // Sets the request tag to "app=concert,env=dev,action=update".
        // This will only be set on this request.
        auto update = client.ExecuteDml(
            txn, std::move(update_statement),
            google::cloud::Options{}.set<spanner::RequestTagOption>(
                "app=concert,env=dev,action=update"));
        if (!update) return std::move(update).status();

        spanner::SqlStatement insert_statement(
            "INSERT INTO Venues (VenueId, VenueName, Capacity, OutdoorVenue, "
            "                    LastUpdateTime)"
            " VALUES (@venueId, @venueName, @capacity, @outdoorVenue, "
            "         PENDING_COMMIT_TIMESTAMP())",
            {
                {"venueId", spanner::Value(81)},
                {"venueName", spanner::Value("Venue 81")},
                {"capacity", spanner::Value(1440)},
                {"outdoorVenue", spanner::Value(true)},
            });
        // Sets the request tag to "app=concert,env=dev,action=insert".
        // This will only be set on this request.
        auto insert = client.ExecuteDml(
            txn, std::move(insert_statement),
            google::cloud::Options{}.set<spanner::RequestTagOption>(
                "app=concert,env=dev,action=select"));
        if (!insert) return std::move(insert).status();
        return spanner::Mutations{};
      },
      commit_options);
  if (!commit) throw std::move(commit).status();
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System.Threading.Tasks;

public class TransactionTagAsyncSample
{
    public async Task<int> TransactionTagAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        return await connection.RunWithRetriableTransactionAsync(async transaction =>
        {
            // Sets the transaction tag to "app=concert,env=dev".
            // This transaction tag will be applied to all the individual operations inside
            // the transaction.
            transaction.TransactionOptions.Tag = "app=concert,env=dev";

            // Sets the request tag to "app=concert,env=dev,action=update".
            // This request tag will only be set on this request.
            var updateCommand =
                connection.CreateDmlCommand("UPDATE Venues SET Capacity = DIV(Capacity, 4) WHERE OutdoorVenue = false");
            updateCommand.Tag = "app=concert,env=dev,action=update";
            updateCommand.Transaction = transaction;
            int rowsModified = await updateCommand.ExecuteNonQueryAsync();

            var insertCommand = connection.CreateDmlCommand(
                @"INSERT INTO Venues (VenueId, VenueName, Capacity, OutdoorVenue, LastUpdateTime)
                    VALUES (@venueId, @venueName, @capacity, @outdoorVenue, PENDING_COMMIT_TIMESTAMP())",
                new SpannerParameterCollection
                {
                    {"venueId", SpannerDbType.Int64, 81},
                    {"venueName", SpannerDbType.String, "Venue 81"},
                    {"capacity", SpannerDbType.Int64, 1440},
                    {"outdoorVenue", SpannerDbType.Bool, true}
                }
            );
            // Sets the request tag to "app=concert,env=dev,action=insert".
            // This request tag will only be set on this request.
            insertCommand.Tag = "app=concert,env=dev,action=insert";
            insertCommand.Transaction = transaction;
            rowsModified += await insertCommand.ExecuteNonQueryAsync();
            return rowsModified;
        });
    }
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

// readWriteTransactionWithTag executes the update and insert queries on venues table with appropriate transaction and requests tag
func readWriteTransactionWithTag(w io.Writer, db string) error {
 // db = `projects/<project>/instances/<instance-id>/database/<database-id>`
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 _, err = client.ReadWriteTransactionWithOptions(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `UPDATE Venues SET Capacity = CAST(Capacity/4 AS INT64) WHERE OutdoorVenue = false`,
     }
     _, err := txn.UpdateWithOptions(ctx, stmt, spanner.QueryOptions{RequestTag: "app=concert,env=dev,action=update"})
     if err != nil {
         return err
     }
     fmt.Fprint(w, "Venue capacities updated.")
     stmt = spanner.Statement{
         SQL: `INSERT INTO Venues (VenueId, VenueName, Capacity, OutdoorVenue, LastUpdateTime) 
                   VALUES (@venueId, @venueName, @capacity, @outdoorVenue, PENDING_COMMIT_TIMESTAMP())`,
         Params: map[string]interface{}{
             "venueId":      81,
             "venueName":    "Venue 81",
             "capacity":     1440,
             "outdoorVenue": true,
         },
     }
     _, err = txn.UpdateWithOptions(ctx, stmt, spanner.QueryOptions{RequestTag: "app=concert,env=dev,action=insert"})
     if err != nil {
         return err
     }
     fmt.Fprint(w, "New venue inserted.")
     return nil
 }, spanner.TransactionOptions{TransactionTag: "app=concert,env=dev"})
 return err
}
```

### Java

``` java
static void setTransactionTag(DatabaseClient databaseClient) {
  // Sets the transaction tag to "app=concert,env=dev".
  // This transaction tag will be applied to all the individual operations inside this
  // transaction.
  databaseClient
      .readWriteTransaction(Options.tag("app=concert,env=dev"))
      .run(transaction -> {
        // Sets the request tag to "app=concert,env=dev,action=update".
        // This request tag will only be set on this request.
        transaction.executeUpdate(
            Statement.of("UPDATE Venues"
                + " SET Capacity = CAST(Capacity/4 AS INT64)"
                + " WHERE OutdoorVenue = false"),
            Options.tag("app=concert,env=dev,action=update"));
        System.out.println("Venue capacities updated.");

        Statement insertStatement = Statement.newBuilder(
            "INSERT INTO Venues"
                + " (VenueId, VenueName, Capacity, OutdoorVenue, LastUpdateTime)"
                + " VALUES ("
                + " @venueId, @venueName, @capacity, @outdoorVenue, PENDING_COMMIT_TIMESTAMP()"
                + " )")
            .bind("venueId")
            .to(81)
            .bind("venueName")
            .to("Venue 81")
            .bind("capacity")
            .to(1440)
            .bind("outdoorVenue")
            .to(true)
            .build();

        // Sets the request tag to "app=concert,env=dev,action=insert".
        // This request tag will only be set on this request.
        transaction.executeUpdate(
            insertStatement,
            Options.tag("app=concert,env=dev,action=insert"));
        System.out.println("New venue inserted.");

        return null;
      });
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

async function transactionTag() {
  // Gets a reference to a Cloud Spanner instance and database.
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);

  // Run a transaction with a transaction tag that will automatically be
  // included with each request in the transaction.
  try {
    await database.runTransactionAsync(
      {requestOptions: {transactionTag: 'app=cart,env=dev'}},
      async tx => {
        // Set the request tag to "app=concert,env=dev,action=update".
        // This request tag will only be set on this request.
        await tx.runUpdate({
          sql: 'UPDATE Venues SET Capacity = DIV(Capacity, 4) WHERE OutdoorVenue = false',
          requestOptions: {requestTag: 'app=concert,env=dev,action=update'},
        });
        console.log('Updated capacity of all indoor venues to 1/4.');

        await tx.runUpdate({
          sql: `INSERT INTO Venues (VenueId, VenueName, Capacity, OutdoorVenue, LastUpdateTime)
                VALUES (@venueId, @venueName, @capacity, @outdoorVenue, PENDING_COMMIT_TIMESTAMP())`,
          params: {
            venueId: 81,
            venueName: 'Venue 81',
            capacity: 1440,
            outdoorVenue: true,
          },
          types: {
            venueId: {type: 'int64'},
            venueName: {type: 'string'},
            capacity: {type: 'int64'},
            outdoorVenue: {type: 'bool'},
          },
          requestOptions: {requestTag: 'app=concert,env=dev,action=update'},
        });
        console.log('Inserted new outdoor venue');

        await tx.commit();
      },
    );
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    await database.close();
  }
}
transactionTag();
```

### PHP

```` php
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Transaction;

/**
 * Executes a transaction with a transaction tag.
 * Example:
 * ```
 * spanner_set_transaction_tag($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function set_transaction_tag(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $database->runTransaction(function (Transaction $t) {
        $t->executeUpdate(
            'UPDATE Venues SET Capacity = CAST(Capacity/4 AS INT64) WHERE OutdoorVenue = false',
            [
                'requestOptions' => ['requestTag' => 'app=concert,env=dev,action=update']
            ]
        );
        print('Venue capacities updated.' . PHP_EOL);
        $t->executeUpdate(
            'INSERT INTO Venues (VenueId, VenueName, Capacity, OutdoorVenue, LastUpdateTime) '
            . 'VALUES (@venueId, @venueName, @capacity, @outdoorVenue, PENDING_COMMIT_TIMESTAMP())',
            [
                'parameters' => [
                    'venueId' => 81,
                    'venueName' => 'Venue 81',
                    'capacity' => 1440,
                    'outdoorVenue' => true,
                ],
                'requestOptions' => ['requestTag' => 'app=concert,env=dev,action=insert']
            ]
        );
        print('New venue inserted.' . PHP_EOL);
        $t->commit();
    }, [
        'requestOptions' => ['transactionTag' => 'app=concert,env=dev']
    ]);
}
````

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

def update_venues(transaction):
    # Sets the request tag to "app=concert,env=dev,action=update".
    #  This request tag will only be set on this request.
    transaction.execute_update(
        "UPDATE Venues SET Capacity = CAST(Capacity/4 AS INT64) WHERE OutdoorVenue = false",
        request_options={"request_tag": "app=concert,env=dev,action=update"},
    )
    print("Venue capacities updated.")

    # Sets the request tag to "app=concert,env=dev,action=insert".
    # This request tag will only be set on this request.
    transaction.execute_update(
        "INSERT INTO Venues (VenueId, VenueName, Capacity, OutdoorVenue, LastUpdateTime) "
        "VALUES (@venueId, @venueName, @capacity, @outdoorVenue, PENDING_COMMIT_TIMESTAMP())",
        params={
            "venueId": 81,
            "venueName": "Venue 81",
            "capacity": 1440,
            "outdoorVenue": True,
        },
        param_types={
            "venueId": param_types.INT64,
            "venueName": param_types.STRING,
            "capacity": param_types.INT64,
            "outdoorVenue": param_types.BOOL,
        },
        request_options={"request_tag": "app=concert,env=dev,action=insert"},
    )
    print("New venue inserted.")

database.run_in_transaction(update_venues, transaction_tag="app=concert,env=dev")
```

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client = spanner.client instance_id, database_id

client.transaction request_options: { tag: "app=cart,env=dev" } do |tx|
  tx.execute_update \
    "UPDATE Venues SET Capacity = CAST(Capacity/4 AS INT64) WHERE OutdoorVenue = false",
    request_options: { tag: "app=concert,env=dev,action=update" }

  puts "Venue capacities updated."

  tx.execute_update \
    "INSERT INTO Venues (VenueId, VenueName, Capacity, OutdoorVenue) " \
    "VALUES (@venue_id, @venue_name, @capacity, @outdoor_venue)",
    params: {
      venue_id: 81,
      venue_name: "Venue 81",
      capacity: 1440,
      outdoor_venue: true
    },
    request_options: { tag: "app=concert,env=dev,action=insert" }

  puts "New venue inserted."
end
```

### How to view transaction tags in Transaction Statistics table

The following query returns the transaction statistics over 10 minute intervals.

``` text
SELECT t.fprint,
       t.transaction_tag,
       t.read_columns,
       t.commit_attempt_count,
       t.avg_total_latency_seconds
FROM SPANNER_SYS.TXN_STATS_TOP_10MINUTE AS t
LIMIT 3;
```

Let's take the following data as an example of the results we get back from our query.

<table>
<colgroup>
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>fprint</strong></th>
<th><strong>transaction_tag</strong></th>
<th><strong>read_columns</strong></th>
<th><strong>commit_attempt_count</strong></th>
<th><strong>avg_total_latency_seconds</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>40015598317</td>
<td>app=concert,env=dev</td>
<td>[Venues._exists,<br />
Venues.VenueId,<br />
Venues.VenueName,<br />
Venues.Capacity]</td>
<td>278802</td>
<td>0.3508</td>
</tr>
<tr class="even">
<td>20524969030</td>
<td>app=product,service=payment</td>
<td>[Singers.SingerInfo]</td>
<td>129012</td>
<td>0.0142</td>
</tr>
<tr class="odd">
<td>77848338483</td>
<td>[empty string]</td>
<td>[Singers.FirstName, Singers.LastName, Singers._exists]</td>
<td>5357</td>
<td>0.048</td>
</tr>
</tbody>
</table>

From this table of results, we can see that if you have assigned a `  TRANSACTION_TAG  ` to a transaction, then it gets populated in the transaction statistics table. If there is no transaction tag assigned, it is displayed as an empty string.

For the tagged transactions, the statistics are aggregated per transaction tag (e.g. transaction tag `  app=concert,env=dev  ` a has an average latency of **0.3508** seconds). If there is no tag assigned then the statistics are aggregated per `  FPRINT  ` (e.g. **77848338483** in the third row has an average latency of **0.048** seconds).

### How to view transaction tags in Lock Statistics table

The following query returns the lock statistics over 10 minute intervals.

The [`  CAST()  `](/spanner/docs/reference/standard-sql/conversion_rules#casting) function converts the `  row_range_start_key  ` BYTES field to a STRING.

``` text
SELECT
   CAST(s.row_range_start_key AS STRING) AS row_range_start_key,
   s.lock_wait_seconds,
   s.sample_lock_requests
FROM SPANNER_SYS.LOCK_STATS_TOP_10MINUTE s
LIMIT 2;
```

Let's take the following data as an example of the results we get back from our query.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>row_range_start_key</strong></th>
<th><strong>lock_wait_seconds</strong></th>
<th><strong>sample_lock_requests</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Songs(2,1,1)</td>
<td>0.61</td>
<td>LOCK_MODE: ReaderShared<br />
COLUMN: Singers.SingerInfo<br />
TRANSACTION_TAG: app=product,service=shipping<br />
<br />
LOCK_MODE: WriterShared<br />
COLUMN: Singers.SingerInfo<br />
TRANSACTION_TAG: app=product,service=payment</td>
</tr>
<tr class="even">
<td>albums(2,1+)</td>
<td>0.48</td>
<td>LOCK_MODE: ReaderShared<br />
COLUMN: users._exists1<br />
TRANSACTION_TAG: [empty string]<br />
<br />
LOCK_MODE: WriterShared<br />
COLUMN: users._exists<br />
TRANSACTION_TAG: [empty string]</td>
</tr>
</tbody>
</table>

From this table of results, we can see that if you have assigned a `  TRANSACTION_TAG  ` to a transaction, then it gets populated in the lock statistics table. If there is no transaction tag assigned, it is displayed as an empty string.

## Mapping between API methods and request/transaction tag

Request tags and transaction tags are applicable to specific API methods based on whether the transaction mode is a read-only transaction or a read-write transaction. Generally, transaction tags are applicable to read-write transactions whereas request tags are applicable to read-only transactions. The following table shows the mapping from API methods to applicable types of tags.

**API Methods**

**Transaction Modes**

**Request Tag**

**Transaction Tag**

Read,  
StreamingRead

Read-only transaction

Yes

No

Read-write transaction

Yes

Yes

ExecuteSql,  
ExecuteStreamingSql <sup>1</sup>

Read-only transaction <sup>1</sup>

Yes <sup>1</sup>

No

Read-write transaction

Yes

Yes

ExecuteBatchDml

Read-write transaction

Yes

Yes

BeginTransaction

Read-write transaction

No

Yes

Commit

Read-write transaction

No

Yes

<sup>1</sup> For change stream queries executed using the Apache Beam SpannerIO Dataflow connector, the `  REQUEST_TAG  ` contains a Dataflow job name.

**Note:** When you use a client library you can set a `  TRANSACTION_TAG  ` at the beginning of the read-write transaction call which gets applied to all the individual operations inside that transaction.

## Limitations

When adding tags to your reads, queries, and transactions, consider the following limitations:

  - The length of a tag string is limited to 50 characters. Strings that exceed this limit are truncated.

  - Only ASCII characters (32-126) are allowed in a tag. Arbitrary unicode characters are replaced by underscores.

  - Any leading underscore (\_) characters are removed from the string.

  - Tags are case-sensitive. For example, if you add the request tag `  APP=cart,ENV=dev  ` to one set of queries, and add `  app=cart,env=dev  ` to another set of queries, Spanner aggregates statistics separately for each tag.

  - Tags may be missing from the statistics tables under the following circumstance:
    
      - If Spanner is unable to store statistics for all tagged operations run during the interval in tables, the system prioritizes operations with the highest consuming resources during the specified interval.

## Tag naming

When assigning tags to your database operations, it is important to consider what information you want to convey in each tag string. The convention or pattern you choose makes your tags more effective. For example, proper tag naming makes it easier to correlate statistics with application code.

You can choose any tag you wish within the stated [limitations](/spanner/docs/introspection/troubleshooting-with-tags#limitations) . However, we recommend constructing a tag string as a set of **key-value** pairs separated by commas.

For example, assume that you are using a Spanner database for an e-commerce use case. You might want to include information about the application, development environment, and the action being taken by the query in the request tag that you are going to assign to a particular query. You can consider assigning the tag string in the key-value format as `  app=cart,env=dev,action=update  ` .This means the query is called from the cart application in the development environment, and is used to update the cart.

Suppose you have another query from a catalog search application and you assign the tag string as `  app=catalogsearch,env=dev,action=list  ` . Now if any of these queries show up in the query statistics table as high latency queries, you can easily identify the source by using the tag.

Here are some examples of how a tagging pattern can be used to organize your operation statistics. These examples are not meant to be exhaustive; you can also combine them in your tag string using a delimiter such as a comma.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Tag keys</strong></th>
<th><strong>Examples of Tag-value pair</strong></th>
<th><strong>Description</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Application</td>
<td>app=cart<br />
app=frontend<br />
app=catalogsearch</td>
<td>Helps in identifying the application that is calling the operation.</td>
</tr>
<tr class="even">
<td>Environment</td>
<td>env=prod<br />
env=dev<br />
env=test<br />
env=staging</td>
<td>Helps in identifying the environment that is associated with the operation.</td>
</tr>
<tr class="odd">
<td>Framework</td>
<td>framework=spring<br />
framework=django<br />
framework=jetty</td>
<td>Helps in identifying the framework that is associated with the operation.</td>
</tr>
<tr class="even">
<td>Action</td>
<td>action=list<br />
action=retrieve<br />
action=update</td>
<td>Helps in identifying the action taken by the operation.</td>
</tr>
<tr class="odd">
<td>Service</td>
<td>service=payment<br />
service=shipping</td>
<td>Helps in identifying the microservice that is calling the operation.</td>
</tr>
</tbody>
</table>

## Things to Note

  - When you assign a `  REQUEST_TAG  ` , statistics for multiple queries that have the same tag string are grouped in a single row in [query statistics](/spanner/docs/introspection/query-statistics) table. Only the text of one of those queries is shown in the `  TEXT  ` field.
  - When you assign a `  REQUEST_TAG  ` , statistics for multiple reads that have the same tag string are grouped in a single row in [read statistics](/spanner/docs/introspection/read-statistics) table. The set of all columns that are read are added to the `  READ_COLUMNS  ` field.
  - When you assign a `  TRANSACTION_TAG  ` , statistics for transactions that have the same tag string are grouped in a single row in [transaction statistics](/spanner/docs/introspection/transaction-statistics) table. The set of all columns that are written by the transactions are added to the `  WRITE_CONSTRUCTIVE_COLUMNS  ` field and the set of all columns that are read are added to the `  READ_COLUMNS  ` field.

**Note:** Avoid using personally identifiable information (PII) or other user data in a tag. Tags are not designed to handle sensitive information.

## Troubleshooting scenarios using tags

### Finding the source of a problematic transaction

The following query returns the raw data for the top transactions in the selected time period.

``` text
SELECT
 fprint,
 transaction_tag,
 ROUND(avg_total_latency_seconds,4) as avg_total_latency_sec,
 ROUND(avg_commit_latency_seconds,4) as avg_commit_latency_sec,
 commit_attempt_count,
 commit_abort_count
FROM SPANNER_SYS.TXN_STATS_TOP_10MINUTE
WHERE interval_end = "2020-05-17T18:40:00"
ORDER BY avg_total_latency_seconds DESC;
```

The following table lists example data returned from our query, where we have three applications, namely **cart** , **product** and **frontend** , that own or query the same database.

Once you identify the transactions experiencing high latency, you can use the associated tags to identify the relevant part of your application code, and troubleshoot further using [transaction statistics](/spanner/docs/introspection/transaction-statistics) .

<table>
<thead>
<tr class="header">
<th><strong>fprint</strong></th>
<th><strong>transaction_tag</strong></th>
<th><strong>avg_total_latency_sec</strong></th>
<th><strong>avg_commit_latency_sec</strong></th>
<th><strong>commit_attempt_count</strong></th>
<th><strong>commit_abort_count</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7129109266372596045</td>
<td>app=cart,service=order</td>
<td>0.3508</td>
<td>0.0139</td>
<td>278802</td>
<td>142205</td>
</tr>
<tr class="even">
<td>9353100217060788102</td>
<td>app=cart,service=redis</td>
<td>0.1633</td>
<td>0.0142</td>
<td>129012</td>
<td>27177</td>
</tr>
<tr class="odd">
<td>9353100217060788102</td>
<td>app=product,service=payment</td>
<td>0.1423</td>
<td>0.0133</td>
<td>5357</td>
<td>636</td>
</tr>
<tr class="even">
<td>898069986622520747</td>
<td>app=product,service=shipping</td>
<td>0.0159</td>
<td>0.0118</td>
<td>4269</td>
<td>1</td>
</tr>
<tr class="odd">
<td>9521689070912159706</td>
<td>app=frontend,service=ads</td>
<td>0.0093</td>
<td>0.0045</td>
<td>164</td>
<td>0</td>
</tr>
<tr class="even">
<td>11079878968512225881</td>
<td>[empty string]</td>
<td>0.031</td>
<td>0.015</td>
<td>14</td>
<td>0</td>
</tr>
</tbody>
</table>

Similarly, Request Tag can be used to find the source of a problematic query from [query statistics](/spanner/docs/introspection/query-statistics) table and source of problematic read from [read statistics](/spanner/docs/introspection/read-statistics) table.

### Finding the latency and other stats for transactions from a particular application or microservice

If you have used the application name or microservice name in the tag string, it helps in filtering the transaction statistics table by tags that contain that application name or microservice name.

Suppose you have added new transactions to the **payment** app and you want to look at latencies and other statistics of those new transactions. If you have used the name of the payment application within the tag, you can filter the transaction statistics table for only those tags that contain `  app=payment  ` .

The following query returns the transaction statistics for payment app over 10 minute intervals.

``` text
SELECT
  transaction_tag,
  avg_total_latency_sec,
  avg_commit_latency_sec,
  commit_attempt_count,
  commit_abort_count
FROM SPANNER_SYS.TXN_STATS_TOP_10MINUTE
WHERE STARTS_WITH(transaction_tag, "app=payment")
LIMIT 3;
```

Here's some example output:

<table>
<thead>
<tr class="header">
<th><strong>transaction_tag</strong></th>
<th><strong>avg_total_latency_sec</strong></th>
<th><strong>avg_commit_latency_sec</strong></th>
<th><strong>commit_attempt_count</strong></th>
<th><strong>commit_abort_count</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>app=payment,action=update</td>
<td>0.3508</td>
<td>0.0139</td>
<td>278802</td>
<td>142205</td>
</tr>
<tr class="even">
<td>app=payment,action=transfer</td>
<td>0.1633</td>
<td>0.0142</td>
<td>129012</td>
<td>27177</td>
</tr>
<tr class="odd">
<td>app=payment, action=retrieve</td>
<td>0.1423</td>
<td>0.0133</td>
<td>5357</td>
<td>636</td>
</tr>
</tbody>
</table>

Similarly, you can find queries or reads from a specific application in [query statistics](/spanner/docs/introspection/query-statistics) or [read statistics](/spanner/docs/introspection/read-statistics) table using request tags.

### Discovering the transactions involved in lock conflict

To find out which transactions and row keys experienced the high lock wait times, we query the `  LOCK_STAT_TOP_10MINUTE  ` table, which lists the row keys, columns, and corresponding transactions that are involved in the lock conflict.

``` text
SELECT CAST(s.row_range_start_key AS STRING) AS row_range_start_key,
       t.total_lock_wait_seconds,
       s.lock_wait_seconds,
       s.lock_wait_seconds/t.total_lock_wait_seconds frac_of_total,
       s.sample_lock_requests
FROM spanner_sys.lock_stats_total_10minute t, spanner_sys.lock_stats_top_10minute s
WHERE
  t.interval_end = "2020-05-17T18:40:00" and s.interval_end = t.interval_end;
```

Here's some example output from our query:

<table>
<colgroup>
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>row_range_start_key</strong></th>
<th><strong>total_lock_wait_seconds</strong></th>
<th><strong>lock_wait_seconds</strong></th>
<th><strong>frac_of_total</strong></th>
<th><strong>sample_lock_requests</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Singers(32)</td>
<td>2.37</td>
<td>1.76</td>
<td>1</td>
<td>LOCK_MODE: WriterShared<br />
COLUMN: Singers.SingerInfo<br />
TRANSACTION_TAG:<br />
app=cart,service=order<br />
<br />
LOCK_MODE: ReaderShared<br />
COLUMN: Singers.SingerInfo<br />
TRANSACTION_TAG:<br />
app=cart,service=redis</td>
</tr>
</tbody>
</table>

From this table of results, we can see the conflict happened on the `  Singers  ` table at key **SingerId=32** . The `  Singers.SingerInfo  ` is the column where the lock conflict happened between `  ReaderShared  ` and `  WriterShared  ` . You can also identify corresponding transactions ( `  app=cart,service=order  ` and `  app=cart,service=redis  ` ) that are experiencing the conflict.

Once the transactions causing the lock conflicts are identified, you can now focus on these transactions by using [Transaction Statistics](/spanner/docs/introspection/transaction-statistics) to get a better sense of what the transactions are doing and if you can avoid a conflict or reduce the time for which the locks are held. For more information, see [Best practices to reduce lock contention](/spanner/docs/introspection/lock-statistics#applying_best_practices_to_reduce_lock_contention) .

## What's next

  - Learn about other [Introspection tools](/spanner/docs/introspection) .
  - Learn about other information Spanner stores for each database in the database's [information schema](/spanner/docs/information-schema) tables.
  - Learn more about [SQL best practices](/spanner/docs/sql-best-practices) for Spanner.
  - Learn more about [Investigating high CPU utilization](/spanner/docs/introspection/investigate-cpu-utilization) .
