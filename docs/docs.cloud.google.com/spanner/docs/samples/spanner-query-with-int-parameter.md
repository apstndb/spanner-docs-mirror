Query data by using an INT parameter.

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void QueryWithIntParameter(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  int example_int = 3000;
  spanner::SqlStatement select(
      "SELECT VenueId, VenueName, Capacity FROM Venues"
      " WHERE Capacity >= @capacity",
      {{"capacity", spanner::Value(example_int)}});
  using RowType = std::tuple<std::int64_t, absl::optional<std::string>,
                             absl::optional<std::int64_t>>;
  auto rows = client.ExecuteQuery(std::move(select));
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "VenueId: " << std::get<0>(*row) << "\t";
    std::cout << "VenueName: " << std::get<1>(*row).value() << "\t";
    std::cout << "Capacity: " << std::get<2>(*row).value() << "\n";
  }
  std::cout << "Query completed for [spanner_query_with_int_parameter]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryWithIntAsyncSample
{
    public class Venue
    {
        public int VenueId { get; set; }
        public string VenueName { get; set; }
        public int Capacity { get; set; }
    }

    public async Task<List<Venue>> QueryWithIntAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        // Initialize a long variable to use for querying.
        long exampleValue = 3000;

        using var connection = new SpannerConnection(connectionString);
        var cmd = connection.CreateSelectCommand("SELECT VenueId, VenueName, Capacity FROM Venues WHERE Capacity >= @ExampleValue");
        cmd.Parameters.Add("ExampleValue", SpannerDbType.Int64, exampleValue);

        var venues = new List<Venue>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            venues.Add(new Venue
            {
                VenueId = reader.GetFieldValue<int>("VenueId"),
                VenueName = reader.GetFieldValue<string>("VenueName"),
                Capacity = reader.GetFieldValue<int>("Capacity")
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

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func queryWithInt(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 var exampleInt = 3000
 stmt := spanner.Statement{
     SQL: `SELECT VenueId, VenueName, Capacity FROM Venues
             WHERE Capacity >= @capacity`,
     Params: map[string]interface{}{
         "capacity": exampleInt,
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
     var venueID, capacity int64
     var venueName string
     if err := row.Columns(&venueID, &venueName, &capacity); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %s %d\n", venueID, venueName, capacity)
 }
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void queryWithInt(DatabaseClient dbClient) {
  long exampleInt = 3000;
  Statement statement =
      Statement.newBuilder(
              "SELECT VenueId, VenueName, Capacity FROM Venues " + "WHERE Capacity >= @capacity")
          .bind("capacity")
          .to(exampleInt)
          .build();
  try (ResultSet resultSet = dbClient.singleUse().executeQuery(statement)) {
    while (resultSet.next()) {
      System.out.printf(
          "%d %s %d\n",
          resultSet.getLong("VenueId"),
          resultSet.getString("VenueName"),
          resultSet.getLong("Capacity"));
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

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database.
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

const fieldType = {
  type: 'int64',
};

const exampleInt = 3000;

const query = {
  sql: `SELECT VenueId, VenueName, Capacity FROM Venues
          WHERE Capacity >= @capacity`,
  params: {
    capacity: exampleInt,
  },
  types: {
    capacity: fieldType,
  },
};

// Queries rows from the Venues table.
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(
      `VenueId: ${json.VenueId}, VenueName: ${json.VenueName},` +
        ` Capacity: ${json.Capacity}`,
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
 * Queries sample data from the database using SQL with a INT64 parameter.
 * Example:
 * ```
 * query_data_with_int_parameter($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_int_parameter(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $exampleInt = 3000;

    $results = $database->execute(
        'SELECT VenueId, VenueName, Capacity FROM Venues ' .
        'WHERE Capacity >= @capacity',
        [
            'parameters' => [
                'capacity' => $exampleInt
            ]
        ]
    );

    foreach ($results as $row) {
        printf('VenueId: %s, VenueName: %s, Capacity: %s' . PHP_EOL,
            $row['VenueId'], $row['VenueName'], $row['Capacity']);
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

exampleInt = 3000
param = {"capacity": exampleInt}
param_type = {"capacity": param_types.INT64}

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT VenueId, VenueName, Capacity FROM Venues "
        "WHERE Capacity >= @capacity",
        params=param,
        param_types=param_type,
    )

    for row in results:
        print("VenueId: {}, VenueName: {}, Capacity: {}".format(*row))
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

sql_query = "SELECT VenueId, VenueName, Capacity FROM Venues
             WHERE Capacity >= @capacity"

params      = { capacity: 3_000 }
param_types = { capacity: :INT64 }

client.execute(sql_query, params: params, types: param_types).rows.each do |row|
  puts "#{row[:VenueId]} #{row[:VenueName]} #{row[:Capacity]}"
end
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
