Update data using DML.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Insert, update, and delete data using data manipulation language (DML)](/spanner/docs/dml-tasks)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void DmlStandardUpdate(google::cloud::spanner::Client client) {
  //! [commit-with-mutator]
  using ::google::cloud::StatusOr;
  namespace spanner = ::google::cloud::spanner;
  auto commit_result = client.Commit(
      [&client](spanner::Transaction txn) -> StatusOr<spanner::Mutations> {
        auto update = client.ExecuteDml(
            std::move(txn),
            spanner::SqlStatement(
                "UPDATE Albums SET MarketingBudget = MarketingBudget * 2"
                "  WHERE SingerId = 1 AND AlbumId = 1"));
        if (!update) return std::move(update).status();
        return spanner::Mutations{};
      });
  if (!commit_result) throw std::move(commit_result).status();
  //! [commit-with-mutator]
  std::cout << "Update was successful [spanner_dml_standard_update]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class UpdateUsingDmlCoreAsyncSample
{
    public async Task<int> UpdateUsingDmlCoreAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        using var cmd = connection.CreateDmlCommand("UPDATE Albums SET MarketingBudget = MarketingBudget * 2 WHERE SingerId = 1 and AlbumId = 1");
        int rowCount = await cmd.ExecuteNonQueryAsync();

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

func updateUsingDML(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `UPDATE Albums
             SET MarketingBudget = MarketingBudget * 2
             WHERE SingerId = 1 and AlbumId = 1`,
     }
     rowCount, err := txn.Update(ctx, stmt)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%d record(s) updated.\n", rowCount)
     return nil
 })
 return err
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void updateUsingDml(DatabaseClient dbClient) {
  dbClient
      .readWriteTransaction()
      .run(transaction -> {
        String sql =
            "UPDATE Albums "
                + "SET MarketingBudget = MarketingBudget * 2 "
                + "WHERE SingerId = 1 and AlbumId = 1";
        long rowCount = transaction.executeUpdate(Statement.of(sql));
        System.out.printf("%d record updated.\n", rowCount);
        return null;
      });
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

database.runTransaction(async (err, transaction) => {
  if (err) {
    console.error(err);
    return;
  }
  try {
    const [rowCount] = await transaction.runUpdate({
      sql: `UPDATE Albums SET MarketingBudget = MarketingBudget * 2
        WHERE SingerId = 1 and AlbumId = 1`,
    });

    console.log(`Successfully updated ${rowCount} record.`);
    await transaction.commit();
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    // Close the database when finished.
    database.close();
  }
});
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Transaction;

/**
 * Updates sample data in the database with a DML statement.
 *
 * This requires the `MarketingBudget` column which must be created before
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
function update_data_with_dml(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $database->runTransaction(function (Transaction $t) {
        $rowCount = $t->executeUpdate(
            'UPDATE Albums '
            . 'SET MarketingBudget = MarketingBudget * 2 '
            . 'WHERE SingerId = 1 and AlbumId = 1');
        $t->commit();
        printf('Updated %d row(s).' . PHP_EOL, $rowCount);
    });
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

def update_albums(transaction):
    row_ct = transaction.execute_update(
        "UPDATE Albums "
        "SET MarketingBudget = MarketingBudget * 2 "
        "WHERE SingerId = 1 and AlbumId = 1"
    )

    print("{} record(s) updated.".format(row_ct))

database.run_in_transaction(update_albums)
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
row_count = 0

client.transaction do |transaction|
  row_count = transaction.execute_update(
    "UPDATE Albums
     SET MarketingBudget = MarketingBudget * 2
     WHERE SingerId = 1 and AlbumId = 1"
  )
end

puts "#{row_count} record updated."
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
