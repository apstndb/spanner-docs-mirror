Update a schema by adding a NUMERIC column.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Work with NUMERIC data](/spanner/docs/working-with-numerics)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void AddNumericColumn(google::cloud::spanner_admin::DatabaseAdminClient client,
                      std::string const& project_id,
                      std::string const& instance_id,
                      std::string const& database_id) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  auto metadata = client
                      .UpdateDatabaseDdl(database.FullName(), {R"""(
                        ALTER TABLE Venues ADD COLUMN Revenue NUMERIC)"""})
                      .get();
  google::cloud::spanner_testing::LogUpdateDatabaseDdl(  //! TODO(#4758)
      client, database, metadata.status());              //! TODO(#4758)
  if (!metadata) throw std::move(metadata).status();
  std::cout << "`Venues` table altered, new DDL:\n" << metadata->DebugString();
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class AddNumericColumnAsyncSample
{
    public async Task AddNumericColumnAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        string alterStatement = "ALTER TABLE Venues ADD COLUMN Revenue NUMERIC";

        using var connection = new SpannerConnection(connectionString);
        using var updateCmd = connection.CreateDdlCommand(alterStatement);
        await updateCmd.ExecuteNonQueryAsync();
        Console.WriteLine("Added the Revenue column.");
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

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

func addNumericColumn(ctx context.Context, w io.Writer, db string) error {
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         "ALTER TABLE Venues ADD COLUMN Revenue NUMERIC",
     },
 })
 if err != nil {
     return err
 }
 if err := op.Wait(ctx); err != nil {
     return err
 }
 fmt.Fprintf(w, "Added Revenue column\n")
 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.common.collect.ImmutableList;
import com.google.spanner.admin.database.v1.DatabaseName;
import java.util.concurrent.ExecutionException;

class AddNumericColumnSample {

  static void addNumericColumn() throws InterruptedException, ExecutionException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";

    addNumericColumn(projectId, instanceId, databaseId);
  }

  static void addNumericColumn(String projectId, String instanceId, String databaseId)
      throws InterruptedException, ExecutionException {
    try (Spanner spanner =
        SpannerOptions.newBuilder()
            .setProjectId(projectId)
            .build()
            .getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {
      // Wait for the operation to finish.
      // This will throw an ExecutionException if the operation fails.
      databaseAdminClient.updateDatabaseDdlAsync(
          DatabaseName.of(projectId, instanceId, databaseId),
          ImmutableList.of("ALTER TABLE Venues ADD COLUMN Revenue NUMERIC")).get();
      System.out.printf("Successfully added column `Revenue`%n");
    }
  }
}
```

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` javascript
/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

const request = ['ALTER TABLE Venues ADD COLUMN Revenue NUMERIC'];

// Alter existing table to add a column.
const [operation] = await databaseAdminClient.updateDatabaseDdl({
  database: databaseAdminClient.databasePath(
    projectId,
    instanceId,
    databaseId,
  ),
  statements: request,
});

console.log(`Waiting for operation on ${databaseId} to complete...`);

await operation.promise();

console.log(
  `Added Revenue column to Venues table in database ${databaseId}.`,
);
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;

/**
 * Adds a NUMERIC column to a table.
 * Example:
 * ```
 * add_numeric_column($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function add_numeric_column(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);

    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => ['ALTER TABLE Venues ADD COLUMN Revenue NUMERIC']
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf('Added Revenue as a NUMERIC column in Venues table' . PHP_EOL);
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def add_numeric_column(instance_id, database_id):
    """Adds a new NUMERIC column to the Venues table in the example database."""

    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        ),
        statements=["ALTER TABLE Venues ADD COLUMN Revenue NUMERIC"],
    )

    operation = database_admin_api.update_database_ddl(request)

    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)

    print(
        'Altered table "Venues" on database {} on instance {}.'.format(
            database_id, instance_id
        )
    )
```

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin

db_path = database_admin_client.database_path project: project_id,
                                              instance: instance_id,
                                              database: database_id

job = database_admin_client.update_database_ddl database: db_path,
                                                statements: [
                                                  "ALTER TABLE Venues ADD COLUMN Revenue NUMERIC"
                                                ]

puts "Waiting for database update to complete"

job.wait_until_done!

puts "Added the Revenue as a numeric column in Venues table"
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
