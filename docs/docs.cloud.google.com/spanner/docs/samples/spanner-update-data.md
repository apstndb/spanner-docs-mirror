Update data by using mutations.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Getting started with Spanner and PGAdapter](/spanner/docs/getting-started/pgadapter)
  - [Getting started with Spanner in C\#](/spanner/docs/getting-started/csharp)
  - [Getting started with Spanner in C++](/spanner/docs/getting-started/cpp)
  - [Getting started with Spanner in Go](/spanner/docs/getting-started/go)
  - [Getting started with Spanner in Go database/sql](/spanner/docs/getting-started/database_sql)
  - [Getting started with Spanner in Java](/spanner/docs/getting-started/java)
  - [Getting started with Spanner in JDBC](/spanner/docs/getting-started/jdbc)
  - [Getting started with Spanner in Node.js](/spanner/docs/getting-started/nodejs)
  - [Getting started with Spanner in PHP](/spanner/docs/getting-started/php)
  - [Getting started with Spanner in Python](/spanner/docs/getting-started/python)
  - [Getting started with Spanner in Ruby](/spanner/docs/getting-started/ruby)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void UpdateData(google::cloud::spanner::Client client) {
  //! [commit-with-mutations]
  namespace spanner = ::google::cloud::spanner;
  auto commit_result = client.Commit(spanner::Mutations{
      spanner::UpdateMutationBuilder("Albums",
                                     {"SingerId", "AlbumId", "MarketingBudget"})
          .EmplaceRow(1, 1, 100000)
          .EmplaceRow(2, 2, 500000)
          .Build()});
  if (!commit_result) throw std::move(commit_result).status();
  //! [commit-with-mutations]
  std::cout << "Update was successful [spanner_update_data]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class UpdateDataAsyncSample
{
    public async Task<int> UpdateDataAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);

        var rowCount = 0;
        SpannerCommand cmd = connection.CreateDmlCommand(
            "UPDATE Albums SET MarketingBudget = @MarketingBudget "
            + "WHERE SingerId = 1 and AlbumId = 1");
        cmd.Parameters.Add("MarketingBudget", SpannerDbType.Int64, 100000);
        rowCount += await cmd.ExecuteNonQueryAsync();

        cmd = connection.CreateDmlCommand(
            "UPDATE Albums SET MarketingBudget = @MarketingBudget "
            + "WHERE SingerId = 2 and AlbumId = 2");
        cmd.Parameters.Add("MarketingBudget", SpannerDbType.Int64, 500000);
        rowCount += await cmd.ExecuteNonQueryAsync();

        Console.WriteLine("Data Updated.");
        return rowCount;
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

func update(w io.Writer, db string) error {
 ctx := context.Background()

 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 cols := []string{"SingerId", "AlbumId", "MarketingBudget"}
 _, err = client.Apply(ctx, []*spanner.Mutation{
     spanner.Update("Albums", cols, []interface{}{1, 1, 100000}),
     spanner.Update("Albums", cols, []interface{}{2, 2, 500000}),
 })
 return err
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void updateDataWithMutations(
    final String project,
    final String instance,
    final String database,
    final Properties properties) throws SQLException {
  try (Connection connection =
      DriverManager.getConnection(
          String.format(
              "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
              project, instance, database),
          properties)) {
    // Unwrap the CloudSpannerJdbcConnection interface
    // from the java.sql.Connection.
    CloudSpannerJdbcConnection cloudSpannerJdbcConnection =
        connection.unwrap(CloudSpannerJdbcConnection.class);

    final long marketingBudgetAlbum1 = 100000L;
    final long marketingBudgetAlbum2 = 500000L;
    // Mutation can be used to update/insert/delete a single row in a table.
    // Here we use newUpdateBuilder to create update mutations.
    List<Mutation> mutations =
        Arrays.asList(
            Mutation.newUpdateBuilder("Albums")
                .set("SingerId")
                .to(1)
                .set("AlbumId")
                .to(1)
                .set("MarketingBudget")
                .to(marketingBudgetAlbum1)
                .build(),
            Mutation.newUpdateBuilder("Albums")
                .set("SingerId")
                .to(2)
                .set("AlbumId")
                .to(2)
                .set("MarketingBudget")
                .to(marketingBudgetAlbum2)
                .build());
    // This writes all the mutations to Cloud Spanner atomically.
    cloudSpannerJdbcConnection.write(mutations);
    System.out.println("Updated albums");
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

// Update a row in the Albums table
// Note: Cloud Spanner interprets Node.js numbers as FLOAT64s, so they
// must be converted to strings before being inserted as INT64s
const albumsTable = database.table('Albums');

try {
  await albumsTable.update([
    {SingerId: '1', AlbumId: '1', MarketingBudget: '100000'},
    {SingerId: '2', AlbumId: '2', MarketingBudget: '500000'},
  ]);
  console.log('Updated data.');
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
 * Updates sample data in the database.
 *
 * This updates the `MarketingBudget` column which must be created before
 * running this sample. You can add the column by running the `add_column`
 * sample or by running this DDL statement against your database:
 *
 *     ALTER TABLE Albums ADD COLUMN MarketingBudget INT64
 *
 * Example:
 * ```
 * update_data($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function update_data(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $operation = $database->transaction(['singleUse' => true])
        ->updateBatch('Albums', [
            ['SingerId' => 1, 'AlbumId' => 1, 'MarketingBudget' => 100000],
            ['SingerId' => 2, 'AlbumId' => 2, 'MarketingBudget' => 500000],
        ])
        ->commit();

    print('Updated data.' . PHP_EOL);
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def update_data(instance_id, database_id):
    """Updates sample data in the database.

    This updates the `MarketingBudget` column which must be created before
    running this sample. You can add the column by running the `add_column`
    sample or by running this DDL statement against your database:

        ALTER TABLE Albums ADD COLUMN MarketingBudget INT64

    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.batch() as batch:
        batch.update(
            table="Albums",
            columns=("SingerId", "AlbumId", "MarketingBudget"),
            values=[(1, 1, 100000), (2, 2, 500000)],
        )

    print("Updated data.")
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
  c.update "Albums", [
    { SingerId: 1, AlbumId: 1, MarketingBudget: 100_000 },
    { SingerId: 2, AlbumId: 2, MarketingBudget: 500_000 }
  ]
end

puts "Updated data"
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
