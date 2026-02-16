Query data by using an ARRAY parameter.

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void QueryWithArrayParameter(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  std::vector<absl::CivilDay> example_array = {absl::CivilDay(2020, 10, 1),
                                               absl::CivilDay(2020, 11, 1)};
  spanner::SqlStatement select(
      "SELECT VenueId, VenueName, AvailableDate FROM Venues v,"
      " UNNEST(v.AvailableDates) as AvailableDate "
      " WHERE AvailableDate in UNNEST(@available_dates)",
      {{"available_dates", spanner::Value(example_array)}});
  using RowType = std::tuple<std::int64_t, absl::optional<std::string>,
                             absl::optional<absl::CivilDay>>;
  auto rows = client.ExecuteQuery(std::move(select));
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "VenueId: " << std::get<0>(*row) << "\t";
    std::cout << "VenueName: " << std::get<1>(*row).value() << "\t";
    std::cout << "AvailableDate: " << std::get<2>(*row).value() << "\n";
  }
  std::cout << "Query completed for [spanner_query_with_array_parameter]\n";
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

public class QueryWithArrayAsyncSample
{
    public class Venue
    {
        public int VenueId { get; set; }
        public string VenueName { get; set; }
        public List<DateTime> AvailableDates { get; set; }
    }

    public async Task<List<Venue>> QueryWithArrayAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        // Initialize a list of dates to use for querying.
        var exampleList = new List<DateTime>
        {
            DateTime.Parse("2020-10-01"),
            DateTime.Parse("2020-11-01")
        };

        using var connection = new SpannerConnection(connectionString);
        var cmd = connection.CreateSelectCommand(
            "SELECT VenueId, VenueName, AvailableDate FROM Venues v, "
            + "UNNEST(v.AvailableDates) as AvailableDate "
            + "WHERE AvailableDate in UNNEST(@ExampleArray)");
        cmd.Parameters.Add("ExampleArray", SpannerDbType.ArrayOf(SpannerDbType.Date), exampleList);

        var venues = new List<Venue>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            venues.Add(new Venue
            {
                VenueId = reader.GetFieldValue<int>("VenueId"),
                VenueName = reader.GetFieldValue<string>("VenueName"),
                AvailableDates = new List<DateTime> { reader.GetFieldValue<DateTime>("AvailableDate") }
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

 "cloud.google.com/go/civil"
 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func queryWithArray(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 var date1 = civil.Date{Year: 2020, Month: time.October, Day: 1}
 var date2 = civil.Date{Year: 2020, Month: time.November, Day: 1}
 var exampleArray = []civil.Date{date1, date2}
 stmt := spanner.Statement{
     SQL: `SELECT VenueId, VenueName, AvailableDate FROM Venues v,
             UNNEST(v.AvailableDates) as AvailableDate 
             WHERE AvailableDate IN UNNEST(@availableDates)`,
     Params: map[string]interface{}{
         "availableDates": exampleArray,
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
     var availableDate civil.Date
     if err := row.Columns(&venueID, &venueName, &availableDate); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %s %s\n", venueID, venueName, availableDate)
 }
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void queryWithArray(DatabaseClient dbClient) {
  Value exampleArray =
      Value.dateArray(Arrays.asList(Date.parseDate("2020-10-01"), Date.parseDate("2020-11-01")));

  Statement statement =
      Statement.newBuilder(
              "SELECT VenueId, VenueName, AvailableDate FROM Venues v, "
                  + "UNNEST(v.AvailableDates) as AvailableDate "
                  + "WHERE AvailableDate in UNNEST(@availableDates)")
          .bind("availableDates")
          .to(exampleArray)
          .build();
  try (ResultSet resultSet = dbClient.singleUse().executeQuery(statement)) {
    while (resultSet.next()) {
      System.out.printf(
          "%d %s %s\n",
          resultSet.getLong("VenueId"),
          resultSet.getString("VenueName"),
          resultSet.getDate("AvailableDate"));
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
  type: 'date',
};

const parentFieldType = {
  type: 'array',
  child: fieldType,
};

const exampleArray = ['2020-10-01', '2020-11-01'];

const query = {
  sql: `SELECT VenueId, VenueName, AvailableDate FROM Venues v,
          UNNEST(v.AvailableDates) as AvailableDate
          WHERE AvailableDate in UNNEST(@availableDates)`,
  params: {
    availableDates: exampleArray,
  },
  types: {
    availableDates: parentFieldType,
  },
};

// Queries rows from the Venues table.
try {
  const [rows] = await database.run(query);
  rows.forEach(row => {
    const availableDate = row[2]['value'];
    const json = row.toJSON();
    console.log(
      `VenueId: ${json.VenueId}, VenueName: ${
        json.VenueName
      }, AvailableDate: ${JSON.stringify(availableDate).substring(1, 11)}`,
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
use Google\Cloud\Spanner\Date;

/**
 * Queries sample data from the database using SQL with an ARRAY parameter.
 * Example:
 * ```
 * query_data_with_array_parameter($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_array_parameter(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $exampleArray = [
        new Date(new \DateTime('2020-10-01')),
        new Date(new \DateTime('2020-11-01'))
    ];

    $results = $database->execute(
        'SELECT VenueId, VenueName, AvailableDate FROM Venues v, ' .
        'UNNEST(v.AvailableDates) as AvailableDate ' .
        'WHERE AvailableDate in UNNEST(@availableDates)',
        [
            'parameters' => [
                'availableDates' => $exampleArray
            ]
        ]
    );

    foreach ($results as $row) {
        printf('VenueId: %s, VenueName: %s, AvailableDate: %s' . PHP_EOL,
            $row['VenueId'], $row['VenueName'], $row['AvailableDate']);
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

exampleArray = ["2020-10-01", "2020-11-01"]
param = {"available_dates": exampleArray}
param_type = {"available_dates": param_types.Array(param_types.DATE)}

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT VenueId, VenueName, AvailableDate FROM Venues v,"
        "UNNEST(v.AvailableDates) as AvailableDate "
        "WHERE AvailableDate in UNNEST(@available_dates)",
        params=param,
        param_types=param_type,
    )

    for row in results:
        print("VenueId: {}, VenueName: {}, AvailableDate: {}".format(*row))
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

sql_query = "SELECT VenueId, VenueName, AvailableDate FROM Venues v,
             UNNEST(v.AvailableDates) as AvailableDate
             WHERE AvailableDate in UNNEST(@available_dates)"

params      = { available_dates: ["2020-10-01", "2020-11-01"] }
param_types = { available_dates: [:DATE] }

client.execute(sql_query, params: params, types: param_types).rows.each do |row|
  puts "#{row[:VenueId]} #{row[:VenueName]} #{row[:AvailableDate]}"
end
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
