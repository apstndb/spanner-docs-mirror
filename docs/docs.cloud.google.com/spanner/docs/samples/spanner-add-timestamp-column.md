Update a schema by adding a TIMESTAMP column.

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void AddTimestampColumn(
    google::cloud::spanner_admin::DatabaseAdminClient client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  auto metadata =
      client
          .UpdateDatabaseDdl(
              database.FullName(),
              {"ALTER TABLE Albums ADD COLUMN LastUpdateTime TIMESTAMP "
               "OPTIONS (allow_commit_timestamp=true)"})
          .get();
  google::cloud::spanner_testing::LogUpdateDatabaseDdl(  //! TODO(#4758)
      client, database, metadata.status());              //! TODO(#4758)
  if (!metadata) throw std::move(metadata).status();
  std::cout << "Added LastUpdateTime column\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class AddCommitTimestampAsyncSample
{
    public async Task AddCommitTimestampAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        string alterStatement = "ALTER TABLE Albums ADD COLUMN LastUpdateTime TIMESTAMP OPTIONS (allow_commit_timestamp=true)";
        using var connection = new SpannerConnection(connectionString);
        var updateCmd = connection.CreateDdlCommand(alterStatement);
        await updateCmd.ExecuteNonQueryAsync();
        Console.WriteLine("Added LastUpdateTime as a commit timestamp column in Albums table.");
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

func addCommitTimestamp(ctx context.Context, w io.Writer, db string) error {
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         "ALTER TABLE Albums ADD COLUMN LastUpdateTime TIMESTAMP " +
             "OPTIONS (allow_commit_timestamp=true)",
     },
 })
 if err != nil {
     return err
 }
 if err := op.Wait(ctx); err != nil {
     return err
 }
 fmt.Fprintf(w, "Added LastUpdateTime as a commit timestamp column in Albums table\n")
 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void addCommitTimestamp(DatabaseAdminClient adminClient, DatabaseId dbId) {
  OperationFuture<Void, UpdateDatabaseDdlMetadata> op =
      adminClient.updateDatabaseDdl(
          dbId.getInstanceId().getInstance(),
          dbId.getDatabase(),
          Arrays.asList(
              "ALTER TABLE Albums ADD COLUMN LastUpdateTime TIMESTAMP "
                  + "OPTIONS (allow_commit_timestamp=true)"),
          null);
  try {
    // Initiate the request which returns an OperationFuture.
    op.get();
    System.out.println("Added LastUpdateTime as a commit timestamp column in Albums table.");
  } catch (ExecutionException e) {
    // If the operation failed during execution, expose the cause.
    throw (SpannerException) e.getCause();
  } catch (InterruptedException e) {
    // Throw when a thread is waiting, sleeping, or otherwise occupied,
    // and the thread is interrupted, either before or during the activity.
    throw SpannerExceptionFactory.propagateInterrupt(e);
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

// Gets a reference to a Cloud Spanner Database Admin Client object
const databaseAdminClient = spanner.getDatabaseAdminClient();

const request = [
  `ALTER TABLE Albums ADD COLUMN LastUpdateTime TIMESTAMP OPTIONS
  (allow_commit_timestamp=true)`,
];

// Adds a new commit timestamp column to the Albums table
try {
  const [operation] = await databaseAdminClient.updateDatabaseDdl({
    database: databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    ),
    statements: request,
  });

  console.log('Waiting for operation to complete...');

  await operation.promise();

  console.log(
    'Added LastUpdateTime as a commit timestamp column in Albums table.',
  );
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the spanner client when finished.
  // The databaseAdminClient does not require explicit closure. The closure of the Spanner client will automatically close the databaseAdminClient.
  spanner.close();
}
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Adds a commit timestamp column to a table.
 * Example:
 * ```
 * add_timestamp_column($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function add_timestamp_column(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $operation = $database->updateDdl(
        'ALTER TABLE Albums ADD COLUMN LastUpdateTime TIMESTAMP OPTIONS (allow_commit_timestamp=true)'
    );

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf('Added LastUpdateTime as a commit timestamp column in Albums table' . PHP_EOL);
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def add_timestamp_column(instance_id, database_id):
    """Adds a new TIMESTAMP column to the Albums table in the example database."""

    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        ),
        statements=[
            "ALTER TABLE Albums ADD COLUMN LastUpdateTime TIMESTAMP "
            "OPTIONS(allow_commit_timestamp=true)"
        ],
    )

    operation = database_admin_api.update_database_ddl(request)

    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)

    print(
        'Altered table "Albums" on database {} on instance {}.'.format(
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
                                                  "ALTER TABLE Albums ADD COLUMN LastUpdateTime TIMESTAMP
   OPTIONS (allow_commit_timestamp=true)"
                                                ]

puts "Waiting for database update to complete"

job.wait_until_done!

puts "Added the LastUpdateTime as a commit timestamp column in Albums table"
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
