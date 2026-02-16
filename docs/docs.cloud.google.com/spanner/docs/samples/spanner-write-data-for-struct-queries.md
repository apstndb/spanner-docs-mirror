Insert data used for STRUCT queries.

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void WriteDataForStructQueries(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto commit_result = client.Commit(
      spanner::Mutations{spanner::InsertMutationBuilder(
                             "Singers", {"SingerId", "FirstName", "LastName"})
                             .EmplaceRow(6, "Elena", "Campbell")
                             .EmplaceRow(7, "Gabriel", "Wright")
                             .EmplaceRow(8, "Benjamin", "Martinez")
                             .EmplaceRow(9, "Hannah", "Harris")
                             .Build()});
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << "Insert was successful "
            << "[spanner_write_data_for_struct_queries]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public class InsertStructSampleDataAsyncSample
{
    public class Singer
    {
        public int SingerId { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
    }

    public async Task<int> InsertStructSampleDataAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        List<Singer> singers = new List<Singer> {
            new Singer { SingerId = 6, FirstName = "Elena", LastName = "Campbell" },
            new Singer { SingerId = 7, FirstName = "Gabriel", LastName = "Wright" },
            new Singer { SingerId = 8, FirstName = "Benjamin", LastName = "Martinez" },
            new Singer { SingerId = 9, FirstName = "Hannah", LastName = "Harris" }
        };

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        var rows = await Task.WhenAll(singers.Select(singer =>
        {
            var cmd = connection.CreateInsertCommand("Singers",
                new SpannerParameterCollection {
                    { "SingerId", SpannerDbType.Int64, singer.SingerId },
                    { "FirstName", SpannerDbType.String, singer.FirstName },
                    { "LastName", SpannerDbType.String, singer.LastName }
                });
            return cmd.ExecuteNonQueryAsync();
        }));
        return rows.Sum();
    }
}
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
import (
 "context"
 "io"

 "cloud.google.com/go/spanner"
)

func writeStructData(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 singerColumns := []string{"SingerId", "FirstName", "LastName"}
 m := []*spanner.Mutation{
     spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{6, "Elena", "Campbell"}),
     spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{7, "Gabriel", "Wright"}),
     spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{8, "Benjamin", "Martinez"}),
     spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{9, "Hannah", "Harris"}),
 }
 _, err = client.Apply(ctx, m)
 return err
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void writeStructExampleData(DatabaseClient dbClient) {
  final List<Singer> singers =
      Arrays.asList(
          new Singer(6, "Elena", "Campbell"),
          new Singer(7, "Gabriel", "Wright"),
          new Singer(8, "Benjamin", "Martinez"),
          new Singer(9, "Hannah", "Harris"));

  List<Mutation> mutations = new ArrayList<>();
  for (Singer singer : singers) {
    mutations.add(
        Mutation.newInsertBuilder("Singers")
            .set("SingerId")
            .to(singer.singerId)
            .set("FirstName")
            .to(singer.firstName)
            .set("LastName")
            .to(singer.lastName)
            .build());
  }
  dbClient.write(mutations);
  System.out.println("Inserted example data for struct parameter queries.");
}
```

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment and update the following lines before running the sample.
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

// Instantiates Spanner table objects
const singersTable = database.table('Singers');

// Inserts rows into the Singers table
// Note: Cloud Spanner interprets Javascript numbers as FLOAT64s.
// Use strings as shown in this example if you need INT64s.
try {
  const data = [
    {
      SingerId: '6',
      FirstName: 'Elena',
      LastName: 'Campbell',
    },
    {
      SingerId: '7',
      FirstName: 'Gabriel',
      LastName: 'Wright',
    },
    {
      SingerId: '8',
      FirstName: 'Benjamin',
      LastName: 'Martinez',
    },
    {
      SingerId: '9',
      FirstName: 'Hannah',
      LastName: 'Harris',
    },
  ];

  await singersTable.insert(data);
  console.log('Inserted data.');
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

/**
 * Inserts sample data that can be used to test STRUCT parameters in queries.
 *
 * The database and table must already exist and can be created using
 * `create_database`.
 * Example:
 * ```
 * insert_struct_data($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function insert_struct_data(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $operation = $database->transaction(['singleUse' => true])
        ->insertBatch('Singers', [
            ['SingerId' => 6, 'FirstName' => 'Elena', 'LastName' => 'Campbell'],
            ['SingerId' => 7, 'FirstName' => 'Gabriel', 'LastName' => 'Wright'],
            ['SingerId' => 8, 'FirstName' => 'Benjamin', 'LastName' => 'Martinez'],
            ['SingerId' => 9, 'FirstName' => 'Hannah', 'LastName' => 'Harris'],
        ])
        ->commit();

    print('Inserted data.' . PHP_EOL);
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def write_struct_data(instance_id, database_id):
    """Inserts sample data that can be used to test STRUCT parameters
    in queries.
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.batch() as batch:
        batch.insert(
            table="Singers",
            columns=("SingerId", "FirstName", "LastName"),
            values=[
                (6, "Elena", "Campbell"),
                (7, "Gabriel", "Wright"),
                (8, "Benjamin", "Martinez"),
                (9, "Hannah", "Harris"),
            ],
        )

    print("Inserted sample data for STRUCT queries")
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

client.commit do |c|
  c.insert "Singers", [
    { SingerId: 6, FirstName: "Elena",    LastName: "Campbell" },
    { SingerId: 7, FirstName: "Gabriel",  LastName: "Wright"    },
    { SingerId: 8, FirstName: "Benjamin", LastName: "Martinez"  },
    { SingerId: 9, FirstName: "Hannah",   LastName: "Harris" }
  ]
end
puts "Inserted Data for Struct queries"
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
