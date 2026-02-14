Bulk update data by using a partitioned DML statement.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Insert, update, and delete data using data manipulation language (DML)](/spanner/docs/dml-tasks)
  - [Partitioned Data Manipulation Language](/spanner/docs/dml-partitioned)
  - [Transactions overview](/spanner/docs/transactions)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void DmlPartitionedUpdate(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  auto result = client.ExecutePartitionedDml(
      spanner::SqlStatement("UPDATE Albums SET MarketingBudget = 100000"
                            "  WHERE SingerId > 1"));
  if (!result) throw std::move(result).status();
  std::cout << "Updated at least " << result->row_count_lower_bound
            << " row(s) [spanner_dml_partitioned_update]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class UpdateUsingPartitionedDmlCoreAsyncSample
{
    public async Task<long> UpdateUsingPartitionedDmlCoreAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        using var cmd = connection.CreateDmlCommand("UPDATE Albums SET MarketingBudget = 100000 WHERE SingerId > 1");
        long rowCount = await cmd.ExecutePartitionedUpdateAsync();

        Console.WriteLine($"{rowCount} row(s) updated...");
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
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
)

func updateUsingPartitionedDML(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{SQL: "UPDATE Albums SET MarketingBudget = 100000 WHERE SingerId > 1"}
 rowCount, err := client.PartitionedUpdate(ctx, stmt)
 if err != nil {
     return err
 }
 fmt.Fprintf(w, "%d record(s) updated.\n", rowCount)
 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void updateUsingPartitionedDml(DatabaseClient dbClient) {
  String sql = "UPDATE Albums SET MarketingBudget = 100000 WHERE SingerId > 1";
  long rowCount = dbClient.executePartitionedUpdate(Statement.of(sql));
  System.out.printf("%d records updated.\n", rowCount);
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

try {
  const [rowCount] = await database.runPartitionedUpdate({
    sql: 'UPDATE Albums SET MarketingBudget = 100000 WHERE SingerId > 1',
  });
  console.log(`Successfully updated ${rowCount} records.`);
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
 * Updates sample data in the database by partition with a DML statement.
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
function update_data_with_partitioned_dml(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $rowCount = $database->executePartitionedUpdate(
        'UPDATE Albums SET MarketingBudget = 100000 WHERE SingerId > 1'
    );

    printf('Updated %d row(s).' . PHP_EOL, $rowCount);
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

row_ct = database.execute_partitioned_dml(
    "UPDATE Albums SET MarketingBudget = 100000 WHERE SingerId > 1"
)

print("{} records updated.".format(row_ct))
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

row_count = client.execute_partition_update(
  "UPDATE Albums SET MarketingBudget = 100000 WHERE SingerId > 1"
)

puts "#{row_count} records updated."
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
