Update data in a table containing a TIMESTAMP column by using mutations.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Commit timestamps in GoogleSQL-dialect databases](/spanner/docs/commit-timestamp)
  - [Commit timestamps in PostgreSQL-dialect databases](/spanner/docs/commit-timestamp-postgresql)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void UpdateDataWithTimestamp(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto commit_result = client.Commit(spanner::Mutations{
      spanner::UpdateMutationBuilder(
          "Albums",
          {"SingerId", "AlbumId", "MarketingBudget", "LastUpdateTime"})
          .EmplaceRow(1, 1, 1000000, spanner::CommitTimestamp{})
          .EmplaceRow(2, 2, 750000, spanner::CommitTimestamp{})
          .Build()});
  if (!commit_result) throw std::move(commit_result).status();
  std::cout
      << "Update was successful [spanner_update_data_with_timestamp_column]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class UpdateDataWithTimestampColumnAsyncSample
{
    public async Task<int> UpdateDataWithTimestampColumnAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);

        var rowCount = 0;
        using var updateCmd1 = connection.CreateUpdateCommand("Albums", new SpannerParameterCollection
        {
            { "SingerId", SpannerDbType.Int64, 1 },
            { "AlbumId", SpannerDbType.Int64, 1 },
            { "MarketingBudget", SpannerDbType.Int64, 1000000 },
            { "LastUpdateTime", SpannerDbType.Timestamp, SpannerParameter.CommitTimestamp },
        });
        rowCount += await updateCmd1.ExecuteNonQueryAsync();

        using var updateCmd2 = connection.CreateUpdateCommand("Albums", new SpannerParameterCollection
        {
            { "SingerId", SpannerDbType.Int64, 2 },
            { "AlbumId", SpannerDbType.Int64, 2 },
            { "MarketingBudget", SpannerDbType.Int64, 750000 },
            { "LastUpdateTime", SpannerDbType.Timestamp, SpannerParameter.CommitTimestamp },
        });
        rowCount += await updateCmd2.ExecuteNonQueryAsync();

        Console.WriteLine("Updated data.");
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

func updateWithTimestamp(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 cols := []string{"SingerId", "AlbumId", "MarketingBudget", "LastUpdateTime"}
 _, err = client.Apply(ctx, []*spanner.Mutation{
     spanner.Update("Albums", cols, []interface{}{1, 1, 1000000, spanner.CommitTimestamp}),
     spanner.Update("Albums", cols, []interface{}{2, 2, 750000, spanner.CommitTimestamp}),
 })
 return err
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void updateWithTimestamp(DatabaseClient dbClient) {
  // Mutation can be used to update/insert/delete a single row in a table. Here we use
  // newUpdateBuilder to create update mutations.
  List<Mutation> mutations =
      Arrays.asList(
          Mutation.newUpdateBuilder("Albums")
              .set("SingerId")
              .to(1)
              .set("AlbumId")
              .to(1)
              .set("MarketingBudget")
              .to(1000000)
              .set("LastUpdateTime")
              .to(Value.COMMIT_TIMESTAMP)
              .build(),
          Mutation.newUpdateBuilder("Albums")
              .set("SingerId")
              .to(2)
              .set("AlbumId")
              .to(2)
              .set("MarketingBudget")
              .to(750000)
              .set("LastUpdateTime")
              .to(Value.COMMIT_TIMESTAMP)
              .build());
  // This writes all the mutations to Cloud Spanner atomically.
  dbClient.write(mutations);
}
```

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` javascript
// ...

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

const data = [
  {
    SingerId: '1',
    AlbumId: '1',
    MarketingBudget: '1000000',
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
  {
    SingerId: '2',
    AlbumId: '2',
    MarketingBudget: '750000',
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
];

try {
  await albumsTable.update(data);
  console.log('Updated data.');
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished
  database.close();
}
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Updates sample data in a table with a commit timestamp column.
 *
 * Before executing this method, a new column MarketingBudget has to be added to the Albums
 * table by applying the DDL statement "ALTER TABLE Albums ADD COLUMN MarketingBudget INT64".
 *
 * In addition this update expects the LastUpdateTime column added by applying the DDL statement
 * "ALTER TABLE Albums ADD COLUMN LastUpdateTime TIMESTAMP OPTIONS (allow_commit_timestamp=true)"
 *
 * Example:
 * ```
 * update_data_with_timestamp_column($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function update_data_with_timestamp_column(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $operation = $database->transaction(['singleUse' => true])
        ->updateBatch('Albums', [
            ['SingerId' => 1, 'AlbumId' => 1, 'MarketingBudget' => 1000000, 'LastUpdateTime' => $spanner->commitTimestamp()],
            ['SingerId' => 2, 'AlbumId' => 2, 'MarketingBudget' => 750000, 'LastUpdateTime' => $spanner->commitTimestamp()],
        ])
        ->commit();

    print('Updated data.' . PHP_EOL);
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def update_data_with_timestamp(instance_id, database_id):
    """Updates Performances tables in the database with the COMMIT_TIMESTAMP
    column.

    This updates the `MarketingBudget` column which must be created before
    running this sample. You can add the column by running the `add_column`
    sample or by running this DDL statement against your database:

        ALTER TABLE Albums ADD COLUMN MarketingBudget INT64

    In addition this update expects the LastUpdateTime column added by
    applying this DDL statement against your database:

        ALTER TABLE Albums ADD COLUMN LastUpdateTime TIMESTAMP
        OPTIONS(allow_commit_timestamp=true)
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)

    database = instance.database(database_id)

    with database.batch() as batch:
        batch.update(
            table="Albums",
            columns=("SingerId", "AlbumId", "MarketingBudget", "LastUpdateTime"),
            values=[
                (1, 1, 1000000, spanner.COMMIT_TIMESTAMP),
                (2, 2, 750000, spanner.COMMIT_TIMESTAMP),
            ],
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

commit_timestamp = client.commit_timestamp

client.commit do |c|
  c.update "Albums", [
    { SingerId: 1, AlbumId: 1, MarketingBudget: 100_000, LastUpdateTime: commit_timestamp },
    { SingerId: 2, AlbumId: 2, MarketingBudget: 750_000, LastUpdateTime: commit_timestamp }
  ]
end

puts "Updated data"
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
