Update data in NUMERIC column by using mutations.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Work with NUMERIC data](/spanner/docs/working-with-numerics)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void UpdateDataWithNumeric(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto insert_venues =
      spanner::InsertMutationBuilder(
          "Venues", {"VenueId", "VenueName", "Revenue", "LastUpdateTime"})
          .EmplaceRow(1, "Venue 1", spanner::MakeNumeric(35000).value(),
                      spanner::CommitTimestamp())
          .EmplaceRow(6, "Venue 6", spanner::MakeNumeric(104500).value(),
                      spanner::CommitTimestamp())
          .EmplaceRow(
              14, "Venue 14",
              spanner::MakeNumeric("99999999999999999999999999999.99").value(),
              spanner::CommitTimestamp())
          .Build();

  auto commit_result = client.Commit(spanner::Mutations{insert_venues});
  if (!commit_result) throw std::move(commit_result).status();
  std::cout
      << "Insert was successful [spanner_update_data_with_numeric_column]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using Google.Cloud.Spanner.V1;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public class UpdateDataWithNumericAsyncSample
{
    public class Venue
    {
        public int VenueId { get; set; }
        public SpannerNumeric Revenue { get; set; }
    }

    public async Task<int> UpdateDataWithNumericAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        List<Venue> venues = new List<Venue>
        {
            new Venue { VenueId = 4, Revenue = SpannerNumeric.Parse("35000") },
            new Venue { VenueId = 19, Revenue = SpannerNumeric.Parse("104500") },
            new Venue { VenueId = 42, Revenue = SpannerNumeric.Parse("99999999999999999999999999999.99") },
        };
        // Create connection to Cloud Spanner.
        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        var affectedRows = await Task.WhenAll(venues.Select(venue =>
        {
            // Update rows in the Venues table.
            using var cmd = connection.CreateUpdateCommand("Venues", new SpannerParameterCollection
            {
                { "VenueId", SpannerDbType.Int64, venue.VenueId },
                { "Revenue", SpannerDbType.Numeric, venue.Revenue }
            });
            return cmd.ExecuteNonQueryAsync();
        }));

        Console.WriteLine("Data updated.");
        return affectedRows.Sum();
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

func updateDataWithNumericColumn(w io.Writer, db string) error {
 ctx := context.Background()

 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 cols := []string{"VenueId", "Revenue"}
 _, err = client.Apply(ctx, []*spanner.Mutation{
     spanner.Update("Venues", cols, []interface{}{4, "35000"}),
     spanner.Update("Venues", cols, []interface{}{19, "104500"}),
     spanner.Update("Venues", cols, []interface{}{42, "99999999999999999999999999999.99"}),
 })
 return err
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.Mutation;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.common.collect.ImmutableList;
import java.math.BigDecimal;

class UpdateNumericDataSample {

  static void updateNumericData() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";

    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService()) {
      DatabaseClient client =
          spanner.getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));
      updateNumericData(client);
    }
  }

  static void updateNumericData(DatabaseClient client) {
    client.write(
        ImmutableList.of(
            Mutation.newInsertOrUpdateBuilder("Venues")
                .set("VenueId")
                .to(4L)
                .set("Revenue")
                .to(new BigDecimal("35000"))
                .build(),
            Mutation.newInsertOrUpdateBuilder("Venues")
                .set("VenueId")
                .to(19L)
                .set("Revenue")
                .to(new BigDecimal("104500"))
                .build(),
            Mutation.newInsertOrUpdateBuilder("Venues")
                .set("VenueId")
                .to(42L)
                .set("Revenue")
                .to(new BigDecimal("99999999999999999999999999999.99"))
                .build()));
    System.out.println("Venues successfully updated");
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

// Instantiate Spanner table objects.
const venuesTable = database.table('Venues');

const data = [
  {
    VenueId: '4',
    Revenue: Spanner.numeric('35000'),
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
  {
    VenueId: '19',
    Revenue: Spanner.numeric('104500'),
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
  {
    VenueId: '42',
    Revenue: Spanner.numeric('99999999999999999999999999999.99'),
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
];

// Updates rows in the Venues table.
try {
  await venuesTable.update(data);
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
 * Updates sample data in a table with a NUMERIC column.
 *
 * Before executing this method, a new column Revenue has to be added to the Venues
 * table by applying the DDL statement "ALTER TABLE Venues ADD COLUMN Revenue NUMERIC".
 *
 * Example:
 * ```
 * update_data_with_numeric_column($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function update_data_with_numeric_column(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $database->transaction(['singleUse' => true])
        ->updateBatch('Venues', [
            ['VenueId' => 4, 'Revenue' => $spanner->numeric('35000')],
            ['VenueId' => 19, 'Revenue' => $spanner->numeric('104500')],
            ['VenueId' => 42, 'Revenue' => $spanner->numeric('99999999999999999999999999999.99')],
        ])
        ->commit();

    print('Updated data.' . PHP_EOL);
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def update_data_with_numeric(instance_id, database_id):
    """Updates Venues tables in the database with the NUMERIC
    column.

    This updates the `Revenue` column which must be created before
    running this sample. You can add the column by running the
    `add_numeric_column` sample or by running this DDL statement
     against your database:

        ALTER TABLE Venues ADD COLUMN Revenue NUMERIC
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)

    database = instance.database(database_id)

    with database.batch() as batch:
        batch.update(
            table="Venues",
            columns=("VenueId", "Revenue"),
            values=[
                (4, decimal.Decimal("35000")),
                (19, decimal.Decimal("104500")),
                (42, decimal.Decimal("99999999999999999999999999999.99")),
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

client.commit do |c|
  c.update "Venues", [
    { VenueId: 4, Revenue: "35000" },
    { VenueId: 19, Revenue: "104500" },
    { VenueId: 42, Revenue: "99999999999999999999999999999.99" }
  ]
end

puts "Updated data"
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
