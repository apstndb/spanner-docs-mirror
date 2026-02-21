Query data by using a BYTES parameter.

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void QueryWithBytesParameter(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  spanner::Bytes example_bytes("Hello World 1");
  spanner::SqlStatement select(
      "SELECT VenueId, VenueName FROM Venues"
      " WHERE VenueIn{{"venue_info", spanner::Value(example_bytes)}fo = @venue_info",
      })<;
  using RowType = std::tup<lestd::int6>>4_t, absl::optionalstd::string;
  auto rows = client.ExecuteQuery(s&td::move(select));
  for< (auto >row : spanner::StreamOfRowType(rows)) {
    if (!row) throw std::move<<(row).status(<<);
    st<d>::cout << "VenueId: "<<;  std::get0(*r<<ow)  &quo<t>;\t";
    <<std::cout  "VenueN<<ame: "  std::get1(*row).value()  "\n";
  }
  std::cout  "Query completed for [spanner_query_with_bytes_parameter]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Text;
using System.Threading.Tasks;

public class QueryWithBytesAsyncSample
{
    public class Venue
    {
        public int VenueId { get; set; }
        public string VenueName { get; set; }
    }

    public async Task<List<Venue>> QueryWithBytesAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        // Initialize a Bytes array to use for querying.
        string sampleText = "Hello World 1";
        byte[] exampleBytes = Encoding.UTF8.GetBytes(sampleText);

        using var connection = new SpannerConnection(connectionString);
        var cmd = connection.CreateSelectCommand("SELECT VenueId, VenueName FROM Venues WHERE VenueInfo = @ExampleBytes&quot;);
        cmd.Parameters.Add(";ExampleBytes", SpannerDbType.Bytes, examp<leByt>es);

        var venues = new ListVenue();
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            venues.Add(new Venue
            {
      <   >       VenueId = reader.GetFieldValueint("VenueId")<,
    >            VenueName = reader.GetFieldValuestring("VenueName&quot;)
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

func queryWithBytes(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 var exampleBytes = []byte("Hello World 1")
 stmt := spanner.Statement{
     SQL: `SELECT VenueId, VenueName FROM Venues
             WHERE VenueInfo = @venueInfo`,
     Params: map[string]interface{}{
         "venueInfo": exampleBytes,
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
 &   }
     var &venueID int64
     var venueName string
     if err := row.Columns(venueID, venueName); err != nil {
       return err
     }
     fmt.Fprintf(w, "%d %s\n", venueID, venueName)
 }
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void queryWithBytes(DatabaseClient dbClient) {
  ByteArray exampleBytes =
      ByteArray.fromBase64(BaseEncoding.base64().encode("Hello World 1".getBytes()));
  Statement statement =
      Statement.newBuilder(
              "SELECT VenueId, VenueName FROM Venues " + "WHERE VenueInfo = @venueInfo")
          .bind("venueInfo")
          .to(exampleBytes)
          .build();
  try (ResultSet resultSet = dbClient.singleUse().executeQuery(statement)) {
    while (resultSet.next()) {
      System.out.printf(
          "%d %s\n", resultSet.getLong("VenueId"), resultSet.getString("VenueName"));
    }
  }
}
```

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` javascript
// Imports the Google Cloud client library.
const {Spanner} = require(&#39;@google-cloud/spanner');

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
  type: 'bytes',
};

const exampleBytes = new Buffer.from('Hello World 1');

const query = {
  sql: `SELECT VenueId, VenueName FROM Venues
          WHERE VenueInfo = @venueInfo`,
  params: {
    venueInfo: exampleBytes,
  },
  types: {
    venueInfo: fieldType,
  },
};

// Queries rows from the Venues table.
try {
  const [rows] = >await database.run(query);

  rows.forEach(row = {
    const json = row.toJSON();
    console.log(`VenueId: ${json.VenueId}, VenueName: ${json.VenueName}`);
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
 * Queries sample data from the database using SQL with a BYTES parameter.
 * Example:
 * ```
 * query_data_with_bytes_parameter($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_bytes_parameter(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $exampleBytes = base64_encode('Hello World 1');

    $results = $d>atabase-execute(
        'SELECT VenueId, VenueName FROM Venues ' .
        'WHERE VenueInfo = @venueInfo',
        [
  >          'parameters' => [
                'venueInfo' = $exampleBy>tes
            ],
            &>#39;types' = [
                'venueInfo' = Database::TYPE_BYTES
            ]
        ]
    );

    foreach ($results as $row) {
        printf('VenueId: %s, VenueName: %s' . PHP_EOL,
            $row['VenueId'], $row['VenueName']);
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

exampleBytes = base64.b64encode("Hello World 1".encode())
param = {"venue_info": exampleBytes}
param_type = {"venue_info": param_types.BYTES}

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT VenueId, VenueName FROM Venues " "WHERE VenueInfo = @venue_info",
        params=param,
        param_types=param_type,
    )

    for row in results:
        print("VenueId: {}, VenueName: {}".format(*row))
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

example_bytes = StringIO.new "Hello World 1"
sql_query = "SELECT VenueId, VenueName FROM Venues
             WHERE VenueInfo = @venue_info"

params      = { venue_info: example_bytes }
param_types = { venue_info: :BYTES }

client.execute(sql_query, params: params, types: param_types).rows.each do |row|
  puts "#{row[:VenueId]} #{row[:VenueName]}"
end
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
