Create a database.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Create and manage databases](/spanner/docs/create-manage-databases)
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
void CreateDatabase(google::cloud::spanner_admin::DatabaseAdminClient client,
                    std::string const& project_id,
                    std::string const& instance_id,
                    std::string const& database_id) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  google::spanner::admin::database::v1::CreateDatabaseRequest request;
  request.set_parent(database.instance().FullName());
  request.set_create_statement("CREATE DATABASE `" + database.database_id() +
                               "`");
  request.add_extra_statements(R"""(
      CREATE TABLE Singers (
          SingerId   INT64 NOT NULL,
          FirstName  STRING(1024),
          LastName   STRING(1024),
          SingerInfo BYTES(MAX),
          FullName   STRING(2049)
              AS (ARRAY_TO_STRING([FirstName, LastName], " ")) STORED
      ) PRIMARY KEY (SingerId))""");
  request.add_extra_statements(R"""(
      CREATE TABLE Albums (
          SingerId     INT64 NOT NULL,
          AlbumId      INT64 NOT NULL,
          AlbumTitle   STRING(MAX)
      ) PRIMARY KEY (SingerId, AlbumId),
          INTERLEAVE IN PARENT Singers ON DELETE CASCADE)""");
  auto db = client.CreateDatabase(request).get();
  if (!db) throw std::move(db).status();
  std::cout << "Database " << db->name() << " created.\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System.Threading.Tasks;

public class CreateDatabaseAsyncSample
{
    public async Task CreateDatabaseAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}";

        using var connection = new SpannerConnection(connectionString);
        var createDatabase = $"CREATE DATABASE `{databaseId}`";
        // Define create table statement for table #1.
        var createSingersTable =
            @"CREATE TABLE Singers (
                SingerId INT64 NOT NULL,
                FirstName STRING(1024),
                LastName STRING(1024),
                ComposerInfo BYTES(MAX),
                FullName STRING(2048) AS (ARRAY_TO_STRING([FirstName, LastName], "" "")) STORED
            ) PRIMARY KEY (SingerId)";
        // Define create table statement for table #2.
        var createAlbumsTable =
            @"CREATE TABLE Albums (
                SingerId INT64 NOT NULL,
                AlbumId INT64 NOT NULL,
                AlbumTitle STRING(MAX)
            ) PRIMARY KEY (SingerId, AlbumId),
            INTERLEAVE IN PARENT Singers ON DELETE CASCADE";

        using var createDbCommand = connection.CreateDdlCommand(createDatabase, createSingersTable, createAlbumsTable);
        await createDbCommand.ExecuteNonQueryAsync();
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
 "regexp"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

func createDatabase(ctx context.Context, w io.Writer, db string) error {
 matches := regexp.MustCompile("^(.*)/databases/(.*)$").FindStringSubmatch(db)
 if matches == nil || len(matches) != 3 {
     return fmt.Errorf("Invalid database id %s", db)
 }

 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 op, err := adminClient.CreateDatabase(ctx, &adminpb.CreateDatabaseRequest{
     Parent:          matches[1],
     CreateStatement: "CREATE DATABASE `" + matches[2] + "`",
     ExtraStatements: []string{
         `CREATE TABLE Singers (
             SingerId   INT64 NOT NULL,
             FirstName  STRING(1024),
             LastName   STRING(1024),
             SingerInfo BYTES(MAX),
             FullName   STRING(2048) AS (
                 ARRAY_TO_STRING([FirstName, LastName], " ")
             ) STORED
         ) PRIMARY KEY (SingerId)`,
         `CREATE TABLE Albums (
             SingerId     INT64 NOT NULL,
             AlbumId      INT64 NOT NULL,
             AlbumTitle   STRING(MAX)
         ) PRIMARY KEY (SingerId, AlbumId),
         INTERLEAVE IN PARENT Singers ON DELETE CASCADE`,
     },
 })
 if err != nil {
     return err
 }
 if _, err := op.Wait(ctx); err != nil {
     return err
 }
 fmt.Fprintf(w, "Created database [%s]\n", db)
 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void createDatabase(DatabaseAdminClient dbAdminClient,
    InstanceName instanceName, String databaseId) {
  CreateDatabaseRequest createDatabaseRequest =
      CreateDatabaseRequest.newBuilder()
          .setCreateStatement("CREATE DATABASE `" + databaseId + "`")
          .setParent(instanceName.toString())
          .addAllExtraStatements(Arrays.asList(
              "CREATE TABLE Singers ("
                  + "  SingerId   INT64 NOT NULL,"
                  + "  FirstName  STRING(1024),"
                  + "  LastName   STRING(1024),"
                  + "  SingerInfo BYTES(MAX),"
                  + "  FullName STRING(2048) AS "
                  + "  (ARRAY_TO_STRING([FirstName, LastName], \" \")) STORED"
                  + ") PRIMARY KEY (SingerId)",
              "CREATE TABLE Albums ("
                  + "  SingerId     INT64 NOT NULL,"
                  + "  AlbumId      INT64 NOT NULL,"
                  + "  AlbumTitle   STRING(MAX)"
                  + ") PRIMARY KEY (SingerId, AlbumId),"
                  + "  INTERLEAVE IN PARENT Singers ON DELETE CASCADE")).build();
  try {
    // Initiate the request which returns an OperationFuture.
    com.google.spanner.admin.database.v1.Database db =
        dbAdminClient.createDatabaseAsync(createDatabaseRequest).get();
    System.out.println("Created database [" + db.getName() + "]");
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
  projectId: projectID,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

const createSingersTableStatement = `
CREATE TABLE Singers (
  SingerId    INT64 NOT NULL,
  FirstName   STRING(1024),
  LastName    STRING(1024),
  SingerInfo  BYTES(MAX),
  FullName    STRING(2048) AS (ARRAY_TO_STRING([FirstName, LastName], " ")) STORED,
) PRIMARY KEY (SingerId)`;
const createAlbumsTableStatement = `
CREATE TABLE Albums (
  SingerId    INT64 NOT NULL,
  AlbumId     INT64 NOT NULL,
  AlbumTitle  STRING(MAX)
) PRIMARY KEY (SingerId, AlbumId),
INTERLEAVE IN PARENT Singers ON DELETE CASCADE`;

// Creates a new database
try {
  const [operation] = await databaseAdminClient.createDatabase({
    createStatement: 'CREATE DATABASE `' + databaseID + '`',
    extraStatements: [
      createSingersTableStatement,
      createAlbumsTableStatement,
    ],
    parent: databaseAdminClient.instancePath(projectID, instanceID),
  });

  console.log(`Waiting for creation of ${databaseID} to complete...`);
  await operation.promise();

  console.log(`Created database ${databaseID} on instance ${instanceID}.`);
} catch (err) {
  console.error('ERROR:', err);
}
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\CreateDatabaseRequest;

/**
 * Creates a database and tables for sample data.
 * Example:
 * ```
 * create_database($instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function create_database(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $instance = $databaseAdminClient->instanceName($projectId, $instanceId);

    $operation = $databaseAdminClient->createDatabase(
        new CreateDatabaseRequest([
            'parent' => $instance,
            'create_statement' => sprintf('CREATE DATABASE `%s`', $databaseId),
            'extra_statements' => [
                'CREATE TABLE Singers (' .
                'SingerId     INT64 NOT NULL,' .
                'FirstName    STRING(1024),' .
                'LastName     STRING(1024),' .
                'SingerInfo   BYTES(MAX),' .
                'FullName     STRING(2048) AS' .
                '(ARRAY_TO_STRING([FirstName, LastName], " ")) STORED' .
                ') PRIMARY KEY (SingerId)',
                'CREATE TABLE Albums (' .
                    'SingerId     INT64 NOT NULL,' .
                    'AlbumId      INT64 NOT NULL,' .
                    'AlbumTitle   STRING(MAX)' .
                ') PRIMARY KEY (SingerId, AlbumId),' .
                'INTERLEAVE IN PARENT Singers ON DELETE CASCADE'
            ]
        ])
    );

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf('Created database %s on instance %s' . PHP_EOL,
        $databaseId, $instanceId);
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def create_database(instance_id, database_id):
    """Creates a database and tables for sample data."""
    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.CreateDatabaseRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        create_statement=f"CREATE DATABASE `{database_id}`",
        extra_statements=[
            """CREATE TABLE Singers (
            SingerId     INT64 NOT NULL,
            FirstName    STRING(1024),
            LastName     STRING(1024),
            SingerInfo   BYTES(MAX),
            FullName   STRING(2048) AS (
                ARRAY_TO_STRING([FirstName, LastName], " ")
            ) STORED
        ) PRIMARY KEY (SingerId)""",
            """CREATE TABLE Albums (
            SingerId     INT64 NOT NULL,
            AlbumId      INT64 NOT NULL,
            AlbumTitle   STRING(MAX)
        ) PRIMARY KEY (SingerId, AlbumId),
        INTERLEAVE IN PARENT Singers ON DELETE CASCADE""",
        ],
    )

    operation = database_admin_api.create_database(request=request)

    print("Waiting for operation to complete...")
    database = operation.result(OPERATION_TIMEOUT_SECONDS)

    print(
        "Created database {} on instance {}".format(
            database.name,
            database_admin_api.instance_path(spanner_client.project, instance_id),
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

instance_path = database_admin_client.instance_path project: project_id, instance: instance_id

job = database_admin_client.create_database parent: instance_path,
                                            create_statement: "CREATE DATABASE `#{database_id}`",
                                            extra_statements: [
                                              "CREATE TABLE Singers (
      SingerId     INT64 NOT NULL,
      FirstName    STRING(1024),
      LastName     STRING(1024),
      SingerInfo   BYTES(MAX)
    ) PRIMARY KEY (SingerId)",

                                              "CREATE TABLE Albums (
      SingerId     INT64 NOT NULL,
      AlbumId      INT64 NOT NULL,
      AlbumTitle   STRING(MAX)
    ) PRIMARY KEY (SingerId, AlbumId),
    INTERLEAVE IN PARENT Singers ON DELETE CASCADE"
                                            ]

puts "Waiting for create database operation to complete"

job.wait_until_done!

puts "Created database #{database_id} on instance #{instance_id}"
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
