Create a table with example data types.

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void CreateTableWithDatatypes(
    google::cloud::spanner_admin::DatabaseAdminClient client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  auto metadata = client
                      .UpdateDatabaseDdl(database.FullName(), {R"""(
                        CREATE TABLE Venues (
                            VenueId         INT64 NOT NULL,
                            VenueName       STRING(100),
                            VenueInfo       BYTES(MAX),
                            Capacity        INT64,
                            AvailableDates  ARRAY<DATE>,
                            LastContactDate DATE,
                            OutdoorVenue    BOOL,
                            PopularityScore FLOAT64,
                            LastUpdateTime  TIMESTAMP NOT NULL OPTIONS
                                (allow_commit_timestamp=true)
                        ) PRIMARY KEY (VenueId))"""})
                      .get();
  google::cloud::spanner_testing::LogUpdateDatabaseDdl(  //! TODO(#4758)
      client, database, metadata.status());              //! TODO(#4758)
  if (!metadata) throw std::move(metadata).status();
  std::cout << "`Venues` table created, new DDL:\n" << metadata->DebugString();
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System.Threading.Tasks;

public class CreateTableWithDataTypesAsyncSample
{
    public async Task CreateTableWithDataTypesAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        using var connection = new SpannerConnection(connectionString);

        // Define create table statement for table with supported datatypes columns.
        string createTableStatement =
        @"CREATE TABLE Venues (
                    VenueId INT64 NOT NULL,
                    VenueName STRING(100),
                    VenueInfo BYTES(MAX),
                    Capacity INT64,
                    AvailableDates ARRAY<DATE>,
                    LastContactDate DATE,
                    OutdoorVenue BOOL,
                    PopularityScore FLOAT64,
                    Revenue NUMERIC,
                    LastUpdateTime TIMESTAMP NOT NULL 
                        OPTIONS (allow_commit_timestamp=true)
                ) PRIMARY KEY (VenueId)";
        var cmd = connection.CreateDdlCommand(createTableStatement);
        await cmd.ExecuteNonQueryAsync();
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

// Creates a Cloud Spanner table comprised of columns for each supported data type
// See https://cloud.google.com/spanner/docs/data-types
func createTableWithDatatypes(ctx context.Context, w io.Writer, db string) error {
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         `CREATE TABLE Venues (
             VenueId INT64 NOT NULL,
             VenueName STRING(100),
             VenueInfo BYTES(MAX),
             Capacity INT64,
             AvailableDates ARRAY<DATE>,
             LastContactDate DATE,
             OutdoorVenue BOOL,
             PopularityScore FLOAT64,
             Revenue NUMERIC,
             LastUpdateTime TIMESTAMP NOT NULL OPTIONS (allow_commit_timestamp=true)
         ) PRIMARY KEY (VenueId)`,
     },
 })
 if err != nil {
     return fmt.Errorf("UpdateDatabaseDdl: %w", err)
 }
 if err := op.Wait(ctx); err != nil {
     return err
 }
 fmt.Fprintf(w, "Created Venues table in database [%s]\n", db)
 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void createTableWithDatatypes(DatabaseAdminClient dbAdminClient, DatabaseId id) {
  OperationFuture<Void, UpdateDatabaseDdlMetadata> op =
      dbAdminClient.updateDatabaseDdl(
          id.getInstanceId().getInstance(),
          id.getDatabase(),
          Arrays.asList(
              "CREATE TABLE Venues ("
                  + "  VenueId         INT64 NOT NULL,"
                  + "  VenueName       STRING(100),"
                  + "  VenueInfo       BYTES(MAX),"
                  + "  Capacity        INT64,"
                  + "  AvailableDates  ARRAY<DATE>,"
                  + "  LastContactDate DATE,"
                  + "  OutdoorVenue    BOOL, "
                  + "  PopularityScore FLOAT64, "
                  + "  Revenue         NUMERIC, "
                  + "  VenueDetails    JSON, "
                  + "  LastUpdateTime  TIMESTAMP NOT NULL OPTIONS (allow_commit_timestamp=true)"
                  + ") PRIMARY KEY (VenueId)"),
          null);
  try {
    // Initiate the request which returns an OperationFuture.
    op.get();
    System.out.println("Created Venues table in database: [" + id + "]");
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

const request = [
  `CREATE TABLE Venues (
      VenueId                INT64 NOT NULL,
      VenueName              STRING(100),
      VenueInfo              BYTES(MAX),
      Capacity               INT64,
      AvailableDates         ARRAY<DATE>,
      LastContactDate        Date,
      OutdoorVenue           BOOL,
      PopularityScore        FLOAT64,
      LastUpdateTime TIMESTAMP NOT NULL OPTIONS (allow_commit_timestamp=true)
    ) PRIMARY KEY (VenueId)`,
];

// Creates a table in an existing database.
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

console.log(`Created table Venues in database ${databaseId}.`);
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;

/**
 * Creates a table with suported datatypes.
 * Example:
 * ```
 * create_table_with_datatypes($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function create_table_with_datatypes(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);
    $statement = 'CREATE TABLE Venues (
        VenueId                 INT64 NOT NULL,
        VenueName              STRING(100),
        VenueInfo              BYTES(MAX),
        Capacity               INT64,
        AvailableDates         ARRAY<DATE>,
        LastContactDate        DATE,
        OutdoorVenue           BOOL,
        PopularityScore        FLOAT64,
        LastUpdateTime TIMESTAMP NOT NULL OPTIONS (allow_commit_timestamp=true)
    ) PRIMARY KEY (VenueId)';
    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => [$statement]
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf('Created Venues table in database %s on instance %s' . PHP_EOL,
        $databaseId, $instanceId);
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"

from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

spanner_client = spanner.Client()
database_admin_api = spanner_client.database_admin_api

request = spanner_database_admin.UpdateDatabaseDdlRequest(
    database=database_admin_api.database_path(
        spanner_client.project, instance_id, database_id
    ),
    statements=[
        """CREATE TABLE Venues (
        VenueId         INT64 NOT NULL,
        VenueName       STRING(100),
        VenueInfo       BYTES(MAX),
        Capacity        INT64,
        AvailableDates  ARRAY<DATE>,
        LastContactDate DATE,
        OutdoorVenue    BOOL,
        PopularityScore FLOAT64,
        LastUpdateTime  TIMESTAMP NOT NULL
        OPTIONS(allow_commit_timestamp=true)
    ) PRIMARY KEY (VenueId)"""
    ],
)
operation = database_admin_api.update_database_ddl(request)

print("Waiting for operation to complete...")
operation.result(OPERATION_TIMEOUT_SECONDS)

print(
    "Created Venues table on database {} on instance {}".format(
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

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.database instance_id, database_id

job = client.update statements: [
  "CREATE TABLE Venues (
    VenueId         INT64 NOT NULL,
    VenueName       STRING(100),
    VenueInfo       BYTES(MAX),
    Capacity        INT64,
    AvailableDates  ARRAY<DATE>,
    LastContactDate DATE,
    OutdoorVenue    BOOL,
    PopularityScore FLOAT64,
    LastUpdateTime  TIMESTAMP NOT NULL OPTIONS (allow_commit_timestamp=true)
   ) PRIMARY KEY (VenueId)"
]

puts "Waiting for update database operation to complete"

job.wait_until_done!

puts "Created table Venues in #{database_id}"
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
