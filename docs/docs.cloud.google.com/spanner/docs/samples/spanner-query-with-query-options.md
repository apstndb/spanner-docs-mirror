Specify query options.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Manage the query optimizer](/spanner/docs/query-optimizer/manage-query-optimizer)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void QueryWithQueryOptions(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto sql = spanner::SqlStatement("SELECT SingerId, FirstName FROM Singers");
  auto opts =
      google::cloud::Options{}
          .set<spanner::QueryOptimizerVersionOption>("1")
          .set<spanner::QueryOptimizerStatisticsPackageOption>("latest");
  auto rows = client.ExecuteQuery(std::move(sql), std::move(opts));

  using RowType = std::tuple<std::int64_t, std::string>;
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row) << "\t";
    std::cout << "FirstName: " << std::get<1>(*row) << "\n";
  }
  std::cout << "Read completed for [spanner_query_with_query_options]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class RunCommandWithQueryOptionsAsyncSample
{
    public class Album
    {
        public int SingerId { get; set; }
        public int AlbumId { get; set; }
        public string AlbumTitle { get; set; }
    }

    public async Task<List<Album>> RunCommandWithQueryOptionsAsync(string projectId, string instanceId, string databaseId)
    {
        var connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand("SELECT SingerId, AlbumId, AlbumTitle FROM Albums");

        cmd.QueryOptions = QueryOptions.Empty
            .WithOptimizerVersion("1")
            // The list of available statistics packages for the database can
            // be found by querying the "INFORMATION_SCHEMA.SPANNER_STATISTICS"
            // table.
            .WithOptimizerStatisticsPackage("latest");
        var albums = new List<Album>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            albums.Add(new Album()
            {
                AlbumId = reader.GetFieldValue<int>("AlbumId"),
                SingerId = reader.GetFieldValue<int>("SingerId"),
                AlbumTitle = reader.GetFieldValue<string>("AlbumTitle")
            });
        }
        return albums;
    }
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
 sppb "cloud.google.com/go/spanner/apiv1/spannerpb"
 "google.golang.org/api/iterator"
)

func queryWithQueryOptions(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{SQL: `SELECT VenueId, VenueName, LastUpdateTime FROM Venues`}
 queryOptions := spanner.QueryOptions{
     Options: &sppb.ExecuteSqlRequest_QueryOptions{
         OptimizerVersion: "1",
         // The list of available statistics packages can be found by
         // querying the "INFORMATION_SCHEMA.SPANNER_STATISTICS" table.
         OptimizerStatisticsPackage: "latest",
     },
 }
 iter := client.Single().QueryWithOptions(ctx, stmt, queryOptions)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var venueID int64
     var venueName string
     var lastUpdateTime time.Time
     if err := row.Columns(&venueID, &venueName, &lastUpdateTime); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %s %s\n", venueID, venueName, lastUpdateTime)
 }
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void queryWithQueryOptions(DatabaseClient dbClient) {
  try (ResultSet resultSet =
      dbClient
          .singleUse()
          .executeQuery(
              Statement
                  .newBuilder("SELECT SingerId, AlbumId, AlbumTitle FROM Albums")
                  .withQueryOptions(QueryOptions
                      .newBuilder()
                      .setOptimizerVersion("1")
                      // The list of available statistics packages can be found by querying the
                      // "INFORMATION_SCHEMA.SPANNER_STATISTICS" table.
                      .setOptimizerStatisticsPackage("latest")
                      .build())
                  .build())) {
    while (resultSet.next()) {
      System.out.printf(
          "%d %d %s\n", resultSet.getLong(0), resultSet.getLong(1), resultSet.getString(2));
    }
  }
}
```

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

const query = {
  sql: `SELECT AlbumId, AlbumTitle, MarketingBudget
        FROM Albums
        ORDER BY AlbumTitle`,
  queryOptions: {
    optimizerVersion: 'latest',
    // The list of available statistics packages can be found by querying the
    // "INFORMATION_SCHEMA.SPANNER_STATISTICS" table.
    optimizerStatisticsPackage: 'latest',
  },
};

// Queries rows from the Albums table
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    const marketingBudget = json.MarketingBudget
      ? json.MarketingBudget
      : null; // This value is nullable
    console.log(
      `AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}, MarketingBudget: ${marketingBudget}`,
    );
  });
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  database.close();
}
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Database;

/**
 * Queries sample data using SQL with query options.
 * Example:
 * ```
 * query_data_with_query_options($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_query_options(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $results = $database->execute(
        'SELECT VenueId, VenueName, LastUpdateTime FROM Venues',
        [
            'queryOptions' => [
                'optimizerVersion' => '1',
                // Pin the statistics package to the latest version just for
                // this query.
                'optimizerStatisticsPackage' => 'latest'
            ]
        ]
    );

    foreach ($results as $row) {
        printf('VenueId: %s, VenueName: %s, LastUpdateTime: %s' . PHP_EOL,
            $row['VenueId'], $row['VenueName'], $row['LastUpdateTime']);
    }
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT VenueId, VenueName, LastUpdateTime FROM Venues",
        query_options={
            "optimizer_version": "1",
            "optimizer_statistics_package": "latest",
        },
    )

    for row in results:
        print("VenueId: {}, VenueName: {}, LastUpdateTime: {}".format(*row))
```

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

sql_query = "SELECT VenueId, VenueName, LastUpdateTime FROM Venues"
query_options = {
  optimizer_version: "1",
  # The list of available statistics packagebs can be
  # found by querying the "INFORMATION_SCHEMA.SPANNER_STATISTICS"
  # table.
  optimizer_statistics_package: "latest"
}

client.execute(sql_query, query_options: query_options).rows.each do |row|
  puts "#{row[:VenueId]} #{row[:VenueName]} #{row[:LastUpdateTime]}"
end
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
