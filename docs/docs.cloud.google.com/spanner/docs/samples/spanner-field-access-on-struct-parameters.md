Query data using field access on a STRUCT.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Working with STRUCT objects](/spanner/docs/structs)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void FieldAccessOnStructParameters(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  // Cloud Spanner STRUCT<> with named fields is represented as
  // tuple<pair<string, T>...>. Create a type alias for this example:
  using SingerName = std::tuple<std::pair<std::string, std::string>,
                                std::pair<std::string, std::string>>;
  SingerName name({"FirstName", "Elena"}, {"LastName", "Campbell"});

  auto rows = client.ExecuteQuery(spanner::SqlStatement(
      "SELECT SingerId FROM Singers WHERE FirstName = @name.FirstName",
      {{"name", spanner::Value(name)}}));

  for (auto& row : spanner::StreamOf<std::tuple<std::int64_t>>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row) << "\n";
  }
  std::cout << "Query completed for"
            << " [spanner_field_access_on_struct_parameters]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class QueryDataWithStructFieldAsyncSample
{
    public async Task<List<int>> QueryDataWithStructFieldAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        var structParam = new SpannerStruct
        {
            { "FirstName", SpannerDbType.String, "Elena" },
            { "LastName", SpannerDbType.String, "Campbell" },
        };

        var singerIds = new List<int>();
        using var connection = new SpannerConnection(connectionString);
        using var cmd = connection.CreateSelectCommand("SELECT SingerId FROM Singers WHERE FirstName = @name.FirstName");

        cmd.Parameters.Add("name", structParam.GetSpannerDbType(), structParam);
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            singerIds.Add(reader.GetFieldValue<int>("SingerId"));
        }
        return singerIds;
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

func queryWithStructField(w io.Writer, db string) error {
 ctx := context.Background()

 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 type structParam struct {
     FirstName string
     LastName  string
 }
 var singerInfo = structParam{"Elena", "Campbell"}
 stmt := spanner.Statement{
     SQL: `SELECT SingerId FROM SINGERS
         WHERE FirstName = @name.FirstName`,
     Params: map[string]interface{}{"name": singerInfo},
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
     var singerID int64
     if err := row.Columns(&singerID); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d\n", singerID)
 }
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void queryStructField(DatabaseClient dbClient) {
  Statement s =
      Statement.newBuilder("SELECT SingerId FROM Singers WHERE FirstName = @name.FirstName")
          .bind("name")
          .to(
              Struct.newBuilder()
                  .set("FirstName")
                  .to("Elena")
                  .set("LastName")
                  .to("Campbell")
                  .build())
          .build();
  try (ResultSet resultSet = dbClient.singleUse().executeQuery(s)) {
    while (resultSet.next()) {
      System.out.printf("%d\n", resultSet.getLong("SingerId"));
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

const nameStruct = Spanner.struct({
  FirstName: 'Elena',
  LastName: 'Campbell',
});
const query = {
  sql: 'SELECT SingerId FROM Singers WHERE FirstName = @name.FirstName',
  params: {
    name: nameStruct,
  },
};

// Queries rows from the Singers table
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(`SingerId: ${json.SingerId}`);
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
use Google\Cloud\Spanner\StructType;

/**
 * Queries sample data from the database using a struct field value.
 * Example:
 * ```
 * query_data_with_struct_field($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_struct_field(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $nameType = (new StructType)
        ->add('FirstName', Database::TYPE_STRING)
        ->add('LastName', Database::TYPE_STRING);
    $results = $database->execute(
        'SELECT SingerId FROM Singers WHERE FirstName = @name.FirstName',
        [
            'parameters' => [
                'name' => [
                    'FirstName' => 'Elena',
                    'LastName' => 'Campbell'
                ]
            ],
            'types' => [
                'name' => $nameType
            ]
        ]
    );
    foreach ($results as $row) {
        printf('SingerId: %s' . PHP_EOL,
            $row['SingerId']);
    }
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def query_struct_field(instance_id, database_id):
    """Query a table using field access on a STRUCT parameter."""
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    name_type = param_types.Struct(
        [
            param_types.StructField("FirstName", param_types.STRING),
            param_types.StructField("LastName", param_types.STRING),
        ]
    )

    with database.snapshot() as snapshot:
        results = snapshot.execute_sql(
            "SELECT SingerId FROM Singers " "WHERE FirstName = @name.FirstName",
            params={"name": ("Elena", "Campbell")},
            param_types={"name": name_type},
        )

    for row in results:
        print("SingerId: {}".format(*row))
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

name_struct = { FirstName: "Elena", LastName: "Campbell" }
client.execute(
  "SELECT SingerId FROM Singers WHERE FirstName = @name.FirstName",
  params: { name: name_struct }
).rows.each do |row|
  puts row[:SingerId]
end
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
