Query data by using a DATE parameter.

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void QueryWithDateParameter(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  absl::CivilDay example_date(2019, 1, 1);
  spanner::SqlStatement select(
      "SELECT VenueId, VenueName, LastContactDate FROM Venues"
      " WHERE LastContactDate < @last_contact_date",
      {{"last_contact_date", spanner::Value(example_date)}});
  using RowType = std::tuple<std::int64_t, absl::optional<std::string>,
                             absl::optional<absl::CivilDay>>;
  auto rows = client.ExecuteQuery(std::move(select));
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "VenueId: " << std::get<0>(*row) << "\t";
    std::cout << "VenueName: " << std::get<1>(*row).value() << "\t";
    std::cout << "LastContactDate: " << std::get<2>(*row).value() << "\n";
  }
  std::cout << "Query completed for [spanner_query_with_date_parameter]\n";
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

public class QueryWithDateAsyncSample
{
    public class Venue
    {
        public int VenueId { get; set; }
        public string VenueName { get; set; }
        public DateTime LastContactDate { get; set; }
    }

    public async Task<List<Venue>> QueryWithDateAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        // Initialize a Date variable to use for querying.
        DateTime exampleDate = new DateTime(2019, 01, 01);

        using var connection = new SpannerConnection(connectionString);
        var cmd = connection.CreateSelectCommand("SELECT VenueId, VenueName, LastContactDate FROM Venues WHERE LastContactDate < @ExampleDate");
        cmd.Parameters.Add("ExampleDate", SpannerDbType.Date, exampleDate);

        var venues = new List<Venue>();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            venues.Add(new Venue
            {
                VenueId = reader.GetFieldValue<int>("VenueId"),
                VenueName = reader.GetFieldValue<string>("VenueName"),
                LastContactDate = reader.GetFieldValue<DateTime>("LastContactDate")
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

func queryWithDate(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 var exampleDate = civil.Date{Year: 2019, Month: time.January, Day: 1}
 stmt := spanner.Statement{
     SQL: `SELECT VenueId, VenueName, LastContactDate FROM Venues
             WHERE LastContactDate < @lastContactDate`,
     Params: map[string]interface{}{
         "lastContactDate": exampleDate,
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
     var lastContactDate civil.Date
     if err := row.Columns(&venueID, &venueName, &lastContactDate); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %s %v\n", venueID, venueName, lastContactDate)
 }
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void queryWithDate(DatabaseClient dbClient) {
  String exampleDate = "2019-01-01";
  Statement statement =
      Statement.newBuilder(
              "SELECT VenueId, VenueName, LastContactDate FROM Venues "
                  + "WHERE LastContactDate < @lastContactDate")
          .bind("lastContactDate")
          .to(exampleDate)
          .build();
  try (ResultSet resultSet = dbClient.singleUse().executeQuery(statement)) {
    while (resultSet.next()) {
      System.out.printf(
          "%d %s %s\n",
          resultSet.getLong("VenueId"),
          resultSet.getString("VenueName"),
          resultSet.getDate("LastContactDate"));
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

const exampleDate = '2019-01-01';

const query = {
  sql: `SELECT VenueId, VenueName, LastContactDate FROM Venues
          WHERE LastContactDate < @lastContactDate`,
  params: {
    lastContactDate: exampleDate,
  },
  types: {
    lastContactDate: fieldType,
  },
};

// Queries rows from the Venues table.
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const date = row[2]['value'];
    const json = row.toJSON();
    console.log(
      `VenueId: ${json.VenueId}, VenueName: ${json.VenueName},` +
        ` LastContactDate: ${JSON.stringify(date).substring(1, 11)}`,
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
 * Queries sample data from the database using SQL with a DATE parameter.
 * Example:
 * ```
 * query_data_with_date_parameter($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_date_parameter(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $exampleDate = '2019-01-01';

    $results = $database->execute(
        'SELECT VenueId, VenueName, LastContactDate FROM Venues ' .
        'WHERE LastContactDate < @lastContactDate',
        [
            'parameters' => [
                'lastContactDate' => $exampleDate
            ]
        ]
    );

    foreach ($results as $row) {
        printf('VenueId: %s, VenueName: %s, LastContactDate: %s' . PHP_EOL,
            $row['VenueId'], $row['VenueName'], $row['LastContactDate']);
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

exampleDate = "2019-01-01"
param = {"last_contact_date": exampleDate}
param_type = {"last_contact_date": param_types.DATE}

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT VenueId, VenueName, LastContactDate FROM Venues "
        "WHERE LastContactDate < @last_contact_date",
        params=param,
        param_types=param_type,
    )

    for row in results:
        print("VenueId: {}, VenueName: {}, LastContactDate: {}".format(*row))
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

sql_query = "SELECT VenueId, VenueName, LastContactDate FROM Venues
             WHERE LastContactDate < @last_contact_date"

params      = { last_contact_date: "2019-01-01" }
param_types = { last_contact_date: :DATE }

client.execute(sql_query, params: params, types: param_types).rows.each do |row|
  puts "#{row[:VenueId]} #{row[:VenueName]} #{row[:LastContactDate]}"
end
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
