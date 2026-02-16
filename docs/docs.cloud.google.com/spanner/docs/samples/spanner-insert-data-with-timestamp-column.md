Write data into a table containing a TIMESTAMP column by using mutations.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Commit timestamps in PostgreSQL-dialect databases](/spanner/docs/commit-timestamp-postgresql)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void InsertDataWithTimestamp(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto commit_result = client.Commit(spanner::Mutations{
      spanner::InsertOrUpdateMutationBuilder(
          "Performances",
          {"SingerId", "VenueId", "EventDate", "Revenue", "LastUpdateTime"})
          .EmplaceRow(1, 4, absl::CivilDay(2017, 10, 5), 11000,
                      spanner::CommitTimestamp{})
          .EmplaceRow(1, 19, absl::CivilDay(2017, 11, 2), 15000,
                      spanner::CommitTimestamp{})
          .EmplaceRow(2, 42, absl::CivilDay(2017, 12, 23), 7000,
                      spanner::CommitTimestamp{})
          .Build()});
  if (!commit_result) throw std::move(commit_result).status();
  std::cout
      << "Update was successful [spanner_insert_data_with_timestamp_column]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public class WriteDataWithTimestampAsyncSample
{
    public class Performance
    {
        public int SingerId { get; set; }
        public int VenueId { get; set; }
        public DateTime EventDate { get; set; }
        public long Revenue { get; set; }
    }

    public async Task<int> WriteDataWithTimestampAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        List<Performance> performances = new List<Performance>
        {
            new Performance { SingerId = 1, VenueId = 4, EventDate = DateTime.Parse("2017-10-05"), Revenue = 11000 },
            new Performance { SingerId = 1, VenueId = 19, EventDate = DateTime.Parse("2017-11-02"), Revenue = 15000 },
            new Performance { SingerId = 2, VenueId = 42, EventDate = DateTime.Parse("2017-12-23"), Revenue = 7000 },
        };
        // Create connection to Cloud Spanner.
        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        // Insert rows into the Performances table.
        var rowCountAarray = await Task.WhenAll(performances.Select(performance =>
        {
            var cmd = connection.CreateInsertCommand("Performances", new SpannerParameterCollection
            {
                { "SingerId", SpannerDbType.Int64, performance.SingerId },
                { "VenueId", SpannerDbType.Int64, performance.VenueId },
                { "EventDate", SpannerDbType.Date, performance.EventDate },
                { "Revenue", SpannerDbType.Int64, performance.Revenue },
                { "LastUpdateTime", SpannerDbType.Timestamp, SpannerParameter.CommitTimestamp },
            });
            return cmd.ExecuteNonQueryAsync();
        }));
        return rowCountAarray.Sum();
    }
}
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
import (
 "context"

 "cloud.google.com/go/spanner"
)

func writeWithTimestamp(db string) error {
 ctx := context.Background()

 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 performanceColumns := []string{"SingerId", "VenueId", "EventDate", "Revenue", "LastUpdateTime"}
 m := []*spanner.Mutation{
     spanner.InsertOrUpdate("Performances", performanceColumns, []interface{}{1, 4, "2017-10-05", 11000, spanner.CommitTimestamp}),
     spanner.InsertOrUpdate("Performances", performanceColumns, []interface{}{1, 19, "2017-11-02", 15000, spanner.CommitTimestamp}),
     spanner.InsertOrUpdate("Performances", performanceColumns, []interface{}{2, 42, "2017-12-23", 7000, spanner.CommitTimestamp}),
 }
 _, err = client.Apply(ctx, m)
 return err
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static final List<Performance> PERFORMANCES =
    Arrays.asList(
        new Performance(1, 4, "2017-10-05", 11000),
        new Performance(1, 19, "2017-11-02", 15000),
        new Performance(2, 42, "2017-12-23", 7000));
static void writeExampleDataWithTimestamp(DatabaseClient dbClient) {
  List<Mutation> mutations = new ArrayList<>();
  for (Performance performance : PERFORMANCES) {
    mutations.add(
        Mutation.newInsertBuilder("Performances")
            .set("SingerId")
            .to(performance.singerId)
            .set("VenueId")
            .to(performance.venueId)
            .set("EventDate")
            .to(performance.eventDate)
            .set("Revenue")
            .to(performance.revenue)
            .set("LastUpdateTime")
            .to(Value.COMMIT_TIMESTAMP)
            .build());
  }
  dbClient.write(mutations);
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

// Instantiate Spanner table objects
const performancesTable = database.table('Performances');

const data = [
  {
    SingerId: '1',
    VenueId: '4',
    EventDate: '2017-10-05',
    Revenue: '11000',
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
  {
    SingerId: '1',
    VenueId: '19',
    EventDate: '2017-11-02',
    Revenue: '15000',
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
  {
    SingerId: '2',
    VenueId: '42',
    EventDate: '2017-12-23',
    Revenue: '7000',
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
];

// Inserts rows into the Singers table
// Note: Cloud Spanner interprets Node.js numbers as FLOAT64s, so
// they must be converted to strings before being inserted as INT64s
try {
  await performancesTable.insert(data);
  console.log('Inserted data.');
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
 * Inserts sample data into a table with a commit timestamp column.
 *
 * The database and table must already exist and can be created using
 * `create_table_with_timestamp_column`.
 * Example:
 * ```
 * insert_data_with_timestamp_column($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function insert_data_with_timestamp_column(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $operation = $database->transaction(['singleUse' => true])
        ->insertBatch('Performances', [
            ['SingerId' => 1, 'VenueId' => 4, 'EventDate' => '2017-10-05', 'Revenue' => 11000, 'LastUpdateTime' => $spanner->commitTimestamp()],
            ['SingerId' => 1, 'VenueId' => 19, 'EventDate' => '2017-11-02', 'Revenue' => 15000, 'LastUpdateTime' => $spanner->commitTimestamp()],
            ['SingerId' => 2, 'VenueId' => 42, 'EventDate' => '2017-12-23', 'Revenue' => 7000, 'LastUpdateTime' => $spanner->commitTimestamp()],
        ])
        ->commit();

    print('Inserted data.' . PHP_EOL);
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def insert_data_with_timestamp(instance_id, database_id):
    """Inserts data with a COMMIT_TIMESTAMP field into a table."""

    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)

    database = instance.database(database_id)

    with database.batch() as batch:
        batch.insert(
            table="Performances",
            columns=("SingerId", "VenueId", "EventDate", "Revenue", "LastUpdateTime"),
            values=[
                (1, 4, "2017-10-05", 11000, spanner.COMMIT_TIMESTAMP),
                (1, 19, "2017-11-02", 15000, spanner.COMMIT_TIMESTAMP),
                (2, 42, "2017-12-23", 7000, spanner.COMMIT_TIMESTAMP),
            ],
        )

    print("Inserted data.")
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

# Get commit_timestamp
commit_timestamp = client.commit_timestamp

client.commit do |c|
  c.insert "Performances", [
    { SingerId: 1, VenueId: 4, EventDate: "2017-10-05", Revenue: 11_000, LastUpdateTime: commit_timestamp },
    { SingerId: 1, VenueId: 19, EventDate: "2017-11-02", Revenue: 15_000, LastUpdateTime: commit_timestamp },
    { SingerId: 2, VenueId: 42, EventDate: "2017-12-23", Revenue: 7000, LastUpdateTime: commit_timestamp }
  ]
end

puts "Inserted data"
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
