Query data by using a TIMESTAMP parameter.

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void QueryWithTimestampParameter(
    google::cloud::spanner::Client client,
    google::cloud::spanner::Timestamp example_timestamp) {
  namespace spanner = ::google::cloud::spanner;
  spanner::SqlStatement select(
      "SELECT VenueId, VenueName, LastUpdateTime FROM Venues"
      " WHERE LastUpdateTime <= @last_update_time",
      {{"last_update_time", spanner::Value(example_timestamp)}});
  using RowType = std::tuple<std::int64_t, absl::optional<std::string>,
                             absl::optional<spanner::Timestamp>>;
  auto rows = client.ExecuteQuery(std::move(select));
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "VenueId: " << std::get<0>(*row) << "\t";
    std::cout << "VenueName: " << std::get<1>(*row).value() << "\t";
    std::cout << "LastUpdateTime: " << std::get<2>(*row).value() << "\n";
  }
  std::cout << "Query completed for [spanner_query_with_timestamp_parameter]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryWithTimestampAsyncSample
{
    public class Venue
    {
        public int VenueId { get; set; }
        public string VenueName { get; set; }
        public DateTime? LastUpdateTime { get; set; }
    }

    public async Task<List<Venue>> QueryWithTimestampAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        // Initialize a DateTime timestamp variable to use for querying.
        DateTime exampleTimestamp = DateTime.UtcNow;

        using var connection = new SpannerConnection(connectionString);
        var cmd = connection.CreateSelectCommand("SELECT VenueId, VenueName, LastUpdateTime FROM Venues WHERE LastUpdateTime < @ExampleTimestamp");
        cmd.Parameters.Add("ExampleTimestamp", SpannerDbType.Timestamp, exampleTimestamp);

        var venues = new List<Venue>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            venues.Add(new Venue
            {
                VenueId = reader.GetFieldValue<int>("VenueId"),
                VenueName = reader.GetFieldValue<string>("VenueName"),
                LastUpdateTime = reader["LastUpdateTime"] != DBNull.Value ? reader.GetFieldValue<DateTime?>("LastUpdateTime") : null
            });
        }
        return venues;
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
 "google.golang.org/api/iterator"
)

func queryWithTimestampParameter(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 var exampleTimestamp = time.Now()
 stmt := spanner.Statement{
     SQL: `SELECT VenueId, VenueName, LastUpdateTime FROM Venues
     WHERE LastUpdateTime <= @lastUpdateTime`,
     Params: map[string]interface{}{
         "lastUpdateTime": exampleTimestamp,
     },
 }
 iter := client.Single().Query(ctx, stmt)
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
static void queryWithTimestampParameter(DatabaseClient dbClient) {
  Instant exampleTimestamp = Instant.now();
  Statement statement =
      Statement.newBuilder(
              "SELECT VenueId, VenueName, LastUpdateTime FROM Venues "
                  + "WHERE LastUpdateTime < @lastUpdateTime")
          .bind("lastUpdateTime")
          .to(exampleTimestamp.toString())
          .build();
  try (ResultSet resultSet = dbClient.singleUse().executeQuery(statement)) {
    while (resultSet.next()) {
      System.out.printf(
          "%d %s %s\n",
          resultSet.getLong("VenueId"),
          resultSet.getString("VenueName"),
          resultSet.getTimestamp("LastUpdateTime"));
    }
  }
}
```

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` javascript
// Imports the Google Cloud client library.
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client.
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database.
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

const fieldType = {
  type: 'timestamp',
};

const exampleTimestamp = new Date().toISOString();

const query = {
  sql: `SELECT VenueId, VenueName, LastUpdateTime FROM Venues
          WHERE LastUpdateTime < @lastUpdateTime`,
  params: {
    lastUpdateTime: exampleTimestamp,
  },
  types: {
    lastUpdateTime: fieldType,
  },
};

// Queries rows from the Venues table.
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(
      `VenueId: ${json.VenueId}, VenueName: ${json.VenueName},` +
        ` LastUpdateTime: ${json.LastUpdateTime}`,
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
 * Queries sample data from the database using SQL with a TIMESTAMP parameter.
 * Example:
 * ```
 * query_data_with_timestamp_parameter($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_timestamp_parameter(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $exampleTimestamp = gmdate('Y-m-d\TH:i:s.u\Z');

    $results = $database->execute(
        'SELECT VenueId, VenueName, LastUpdateTime FROM Venues ' .
        'WHERE LastUpdateTime < @lastUpdateTime',
        [
            'parameters' => [
                'lastUpdateTime' => $exampleTimestamp
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

example_timestamp = datetime.datetime.utcnow().isoformat() + "Z"
param = {"last_update_time": example_timestamp}
param_type = {"last_update_time": param_types.TIMESTAMP}

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT VenueId, VenueName, LastUpdateTime FROM Venues "
        "WHERE LastUpdateTime < @last_update_time",
        params=param,
        param_types=param_type,
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

example_timestamp = DateTime.now
sql_query = "SELECT VenueId, VenueName, LastUpdateTime FROM Venues
             WHERE LastUpdateTime < @last_update_time"

params      = { last_update_time: example_timestamp }
param_types = { last_update_time: :TIMESTAMP }

client.execute(sql_query, params: params, types: param_types).rows.each do |row|
  puts "#{row[:VenueId]} #{row[:VenueName]} #{row[:LastUpdateTime]}"
end
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
