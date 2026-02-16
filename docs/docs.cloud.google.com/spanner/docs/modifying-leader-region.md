This page explains how to modify the [leader region](/spanner/docs/instance-configurations#replication_2) of a database. You can only change the leader region of a Spanner instance that uses a dual-region or multi-region instance configuration. The new leader region must be one of the two read-write regions within the [dual-region](/spanner/docs/instance-configurations#available-configurations-dual) or [multi-region](/spanner/docs/instance-configurations#available-configurations-multi-region) configuration of your database. For more information about changing the leader region, see [Configure the default leader region](/spanner/docs/instance-configurations#configure-leader-region) .

To view data on the database leader distribution of a dual-region or multi-region instance, open the Google Cloud console, and refer to the **Leader distribution** charts. For more information, see [Spanner charts and metrics](/spanner/docs/monitoring-console#charts-metrics) .

## Change the leader region of a database

You can change the leader region of a database. To monitor the progress of this change, you can refer to the [Leader distribution chart](/spanner/docs/monitoring-console#charts-metrics) .

### Console

1.  Go to the **Spanner Instances** page in the Google Cloud console.

2.  Click the name of the instance that contains the database whose leader region you want to change.

3.  Click the database whose leader region you want to change.

4.  Click the pencil icon next to **Leader Region** .

5.  Modify the DDL statement with your selected leader region as `  default_leader  ` .

### gcloud

1.  To change the leader region of an existing database, run the following command:
    
    ``` text
    gcloud spanner databases ddl update DATABASE_NAME \
    --instance=INSTANCE_ID \
    --ddl='ALTER DATABASE `DATABASE_NAME` \
     SET OPTIONS ( default_leader = "REGION" )'
    ```
    
    Replace the following:
    
      - `  DATABASE_NAME  ` : the name of your database.
      - `  INSTANCE_ID  ` : the identifier of your database instance.
      - `  REGION  ` : the region you want to set as the default leader.

### Client libraries

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using Google.LongRunning;
using Google.Protobuf.WellKnownTypes;
using System;
using System.Threading.Tasks;

public class UpdateDatabaseWithDefaultLeaderAsyncSample
{
    public async Task<Operation<Empty, UpdateDatabaseDdlMetadata>> UpdateDatabaseWithDefaultLeaderAsync(string projectId, string instanceId, string databaseId, string defaultLeader)
    {
        DatabaseAdminClient databaseAdminClient = await DatabaseAdminClient.CreateAsync();

        var alterDatabaseStatement = @$"ALTER DATABASE `{databaseId}` SET OPTIONS
                   (default_leader = '{defaultLeader}')";

        // Create the UpdateDatabaseDdl request and execute it.
        var request = new UpdateDatabaseDdlRequest
        {
            DatabaseAsDatabaseName = DatabaseName.FromProjectInstanceDatabase(projectId, instanceId, databaseId),
            Statements = { alterDatabaseStatement }
        };
        var operation = await databaseAdminClient.UpdateDatabaseDdlAsync(request);

        // Wait until the operation has finished.
        Console.WriteLine("Waiting for the operation to finish.");
        Operation<Empty, UpdateDatabaseDdlMetadata> completedResponse = await operation.PollUntilCompletedAsync();
        if (completedResponse.IsFaulted)
        {
            Console.WriteLine($"Error while updating database: {completedResponse.Exception}");
            throw completedResponse.Exception;
        }

        Console.WriteLine("Updated default leader");
        return completedResponse;
    }
}
```

### C++

``` cpp
void UpdateDatabaseWithDefaultLeader(
    google::cloud::spanner_admin::DatabaseAdminClient client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id, std::string const& default_leader) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  std::vector<std::string> statements;
  statements.push_back("ALTER DATABASE `" + database_id + "` " +
                       "SET OPTIONS (default_leader='" + default_leader + "')");
  auto metadata =
      client.UpdateDatabaseDdl(database.FullName(), std::move(statements))
          .get();
  if (!metadata) throw std::move(metadata).status();
  std::cout << "`default_leader` altered, new DDL metadata:\n"
            << metadata->DebugString();
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"
 "regexp"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

// updateDatabaseWithDefaultLeader updates the default leader for a given database
func updateDatabaseWithDefaultLeader(w io.Writer, db string, defaultLeader string) error {
 // db = `projects/<project>/instances/<instance-id>/database/<database-id>`
 // defaultLeader = `nam3`
 matches := regexp.MustCompile("^(.+)/databases/(.+)$").FindStringSubmatch(db)
 if matches == nil || len(matches) != 3 {
     return fmt.Errorf("updateDatabaseWithDefaultLeader: invalid database id %q", db)
 }

 ctx := context.Background()
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         fmt.Sprintf(
             "ALTER DATABASE `%s` SET OPTIONS (default_leader = '%s')",
             matches[2], defaultLeader,
         ),
     },
 })
 if err != nil {
     return err
 }
 if err := op.Wait(ctx); err != nil {
     return err
 }
 fmt.Fprintf(w, "Updated the default leader\n")
 return nil
}
```

### Java

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerException;
import com.google.cloud.spanner.SpannerExceptionFactory;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.spanner.admin.database.v1.DatabaseName;
import java.util.Collections;
import java.util.concurrent.ExecutionException;

public class UpdateDatabaseWithDefaultLeaderSample {

  static void updateDatabaseWithDefaultLeader() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceId = "my-instance";
    final String databaseId = "my-database";
    final String defaultLeader = "my-default-leader";
    updateDatabaseWithDefaultLeader(projectId, instanceId, databaseId, defaultLeader);
  }

  static void updateDatabaseWithDefaultLeader(
      String projectId, String instanceId, String databaseId, String defaultLeader) {
    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {
      databaseAdminClient
          .updateDatabaseDdlAsync(
              DatabaseName.of(projectId, instanceId, databaseId),
              Collections.singletonList(
                  String.format(
                      "ALTER DATABASE `%s` SET OPTIONS (default_leader = '%s')",
                      databaseId,
                      defaultLeader
                  )
              )
          ).get();
      System.out.println("Updated default leader to " + defaultLeader);
    } catch (ExecutionException e) {
      // If the operation failed during execution, expose the cause.
      throw (SpannerException) e.getCause();
    } catch (InterruptedException e) {
      // Throw when a thread is waiting, sleeping, or otherwise occupied,
      // and the thread is interrupted, either before or during the activity.
      throw SpannerExceptionFactory.propagateInterrupt(e);
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance-id';
// const databaseId = 'my-database-id';
// const defaultLeader = 'my-default-leader';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

async function updateDatabaseWithDefaultLeader() {
  console.log(
    `Updating database ${databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    )}.`,
  );
  const setDefaultLeaderStatement = `
    ALTER DATABASE \`${databaseId}\`
    SET OPTIONS (default_leader = '${defaultLeader}')`;
  const [operation] = await databaseAdminClient.updateDatabaseDdl({
    database: databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    ),
    statements: [setDefaultLeaderStatement],
  });

  console.log(`Waiting for updating of ${databaseId} to complete...`);
  await operation.promise();
  console.log(
    `Updated database ${databaseId} with default leader ${defaultLeader}.`,
  );
}
updateDatabaseWithDefaultLeader();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\GetDatabaseRequest;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;

/**
 * Updates the default leader of the database.
 * Example:
 * ```
 * update_database_with_default_leader($projectId, $instanceId, $databaseId, $defaultLeader);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 * @param string $defaultLeader The leader instance configuration used by default.
 */
function update_database_with_default_leader(
    string $projectId,
    string $instanceId,
    string $databaseId,
    string $defaultLeader
): void {
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);
    $statement = "ALTER DATABASE `$databaseId` SET OPTIONS (default_leader = '$defaultLeader')";
    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => [$statement]
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    $database = $databaseAdminClient->getDatabase(
        new GetDatabaseRequest(['name' => $databaseName])
    );

    printf('Updated the default leader to %s' . PHP_EOL, $database->getDefaultLeader());
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def update_database_with_default_leader(instance_id, database_id, default_leader):
    """Updates a database with tables with a default leader."""
    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        ),
        statements=[
            "ALTER DATABASE {}"
            " SET OPTIONS (default_leader = '{}')".format(database_id, default_leader)
        ],
    )
    operation = database_admin_api.update_database_ddl(request)

    operation.result(OPERATION_TIMEOUT_SECONDS)

    print(
        "Database {} updated with default leader {}".format(database_id, default_leader)
    )
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"
# default_leader = "Spanner database default leader"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

db_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin project_id: project_id

db_path = db_admin_client.database_path project: project_id,
                                        instance: instance_id,
                                        database: database_id
statements = [
  "ALTER DATABASE `#{database_id}` SET OPTIONS (
    default_leader = '#{default_leader}'
  )"
]

job = db_admin_client.update_database_ddl database: db_path,
                                          statements: statements

job.wait_until_done!
database = job.results

puts "Updated default leader"
```

## Reset a database leader region to the system default

### Console

1.  Go to the **Spanner Instances** page in the Google Cloud console.

2.  Click the name of the instance that contains the database whose leader region you want to change.

3.  Click the database whose leader region you want to change.

4.  Click the pencil icon next to **Leader Region** .

5.  Modify the DDL statement with your selected leader region and set `  default_leader  ` to `  NULL  ` .

### gcloud

To reset the leader region of an existing database to the default leader region, run the following command:

### GoogleSQL

``` text
gcloud spanner databases ddl update DATABASE_NAME \
--instance=INSTANCE_ID \
--ddl='ALTER DATABASE `DATABASE_NAME` \
  SET OPTIONS ( default_leader = NULL )'
```

Replace the following:

  - `  DATABASE_NAME  ` : the name of your database.
  - `  INSTANCE_ID  ` : the identifier of your database instance.

### PostgreSQL

``` text
gcloud spanner databases ddl update DATABASE_NAME \
--instance=INSTANCE_ID \
--ddl='ALTER DATABASE DATABASE_NAME \
  RESET spanner.default_leader'
```

Replace the following:

  - `  DATABASE_NAME  ` : the name of your database.
  - `  INSTANCE_ID  ` : the identifier of your database instance.

## Set a leader region when creating a database

### gcloud

To set the default leader region when creating a database, run the following command:

``` text
gcloud spanner databases create DATABASE_NAME \
--instance=INSTANCE_ID \
--ddl='CREATE TABLE TABLE_NAME (a INT64, b INT64) PRIMARY KEY(a); \
  ALTER DATABASE `DATABASE_NAME` \
  SET OPTIONS (default_leader = "REGION")'
```

Replace the following:

  - `  DATABASE_NAME  ` : the name of your database.
  - `  INSTANCE_ID  ` : the identifier of your database instance.
  - `  TABLE_NAME  ` : the name of your database table.
  - `  REGION  ` : the region you want to set as the default leader.

### Client libraries

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using System;
using System.Threading.Tasks;

public class CreateDatabaseWithDefaultLeaderAsyncSample
{
    public async Task<Database> CreateDatabaseWithDefaultLeaderAsync(string projectId, string instanceId, string databaseId, string defaultLeader)
    {
        DatabaseAdminClient databaseAdminClient = await DatabaseAdminClient.CreateAsync();
        // Define create table statement for table #1.
        var createSingersTable =
            @"CREATE TABLE Singers (
                SingerId INT64 NOT NULL,
                FirstName STRING(1024),
                LastName STRING(1024),
                ComposerInfo BYTES(MAX)
            ) PRIMARY KEY (SingerId)";
        // Define create table statement for table #2.
        var createAlbumsTable =
            @"CREATE TABLE Albums (
                SingerId INT64 NOT NULL,
                AlbumId INT64 NOT NULL,
                AlbumTitle STRING(MAX)
            ) PRIMARY KEY (SingerId, AlbumId),
            INTERLEAVE IN PARENT Singers ON DELETE CASCADE";

        // Define alter database statement to set default leader.
        var alterDatabaseStatement = @$"ALTER DATABASE `{databaseId}` SET OPTIONS (default_leader = '{defaultLeader}')";

        // Create the CreateDatabase request with default leader and execute it.
        var request = new CreateDatabaseRequest
        {
            ParentAsInstanceName = InstanceName.FromProjectInstance(projectId, instanceId),
            CreateStatement = $"CREATE DATABASE `{databaseId}`",
            ExtraStatements = { createSingersTable, createAlbumsTable, alterDatabaseStatement },
        };
        var operation = await databaseAdminClient.CreateDatabaseAsync(request);

        // Wait until the operation has finished.
        Console.WriteLine("Waiting for the operation to finish.");
        var completedResponse = await operation.PollUntilCompletedAsync();
        if (completedResponse.IsFaulted)
        {
            Console.WriteLine($"Error while creating database: {completedResponse.Exception}");
            throw completedResponse.Exception;
        }

        var database = completedResponse.Result;
        Console.WriteLine($"Created database [{databaseId}]");
        Console.WriteLine($"\t Default leader: {database.DefaultLeader}");
        return database;
    }
}
```

### C++

``` cpp
void CreateDatabaseWithDefaultLeader(
    google::cloud::spanner_admin::DatabaseAdminClient client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id, std::string const& default_leader) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  google::spanner::admin::database::v1::CreateDatabaseRequest request;
  request.set_parent(database.instance().FullName());
  request.set_create_statement("CREATE DATABASE `" + database.database_id() +
                               "`");
  request.add_extra_statements("ALTER DATABASE `" + database_id + "` " +
                               "SET OPTIONS (default_leader='" +
                               default_leader + "')");
  auto db = client.CreateDatabase(request).get();
  if (!db) throw std::move(db).status();
  std::cout << "Database " << db->name() << " created.\n" << db->DebugString();
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"
 "regexp"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

// createDatabaseWithDefaultLeader creates a database with a default leader
func createDatabaseWithDefaultLeader(w io.Writer, db string, defaultLeader string) error {
 // db = `projects/<project>/instances/<instance-id>/database/<database-id>`
 // defaultLeader = `my-default-leader`
 matches := regexp.MustCompile("^(.*)/databases/(.*)$").FindStringSubmatch(db)
 if matches == nil || len(matches) != 3 {
     return fmt.Errorf("createDatabaseWithDefaultLeader: invalid database id %s", db)
 }

 ctx := context.Background()
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 alterDatabase := fmt.Sprintf(
     "ALTER DATABASE `%s` SET OPTIONS (default_leader = '%s')",
     matches[2], defaultLeader,
 )

 req := adminpb.CreateDatabaseRequest{
     Parent:          matches[1],
     CreateStatement: "CREATE DATABASE `" + matches[2] + "`",
     ExtraStatements: []string{
         `CREATE TABLE Singers (
             SingerId   INT64 NOT NULL,
             FirstName  STRING(1024),
             LastName   STRING(1024),
             SingerInfo BYTES(MAX)
         ) PRIMARY KEY (SingerId)`,
         `CREATE TABLE Albums (
             SingerId     INT64 NOT NULL,
             AlbumId      INT64 NOT NULL,
             AlbumTitle   STRING(MAX)
         ) PRIMARY KEY (SingerId, AlbumId),
         INTERLEAVE IN PARENT Singers ON DELETE CASCADE`,
         alterDatabase,
     },
 }
 op, err := adminClient.CreateDatabase(ctx, &req)
 if err != nil {
     return fmt.Errorf("createDatabaseWithDefaultLeader.CreateDatabase: %w", err)
 }
 dbObj, err := op.Wait(ctx)
 if err != nil {
     return fmt.Errorf("createDatabaseWithDefaultLeader.Wait: %w", err)
 }
 fmt.Fprintf(w, "Created database [%s] with default leader%q\n", dbObj.Name, dbObj.DefaultLeader)
 return nil

}
```

### Java

``` java
import com.google.cloud.spanner.SpannerException;
import com.google.cloud.spanner.SpannerExceptionFactory;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.common.collect.ImmutableList;
import com.google.spanner.admin.database.v1.CreateDatabaseRequest;
import com.google.spanner.admin.database.v1.Database;
import java.io.IOException;
import java.util.concurrent.ExecutionException;

public class CreateDatabaseWithDefaultLeaderSample {

  static void createDatabaseWithDefaultLeader() throws IOException {
    // TODO(developer): Replace these variables before running the sample.
    final String instanceName = "projects/my-project/instances/my-instance-id";
    final String databaseId = "my-database-name";
    final String defaultLeader = "my-default-leader";
    createDatabaseWithDefaultLeader(instanceName, databaseId, defaultLeader);
  }

  static void createDatabaseWithDefaultLeader(String instanceName, String databaseId,
      String defaultLeader) throws IOException {
    try (DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.create()) {
      Database createdDatabase =
          databaseAdminClient.createDatabaseAsync(
              CreateDatabaseRequest.newBuilder()
                  .setParent(instanceName)
                  .setCreateStatement("CREATE DATABASE `" + databaseId + "`")
                  .addAllExtraStatements(
                      ImmutableList.of("CREATE TABLE Singers ("
                              + "  SingerId   INT64 NOT NULL,"
                              + "  FirstName  STRING(1024),"
                              + "  LastName   STRING(1024),"
                              + "  SingerInfo BYTES(MAX)"
                              + ") PRIMARY KEY (SingerId)",
                          "CREATE TABLE Albums ("
                              + "  SingerId     INT64 NOT NULL,"
                              + "  AlbumId      INT64 NOT NULL,"
                              + "  AlbumTitle   STRING(MAX)"
                              + ") PRIMARY KEY (SingerId, AlbumId),"
                              + "  INTERLEAVE IN PARENT Singers ON DELETE CASCADE",
                          "ALTER DATABASE " + "`" + databaseId + "`"
                              + " SET OPTIONS ( default_leader = '" + defaultLeader + "' )"))
                  .build()).get();
      System.out.println("Created database [" + createdDatabase.getName() + "]");
      System.out.println("\tDefault leader: " + createdDatabase.getDefaultLeader());
    } catch (ExecutionException e) {
      // If the operation failed during execution, expose the cause.
      throw (SpannerException) e.getCause();
    } catch (InterruptedException e) {
      // Throw when a thread is waiting, sleeping, or otherwise occupied,
      // and the thread is interrupted, either before or during the activity.
      throw SpannerExceptionFactory.propagateInterrupt(e);
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance-id';
// const databaseId = 'my-database-id';
// const defaultLeader = 'my-default-leader'; example: 'asia-northeast1'

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner Database Admin Client object
const databaseAdminClient = spanner.getDatabaseAdminClient();

async function createDatabaseWithDefaultLeader() {
  // Create a new database with an extra statement which will alter the
  // database after creation to set the default leader.
  console.log(
    `Creating database ${databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    )}.`,
  );
  const createSingersTableStatement = `
    CREATE TABLE Singers (
      SingerId   INT64 NOT NULL,
      FirstName  STRING(1024),
      LastName   STRING(1024),
      SingerInfo BYTES(MAX)
    ) PRIMARY KEY (SingerId)`;
  const createAlbumsStatement = `
    CREATE TABLE Albums (
      SingerId     INT64 NOT NULL,
      AlbumId      INT64 NOT NULL,
      AlbumTitle   STRING(MAX)
    ) PRIMARY KEY (SingerId, AlbumId),
      INTERLEAVE IN PARENT Singers ON DELETE CASCADE`;

  // Default leader is one of the possible values in the leaderOptions field of the
  // instance config of the instance where the database is created.
  const setDefaultLeaderStatement = `
    ALTER DATABASE \`${databaseId}\`
    SET OPTIONS (default_leader = '${defaultLeader}')`;

  const [operation] = await databaseAdminClient.createDatabase({
    createStatement: 'CREATE DATABASE `' + databaseId + '`',
    extraStatements: [
      createSingersTableStatement,
      createAlbumsStatement,
      setDefaultLeaderStatement,
    ],
    parent: databaseAdminClient.instancePath(projectId, instanceId),
  });

  console.log(`Waiting for creation of ${databaseId} to complete...`);
  await operation.promise();
  console.log(
    `Created database ${databaseId} with default leader ${defaultLeader}.`,
  );
}
createDatabaseWithDefaultLeader();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\CreateDatabaseRequest;
use Google\Cloud\Spanner\Admin\Database\V1\GetDatabaseRequest;

/**
 * Creates a database with a default leader.
 * Example:
 * ```
 * create_database_with_default_leader($projectId, $instanceId, $databaseId, $defaultLeader);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 * @param string $defaultLeader The leader instance configuration used by default.
 */
function create_database_with_default_leader(
    string $projectId,
    string $instanceId,
    string $databaseId,
    string $defaultLeader
): void {
    $databaseAdminClient = new DatabaseAdminClient();

    $instance = $databaseAdminClient->instanceName($projectId, $instanceId);
    $databaseIdFull = $databaseAdminClient->databaseName($projectId, $instanceId, $databaseId);

    $operation = $databaseAdminClient->createDatabase(
        new CreateDatabaseRequest([
            'parent' => $instance,
            'create_statement' => sprintf('CREATE DATABASE `%s`', $databaseId),
            'extra_statements' => [
                'CREATE TABLE Singers (' .
                'SingerId     INT64 NOT NULL,' .
                'FirstName    STRING(1024),' .
                'LastName     STRING(1024),' .
                'SingerInfo   BYTES(MAX)' .
                ') PRIMARY KEY (SingerId)',
                'CREATE TABLE Albums (' .
                    'SingerId     INT64 NOT NULL,' .
                    'AlbumId      INT64 NOT NULL,' .
                    'AlbumTitle   STRING(MAX)' .
                ') PRIMARY KEY (SingerId, AlbumId),' .
                'INTERLEAVE IN PARENT Singers ON DELETE CASCADE',
                "ALTER DATABASE `$databaseId` SET OPTIONS(default_leader='$defaultLeader')"
            ]
        ])
    );

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    $database = $databaseAdminClient->getDatabase(
        new GetDatabaseRequest(['name' => $databaseIdFull])
    );
    printf('Created database %s on instance %s with default leader %s' . PHP_EOL,
        $databaseId, $instanceId, $database->getDefaultLeader());
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def create_database_with_default_leader(instance_id, database_id, default_leader):
    """Creates a database with tables with a default leader."""
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
                SingerInfo   BYTES(MAX)
            ) PRIMARY KEY (SingerId)""",
            """CREATE TABLE Albums (
                SingerId     INT64 NOT NULL,
                AlbumId      INT64 NOT NULL,
                AlbumTitle   STRING(MAX)
            ) PRIMARY KEY (SingerId, AlbumId),
            INTERLEAVE IN PARENT Singers ON DELETE CASCADE""",
            "ALTER DATABASE {}"
            " SET OPTIONS (default_leader = '{}')".format(database_id, default_leader),
        ],
    )
    operation = database_admin_api.create_database(request=request)

    print("Waiting for operation to complete...")
    database = operation.result(OPERATION_TIMEOUT_SECONDS)

    print(
        "Database {} created with default leader {}".format(
            database.name, database.default_leader
        )
    )
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"
# default_leader = "Spanner database default leader"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

db_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin project_id: project_id

instance_path = \
  db_admin_client.instance_path project: project_id, instance: instance_id
statements = [
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
  INTERLEAVE IN PARENT Singers ON DELETE CASCADE",

  "ALTER DATABASE `#{database_id}` SET OPTIONS (
    default_leader = '#{default_leader}'
  )"
]

job = db_admin_client.create_database \
  parent: instance_path,
  create_statement: "CREATE DATABASE `#{database_id}`",
  extra_statements: statements

job.wait_until_done!
database = job.results

puts "Created database [#{database.name}] with default leader: #{database.default_leader}"
```

## View the leader region

#### View the leader region of a database with the Google Cloud console and gcloud

### Console

1.  Go to the **Spanner Instances** page in the Google Cloud console.

2.  The leader region of your database is listed under **Leader region** .

### gcloud

To view the leader region of an existing database, run the following command:

``` text
gcloud spanner databases describe DATABASE_NAME \
--instance=INSTANCE_ID
```

Replace the following:

  - `  DATABASE_NAME  ` : the name of your database.
  - `  INSTANCE_ID  ` : the identifier of your database instance.

If a default leader region has been set, it is listed under `  defaultLeader  ` .

If a default leader has not been set, `  defaultLeader  ` is not listed. In this case Spanner uses the Google-defined, default leader region for your [dual-region](/spanner/docs/instance-configurations#available-configurations-dual) or [multi-region](/spanner/docs/instance-configurations#available-configurations-multi-region) configuration as shown in respective available configurations.

#### View the leader region in the DDL

### gcloud

To view the leader region of an a database in the DDL, run the following command:

``` text
gcloud spanner databases ddl describe DATABASE_NAME \
--instance=INSTANCE_NAME
```

Replace the following:

  - `  DATABASE_NAME  ` : the name of your database database.
  - `  INSTANCE_ID  ` : the identifier of your database instance.

### Client libraries

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

public class GetDatabaseDdlAsyncSample
{
    public async Task<GetDatabaseDdlResponse> GetDatabaseDdlAsync(string projectId, string instanceId, string databaseId)
    {
        DatabaseAdminClient databaseAdminClient = await DatabaseAdminClient.CreateAsync();
        DatabaseName databaseName = DatabaseName.FromProjectInstanceDatabase(projectId, instanceId, databaseId);
        GetDatabaseDdlResponse databaseDdl = await databaseAdminClient.GetDatabaseDdlAsync(databaseName);

        Console.WriteLine($"DDL statements for database {databaseId}:");
        foreach (var statement in databaseDdl.Statements)
        {
            Console.WriteLine(statement);
        }

        return databaseDdl;
    }
}
```

### C++

``` cpp
void GetDatabaseDdl(google::cloud::spanner_admin::DatabaseAdminClient client,
                    std::string const& project_id,
                    std::string const& instance_id,
                    std::string const& database_id) {
  namespace spanner = ::google::cloud::spanner;
  auto database = client.GetDatabaseDdl(
      spanner::Database(project_id, instance_id, database_id).FullName());
  if (!database) throw std::move(database).status();
  std::cout << "Database metadata is:\n" << database->DebugString();
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"
 "regexp"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

// getDatabaseDdl gets the DDL for the database
func getDatabaseDdl(w io.Writer, db string) error {
 // db = `projects/<project>/instances/<instance-id>/database/<database-id>`
 matches := regexp.MustCompile("^(.*)/databases/(.*)$").FindStringSubmatch(db)
 if matches == nil || len(matches) != 3 {
     return fmt.Errorf("getDatabaseDdl: invalid database id %s", db)
 }

 ctx := context.Background()
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 op, err := adminClient.GetDatabaseDdl(ctx, &adminpb.GetDatabaseDdlRequest{
     Database: db,
 })

 if err != nil {
     return err
 }

 fmt.Fprintf(w, "Database DDL is as follows: \n [%v]", op.GetStatements())

 return nil

}
```

### Java

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.spanner.admin.database.v1.DatabaseName;
import com.google.spanner.admin.database.v1.GetDatabaseDdlResponse;

public class GetDatabaseDdlSample {

  static void getDatabaseDdl() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceId = "my-instance";
    final String databaseId = "my-database";
    getDatabaseDdl(projectId, instanceId, databaseId);
  }

  static void getDatabaseDdl(
      String projectId, String instanceId, String databaseId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {
      final GetDatabaseDdlResponse response =
          databaseAdminClient.getDatabaseDdl(DatabaseName.of(projectId, instanceId, databaseId));
      System.out.println("Retrieved database DDL for " + databaseId);
      for (String ddl : response.getStatementsList()) {
        System.out.println(ddl);
      }
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance-id';
// const databaseId = 'my-database-id';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

async function getDatabaseDdl() {
  // Get the schema definition of the database.
  const [ddlStatements] = await databaseAdminClient.getDatabaseDdl({
    database: databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    ),
  });

  console.log(
    `Retrieved database DDL for ${databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    )}:`,
  );
  ddlStatements.statements.forEach(element => {
    console.log(element);
  });
}
getDatabaseDdl();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\GetDatabaseDdlRequest;

/**
 * Gets the database DDL statements.
 * Example:
 * ```
 * get_database_ddl($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function get_database_ddl(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);

    $request = new GetDatabaseDdlRequest(['database' => $databaseName]);

    $statements = $databaseAdminClient->getDatabaseDdl($request)->getStatements();

    printf("Retrieved database DDL for $databaseId" . PHP_EOL);
    foreach ($statements as $statement) {
        printf($statement . PHP_EOL);
    }
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def get_database_ddl(instance_id, database_id):
    """Gets the database DDL statements."""
    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api
    ddl = database_admin_api.get_database_ddl(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        )
    )
    print("Retrieved database DDL for {}".format(database_id))
    for statement in ddl.statements:
        print(statement)
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

db_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin project_id: project_id

db_path = db_admin_client.database_path project: project_id,
                                        instance: instance_id,
                                        database: database_id
ddl = db_admin_client.get_database_ddl database: db_path

puts ddl.statements
```

#### View the leader region in the information schema

### gcloud

To view the leader region of an existing database, run the following command:

``` text
gcloud spanner databases execute-sql DATABASE_NAME \
--instance=INSTANCE_ID \
--sql="SELECT s.OPTION_NAME, s.OPTION_VALUE \
  FROM INFORMATION_SCHEMA.DATABASE_OPTIONS s \
  WHERE s.OPTION_NAME = 'default_leader'"
```

Replace the following:

  - `  DATABASE_NAME  ` : the name of your database.
  - `  INSTANCE_ID  ` : the identifier of your database instance.

### Client libraries

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

public class GetDatabaseDefaultLeaderFromInformationSchemaAsyncSample
{
    public async Task<string> GetDatabaseDefaultLeaderFromInformationSchemaAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        using var connection = new SpannerConnection(connectionString);
        var cmd =
            connection.CreateSelectCommand(
            @"SELECT 
                s.OPTION_NAME,
                s.OPTION_VALUE
            FROM
                INFORMATION_SCHEMA.DATABASE_OPTIONS s
            WHERE
                s.OPTION_NAME = 'default_leader'");

        var defaultLeader = string.Empty;

        Console.WriteLine($"Default leader for {databaseId}:");
        using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            defaultLeader = reader.GetFieldValue<string>("OPTION_VALUE");
            Console.WriteLine(defaultLeader);
        }

        return defaultLeader;
    }
}
```

### C++

``` cpp
void QueryInformationSchemaDatabaseOptions(
    google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  // clang-format misinterprets the namespace alias followed by a block as
  // introducing a sub-namespace and so adds a misleading namespace-closing
  // comment at the end. This separating comment defeats that.
  {
    auto rows = client.ExecuteQuery(spanner::SqlStatement(R"""(
        SELECT s.OPTION_NAME, s.OPTION_VALUE
        FROM INFORMATION_SCHEMA.DATABASE_OPTIONS s
        WHERE s.OPTION_NAME = 'default_leader'
      )"""));
    using RowType = std::tuple<std::string, std::string>;
    for (auto& row : spanner::StreamOf<RowType>(rows)) {
      if (!row) throw std::move(row).status();
      std::cout << std::get<0>(*row) << "=" << std::get<1>(*row) << "\n";
    }
  }
  {
    auto rows = client.ExecuteQuery(spanner::SqlStatement(R"""(
        SELECT s.OPTION_NAME, s.OPTION_VALUE
        FROM INFORMATION_SCHEMA.DATABASE_OPTIONS s
        WHERE s.OPTION_NAME = 'version_retention_period'
      )"""));
    using RowType = std::tuple<std::string, std::string>;
    for (auto& row : spanner::StreamOf<RowType>(rows)) {
      if (!row) throw std::move(row).status();
      std::cout << std::get<0>(*row) << "=" << std::get<1>(*row) << "\n";
    }
  }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"
 "regexp"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

// queryInformationSchemaDatabaseOptions queries the database options from the
// information schema table.
func queryInformationSchemaDatabaseOptions(w io.Writer, db string) error {
 // db = `projects/<project>/instances/<instance-id>/database/<database-id>`
 matches := regexp.MustCompile("^(.+)/databases/(.+)$").FindStringSubmatch(db)
 if matches == nil || len(matches) != 3 {
     return fmt.Errorf("queryInformationSchemaDatabaseOptions: invalid database id %q", db)
 }
 databaseID := matches[2]

 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{SQL: `SELECT OPTION_NAME, OPTION_VALUE
                                 FROM INFORMATION_SCHEMA.DATABASE_OPTIONS 
                                    WHERE OPTION_NAME = 'default_leader'`}
 iter := client.Single().Query(ctx, stmt)
 defer iter.Stop()

 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var option_name, option_value string
     if err := row.Columns(&option_name, &option_value); err != nil {
         return err
     }
     fmt.Fprintf(w, "The result of the query to get %s for %s is %s", option_name, databaseID, option_value)
 }
}
```

### Java

``` java
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.ResultSet;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.Statement;

public class QueryInformationSchemaDatabaseOptionsSample {

  static void queryInformationSchemaDatabaseOptions() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceId = "my-instance";
    final String databaseId = "my-database";
    queryInformationSchemaDatabaseOptions(projectId, instanceId, databaseId);
  }

  static void queryInformationSchemaDatabaseOptions(
      String projectId, String instanceId, String databaseId) {
    try (Spanner spanner = SpannerOptions
        .newBuilder()
        .setProjectId(projectId)
        .build()
        .getService()) {
      final DatabaseId id = DatabaseId.of(projectId, instanceId, databaseId);
      final DatabaseClient databaseClient = spanner.getDatabaseClient(id);

      try (ResultSet resultSet = databaseClient
          .singleUse()
          .executeQuery(Statement.of(
              "SELECT OPTION_NAME, OPTION_VALUE"
                  + " FROM INFORMATION_SCHEMA.DATABASE_OPTIONS"
                  + " WHERE OPTION_NAME = 'default_leader'")
          )) {
        if (resultSet.next()) {
          final String optionName = resultSet.getString("OPTION_NAME");
          final String optionValue = resultSet.getString("OPTION_VALUE");

          System.out.println("The " + optionName + " for " + id + " is " + optionValue);
        } else {
          System.out.println(
              "Database " + id + " does not have a value for option 'default_leader'"
          );
        }
      }
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance-id';
// const databaseId = 'my-database-id';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});
// Gets a reference to a Cloud Spanner instance and a database.
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

async function getDatabaseDdl() {
  // Get the default leader option for the database.
  const [rows] = await database.run({
    sql: `
      SELECT s.OPTION_NAME, s.OPTION_VALUE
      FROM INFORMATION_SCHEMA.DATABASE_OPTIONS s
      WHERE s.OPTION_NAME = 'default_leader'`,
    json: true,
  });
  if (rows.length > 0) {
    const option = rows[0];
    console.log(
      `The ${option.OPTION_NAME} for ${databaseId} is ${option.OPTION_VALUE}`,
    );
  } else {
    console.log(
      `Database ${databaseId} does not have a value for option 'default_leader'`,
    );
  }
}
getDatabaseDdl();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Queries the default leader of a database.
 * Example:
 * ```
 * query_information_schema_database_options($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_information_schema_database_options(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $results = $database->execute(
        "SELECT s.OPTION_NAME, s.OPTION_VALUE
        FROM INFORMATION_SCHEMA.DATABASE_OPTIONS s
        WHERE s.OPTION_NAME = 'default_leader'"
    );

    foreach ($results as $row) {
        $optionName = $row['OPTION_NAME'];
        $optionValue = $row['OPTION_VALUE'];
        printf("The $optionName for $databaseId is $optionValue" . PHP_EOL);
    }
    if (!$results->stats()['rowCountExact']) {
        printf("Database $databaseId does not have a value for option 'default_leader'");
    }
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def query_information_schema_database_options(instance_id, database_id):
    """Queries the default leader of a database."""
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)
    with database.snapshot() as snapshot:
        results = snapshot.execute_sql(
            "SELECT OPTION_VALUE AS default_leader "
            "FROM INFORMATION_SCHEMA.DATABASE_OPTIONS "
            "WHERE SCHEMA_NAME = '' AND OPTION_NAME = 'default_leader'"
        )
        for result in results:
            print("Database {} has default leader {}".format(database_id, result[0]))
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client = spanner.client instance_id, database_id

client.execute(
  "SELECT s.OPTION_NAME, s.OPTION_VALUE " \
  "FROM INFORMATION_SCHEMA.DATABASE_OPTIONS s " \
  "WHERE s.OPTION_NAME = 'default_leader'"
).rows.each do |row|
  puts row
end
```

#### View the leader regions for multiple databases

### Client libraries

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using System;
using System.Collections.Generic;
using System.Linq;

public class ListDatabasesSample
{
    public IEnumerable<Database> ListDatabases(string projectId, string instanceId)
    {
        var databaseAdminClient = DatabaseAdminClient.Create();
        var instanceName = InstanceName.FromProjectInstance(projectId, instanceId);
        var databases = databaseAdminClient.ListDatabases(instanceName);

        // We print the first 5 elements for demonstration purposes.
        // You can print all databases in the sequence by removing the call to Take(5).
        // The sequence will lazily fetch elements in pages as needed.
        foreach (var database in databases.Take(5))
        {
            Console.WriteLine($"Default leader for database {database.DatabaseName.DatabaseId}: {database.DefaultLeader}");
        }

        return databases;
    }
}
```

### C++

``` cpp
void ListDatabases(google::cloud::spanner_admin::DatabaseAdminClient client,
                   std::string const& project_id,
                   std::string const& instance_id) {
  google::cloud::spanner::Instance in(project_id, instance_id);
  int count = 0;
  for (auto& database : client.ListDatabases(in.FullName())) {
    if (!database) throw std::move(database).status();
    std::cout << "Database " << database->name() << " full metadata:\n"
              << database->DebugString();
    ++count;
  }
  if (count == 0) {
    std::cout << "No databases found in instance " << instance_id
              << " for project << " << project_id << "\n";
  }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
 "google.golang.org/api/iterator"
)

// listDatabases gets list of all databases for an instance
func listDatabases(w io.Writer, instanceId string) error {
 // instanceId = `projects/<project>/instances/<instance-id>
 ctx := context.Background()
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 iter := adminClient.ListDatabases(ctx, &adminpb.ListDatabasesRequest{
     Parent: instanceId,
 })

 fmt.Fprintf(w, "Databases for instance/[%s]", instanceId)
 for {
     resp, err := iter.Next()
     if err == iterator.Done {
         break
     }
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%s\n", resp.Name)
 }

 return nil

}
```

### Java

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient.ListDatabasesPage;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient.ListDatabasesPagedResponse;
import com.google.spanner.admin.database.v1.Database;
import com.google.spanner.admin.database.v1.InstanceName;

public class ListDatabasesSample {

  static void listDatabases() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceId = "my-instance";
    listDatabases(projectId, instanceId);
  }

  static void listDatabases(String projectId, String instanceId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {
      ListDatabasesPagedResponse response =
          databaseAdminClient.listDatabases(InstanceName.of(projectId, instanceId));

      System.out.println("Databases for projects/" + projectId + "/instances/" + instanceId);

      for (ListDatabasesPage page : response.iteratePages()) {
        for (Database database : page.iterateAll()) {
          final String defaultLeader = database.getDefaultLeader().equals("")
              ? "" : "(default leader = " + database.getDefaultLeader() + ")";
          System.out.println("\t" + database.getName() + " " + defaultLeader);
        }
      }
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance-id';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

async function listDatabases() {
  // Lists all databases on the instance.
  const [databases] = await databaseAdminClient.listDatabases({
    parent: databaseAdminClient.instancePath(projectId, instanceId),
  });
  console.log(`Databases for projects/${projectId}/instances/${instanceId}:`);
  databases.forEach(database => {
    const defaultLeader = database.defaultLeader
      ? `(default leader = ${database.defaultLeader})`
      : '';
    console.log(`\t${database.name} ${defaultLeader}`);
  });
}
listDatabases();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\ListDatabasesRequest;

/**
 * Lists the databases and their leader options.
 * Example:
 * ```
 * list_databases($projectId, $instanceId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 */
function list_databases(string $projectId, string $instanceId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $instanceName = DatabaseAdminClient::instanceName($projectId, $instanceId);

    $request = new ListDatabasesRequest(['parent' => $instanceName]);
    $resp = $databaseAdminClient->listDatabases($request);
    $databases = $resp->iterateAllElements();
    printf('Databases for %s' . PHP_EOL, $instanceName);
    foreach ($databases as $database) {
        printf("\t%s (default leader = %s)" . PHP_EOL, $database->getName(), $database->getDefaultLeader());
    }
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def list_databases(instance_id):
    """Lists databases and their leader options."""
    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.ListDatabasesRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id)
    )

    for database in database_admin_api.list_databases(request=request):
        print(
            "Database {} has default leader {}".format(
                database.name, database.default_leader
            )
        )
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

db_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin project_id: project_id

instance_path = db_admin_client.instance_path project: project_id,
                                              instance: instance_id
databases = db_admin_client.list_databases parent: instance_path

databases.each do |db|
  puts "#{db.name} : default leader #{db.default_leader}"
end
```

## View available leader options

#### View available leader options for an instance configuration

### gcloud

To view the regions which you can set to be the default leader region, use the following command:

``` text
gcloud spanner instance-configs describe INSTANCE_CONFIG
```

Replace the following:

  - `  INSTANCE_CONFIG  ` : a permanent identifier of your instance configuration, which defines the geographic location of the instance and affects how data is replicated. For custom instance configurations, it starts with `  custom-  ` . For more information, see [instance configurations](/spanner/docs/instances) .

The regions that you can choose are listed under `  leaderOptions  ` .

### Client libraries

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Instance.V1;
using System;
using System.Threading.Tasks;

public class GetInstanceConfigAsyncSample
{
    public async Task<InstanceConfig> GetInstanceConfigAsync(string projectId, string instanceConfigId)
    {
        var instanceAdminClient = await InstanceAdminClient.CreateAsync();
        var instanceConfigName = InstanceConfigName.FromProjectInstanceConfig(projectId, instanceConfigId);
        var instanceConfig = await instanceAdminClient.GetInstanceConfigAsync(instanceConfigName);

        Console.WriteLine($"Available leader options for instance config {instanceConfigName.InstanceConfigId}:");
        foreach (var leader in instanceConfig.LeaderOptions)
        {
            Console.WriteLine(leader);
        }
        return instanceConfig;
    }
}
```

### C++

``` cpp
void GetInstanceConfig(google::cloud::spanner_admin::InstanceAdminClient client,
                       std::string const& project_id,
                       std::string const& config_id) {
  auto project = google::cloud::Project(project_id);
  auto config = client.GetInstanceConfig(project.FullName() +
                                         "/instanceConfigs/" + config_id);
  if (!config) throw std::move(config).status();
  std::cout << "The instanceConfig " << config->name()
            << " exists and its metadata is:\n"
            << config->DebugString();
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 instance "cloud.google.com/go/spanner/admin/instance/apiv1"
 "cloud.google.com/go/spanner/admin/instance/apiv1/instancepb"
)

// getInstanceConfig gets available leader options
func getInstanceConfig(w io.Writer, instanceConfigName string) error {
 // defaultLeader = `nam3`
 ctx := context.Background()
 instanceAdmin, err := instance.NewInstanceAdminClient(ctx)
 if err != nil {
     return err
 }
 defer instanceAdmin.Close()

 ic, err := instanceAdmin.GetInstanceConfig(ctx, &instancepb.GetInstanceConfigRequest{
     Name: instanceConfigName,
 })

 if err != nil {
     return fmt.Errorf("could not get instance config %s: %w", instanceConfigName, err)
 }

 fmt.Fprintf(w, "Available leader options for instance config %s: %v", instanceConfigName, ic.LeaderOptions)

 return nil
}
```

### Java

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.instance.v1.InstanceAdminClient;
import com.google.spanner.admin.instance.v1.InstanceConfig;
import com.google.spanner.admin.instance.v1.InstanceConfigName;

public class GetInstanceConfigSample {

  static void getInstanceConfig() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceConfigId = "nam6";
    getInstanceConfig(projectId, instanceConfigId);
  }

  static void getInstanceConfig(String projectId, String instanceConfigId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder()
            .setProjectId(projectId)
            .build()
            .getService();
        InstanceAdminClient instanceAdminClient = spanner.createInstanceAdminClient()) {
      final InstanceConfigName instanceConfigName = InstanceConfigName.of(projectId,
          instanceConfigId);

      final InstanceConfig instanceConfig =
          instanceAdminClient.getInstanceConfig(instanceConfigName.toString());

      System.out.printf(
          "Available leader options for instance config %s: %s%n",
          instanceConfig.getName(),
          instanceConfig.getLeaderOptionsList()
      );
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
/**
 * TODO(developer): Uncomment the following line before running the sample.
 */
// const projectId = 'my-project-id';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const instanceAdminClient = spanner.getInstanceAdminClient();

async function getInstanceConfig() {
  // Get the instance config for the multi-region North America 6 (NAM6).
  // See https://cloud.google.com/spanner/docs/instance-configurations#configuration for a list of all available
  // configurations.
  const [instanceConfig] = await instanceAdminClient.getInstanceConfig({
    name: instanceAdminClient.instanceConfigPath(projectId, 'nam6'),
  });
  console.log(
    `Available leader options for instance config ${instanceConfig.name} ('${
      instanceConfig.displayName
    }'): 
         ${instanceConfig.leaderOptions.join()}`,
  );
}
getInstanceConfig();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

``` php
use Google\Cloud\Spanner\Admin\Instance\V1\Client\InstanceAdminClient;
use Google\Cloud\Spanner\Admin\Instance\V1\GetInstanceConfigRequest;

/**
 * Gets the leader options for the instance configuration.
 *
 * @param string $projectId The Google Cloud Project ID.
 * @param string $instanceConfig The name of the instance configuration.
 */
function get_instance_config(string $projectId, string $instanceConfig): void
{
    $instanceAdminClient = new InstanceAdminClient();
    $instanceConfigName = InstanceAdminClient::instanceConfigName($projectId, $instanceConfig);

    $request = (new GetInstanceConfigRequest())
        ->setName($instanceConfigName);
    $configInfo = $instanceAdminClient->getInstanceConfig($request);

    printf('Available leader options for instance config %s: %s' . PHP_EOL,
        $instanceConfig,
        implode(',', array_keys(iterator_to_array($configInfo->getLeaderOptions())))
    );
}
```

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def get_instance_config(instance_config):
    """Gets the leader options for the instance configuration."""
    spanner_client = spanner.Client()
    config_name = "{}/instanceConfigs/{}".format(
        spanner_client.project_name, instance_config
    )
    config = spanner_client.instance_admin_api.get_instance_config(name=config_name)
    print(
        "Available leader options for instance config {}: {}".format(
            instance_config, config.leader_options
        )
    )
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_config_id = "Spanner instance config ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/instance"

instance_admin_client = Google::Cloud::Spanner::Admin::Instance.instance_admin

instance_config_path = instance_admin_client.instance_config_path \
  project: project_id, instance_config: instance_config_id
config = instance_admin_client.get_instance_config name: instance_config_path

puts "Available leader options for instance config #{config.name} : #{config.leader_options}"
```

#### View available leader options for all instance configurations

### gcloud

To view the regions which you can set to be the default leader region for all of the instance configs, run the following command:

``` text
gcloud spanner instance-configs list
```

### Client libraries

### C\#

``` csharp
using Google.Api.Gax.ResourceNames;
using Google.Cloud.Spanner.Admin.Instance.V1;
using System;
using System.Collections.Generic;
using System.Linq;

public class ListInstanceConfigsSample
{
    public IEnumerable<InstanceConfig> ListInstanceConfigs(string projectId)
    {
        var instanceAdminClient = InstanceAdminClient.Create();
        var projectName = ProjectName.FromProject(projectId);
        var instanceConfigs = instanceAdminClient.ListInstanceConfigs(projectName);

        // We print the first 5 elements for demonstration purposes.
        // You can print all configs in the sequence by removing the call to Take(5).
        // The sequence will lazily fetch elements in pages as needed.
        foreach (var instanceConfig in instanceConfigs.Take(5))
        {
            Console.WriteLine($"Available leader options for instance config {instanceConfig.InstanceConfigName.InstanceConfigId}:");
            foreach (var leader in instanceConfig.LeaderOptions)
            {
                Console.WriteLine(leader);
            }

            Console.WriteLine($"Available optional replica for instance config {instanceConfig.InstanceConfigName.InstanceConfigId}:");
            foreach (var optionalReplica in instanceConfig.OptionalReplicas)
            {
                Console.WriteLine($"Replica type - {optionalReplica.Type}, default leader location - {optionalReplica.DefaultLeaderLocation}, location - {optionalReplica.Location}");
            }
        }
        return instanceConfigs;
    }
}
```

### C++

``` cpp
void ListInstanceConfigs(
    google::cloud::spanner_admin::InstanceAdminClient client,
    std::string const& project_id) {
  int count = 0;
  auto project = google::cloud::Project(project_id);
  for (auto& config : client.ListInstanceConfigs(project.FullName())) {
    if (!config) throw std::move(config).status();
    ++count;
    std::cout << "Instance config [" << count << "]:\n"
              << config->DebugString();
  }
  if (count == 0) {
    std::cout << "No instance configs found in project " << project_id << "\n";
  }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 instance "cloud.google.com/go/spanner/admin/instance/apiv1"
 "cloud.google.com/go/spanner/admin/instance/apiv1/instancepb"
 "google.golang.org/api/iterator"
)

// istInstanceConfigs gets available leader options for all instances
func listInstanceConfigs(w io.Writer, projectName string) error {
 // projectName = `projects/<project>
 ctx := context.Background()
 instanceAdmin, err := instance.NewInstanceAdminClient(ctx)
 if err != nil {
     return err
 }
 defer instanceAdmin.Close()

 request := &instancepb.ListInstanceConfigsRequest{
     Parent: projectName,
 }
 for {
     iter := instanceAdmin.ListInstanceConfigs(ctx, request)
     for {
         ic, err := iter.Next()
         if err == iterator.Done {
             break
         }
         if err != nil {
             return err
         }
         fmt.Fprintf(w, "Available leader options for instance config %s: %v\n", ic.Name, ic.LeaderOptions)
     }
     pageToken := iter.PageInfo().Token
     if pageToken == "" {
         break
     } else {
         request.PageToken = pageToken
     }
 }

 return nil
}
```

### Java

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.instance.v1.InstanceAdminClient;
import com.google.spanner.admin.instance.v1.InstanceConfig;
import com.google.spanner.admin.instance.v1.ProjectName;

public class ListInstanceConfigsSample {

  static void listInstanceConfigs() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    listInstanceConfigs(projectId);
  }

  static void listInstanceConfigs(String projectId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder()
            .setProjectId(projectId)
            .build()
            .getService();
        InstanceAdminClient instanceAdminClient = spanner.createInstanceAdminClient()) {
      final ProjectName projectName = ProjectName.of(projectId);
      for (InstanceConfig instanceConfig :
          instanceAdminClient.listInstanceConfigs(projectName).iterateAll()) {
        System.out.printf(
            "Available leader options for instance config %s: %s%n",
            instanceConfig.getName(),
            instanceConfig.getLeaderOptionsList()
        );
      }
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
/**
 * TODO(developer): Uncomment the following line before running the sample.
 */
// const projectId = 'my-project-id';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const instanceAdminClient = spanner.getInstanceAdminClient();

async function listInstanceConfigs() {
  // Lists all available instance configurations in the project.
  // See https://cloud.google.com/spanner/docs/instance-configurations#configuration for a list of all available
  // configurations.
  const [instanceConfigs] = await instanceAdminClient.listInstanceConfigs({
    parent: instanceAdminClient.projectPath(projectId),
  });
  console.log(`Available instance configs for project ${projectId}:`);
  instanceConfigs.forEach(instanceConfig => {
    console.log(
      `Available leader options for instance config ${
        instanceConfig.name
      } ('${instanceConfig.displayName}'): 
         ${instanceConfig.leaderOptions.join()}`,
    );
  });
}
listInstanceConfigs();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Instance\V1\Client\InstanceAdminClient;
use Google\Cloud\Spanner\Admin\Instance\V1\ListInstanceConfigsRequest;

/**
 * Lists the available instance configurations.
 * Example:
 * ```
 * list_instance_configs();
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 */
function list_instance_configs(string $projectId): void
{
    $instanceAdminClient = new InstanceAdminClient();
    $projectName = InstanceAdminClient::projectName($projectId);
    $request = new ListInstanceConfigsRequest();
    $request->setParent($projectName);
    $resp = $instanceAdminClient->listInstanceConfigs($request);
    foreach ($resp as $element) {
        printf(
            'Available leader options for instance config %s: %s' . PHP_EOL,
            $element->getDisplayName(),
            implode(',', iterator_to_array($element->getLeaderOptions()))
        );
    }
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def list_instance_config():
    """Lists the available instance configurations."""
    from google.cloud.spanner_admin_instance_v1.types import spanner_instance_admin

    spanner_client = spanner.Client()

    request = spanner_instance_admin.ListInstanceConfigsRequest(
        parent=spanner_client.project_name
    )
    for config in spanner_client.instance_admin_api.list_instance_configs(
        request=request
    ):
        print(
            "Available leader options for instance config {}: {}".format(
                config.name, config.leader_options
            )
        )
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/instance"

instance_admin_client = Google::Cloud::Spanner::Admin::Instance.instance_admin

project_path = instance_admin_client.project_path project: project_id
configs = instance_admin_client.list_instance_configs parent: project_path

configs.each do |c|
  puts "Available leader options for instance config #{c.name} : #{c.leader_options}"
end
```
