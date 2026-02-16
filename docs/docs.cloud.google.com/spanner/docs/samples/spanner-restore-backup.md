Restore a database from a backup.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Recover data using point-in-time recovery (PITR)](/spanner/docs/use-pitr)
  - [Restore from a backup](/spanner/docs/backup/restore-backups)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void RestoreDatabase(google::cloud::spanner_admin::DatabaseAdminClient client,
                     std::string const& project_id,
                     std::string const& instance_id,
                     std::string const& database_id,
                     std::string const& backup_id) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  google::cloud::spanner::Backup backup(database.instance(), backup_id);
  auto restored_db =
      client
          .RestoreDatabase(database.instance().FullName(),
                           database.database_id(), backup.FullName())
          .get();
  if (!restored_db) throw std::move(restored_db).status();
  std::cout << "Database";
  if (restored_db->restore_info().source_type() ==
      google::spanner::admin::database::v1::BACKUP) {
    auto const& backup_info = restored_db->restore_info().backup_info();
    std::cout << " " << backup_info.source_database() << " as of "
              << google::cloud::spanner::MakeTimestamp(
                     backup_info.version_time())
                     .value();
  }
  std::cout << " restored to " << restored_db->name();
  std::cout << " from backup " << backup.FullName();
  std::cout << ".\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using Google.LongRunning;
using System;

public class RestoreDatabaseSample
{
    public RestoreInfo RestoreDatabase(string projectId, string instanceId, string databaseId, string backupId)
    {
        // Create the DatabaseAdminClient instance.
        DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();

        InstanceName parentAsInstanceName = InstanceName.FromProjectInstance(projectId, instanceId);
        BackupName backupAsBackupName = BackupName.FromProjectInstanceBackup(projectId, instanceId, backupId);

        // Make the RestoreDatabase request.
        Operation<Database, RestoreDatabaseMetadata> response = databaseAdminClient.RestoreDatabase(parentAsInstanceName, databaseId, backupAsBackupName);

        Console.WriteLine("Waiting for the operation to finish");

        // Poll until the returned long-running operation is complete.
        var completedResponse = response.PollUntilCompleted();

        if (completedResponse.IsFaulted)
        {
            Console.WriteLine($"Database Restore Failed: {completedResponse.Exception}");
            throw completedResponse.Exception;
        }

        RestoreInfo restoreInfo = completedResponse.Result.RestoreInfo;
        Console.WriteLine(
            $"Database {restoreInfo.BackupInfo.SourceDatabase} was restored " +
            $"to {databaseId} from backup {restoreInfo.BackupInfo.Backup} " +
            $"with version time {restoreInfo.BackupInfo.VersionTime}");

        return restoreInfo;
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

func restoreBackup(ctx context.Context, w io.Writer, db, backupID string) error {
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 matches := regexp.MustCompile("^(.*)/databases/(.*)$").FindStringSubmatch(db)
 if matches == nil || len(matches) != 3 {
     return fmt.Errorf("Invalid database id %s", db)
 }
 instanceName := matches[1]
 databaseID := matches[2]
 backupName := instanceName + "/backups/" + backupID

 // Start restoring backup to a new database.
 restoreOp, err := adminClient.RestoreDatabase(ctx, &adminpb.RestoreDatabaseRequest{
     Parent:     instanceName,
     DatabaseId: databaseID,
     Source: &adminpb.RestoreDatabaseRequest_Backup{
         Backup: backupName,
     },
 })
 if err != nil {
     return err
 }
 // Wait for restore operation to complete.
 dbObj, err := restoreOp.Wait(ctx)
 if err != nil {
     return err
 }
 // Newly created database has restore information.
 backupInfo := dbObj.RestoreInfo.GetBackupInfo()
 if backupInfo != nil {
     fmt.Fprintf(w, "Source database %s restored from backup %s\n", backupInfo.SourceDatabase, backupInfo.Backup)
 }

 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void restoreBackup(
    DatabaseAdminClient dbAdminClient,
    String projectId,
    String instanceId,
    String backupId,
    String restoreToDatabaseId) {
  BackupName backupName = BackupName.of(projectId, instanceId, backupId);
  Backup backup = dbAdminClient.getBackup(backupName);
  // Initiate the request which returns an OperationFuture.
  System.out.println(String.format(
      "Restoring backup [%s] to database [%s]...", backup.getName(), restoreToDatabaseId));
  try {
    RestoreDatabaseRequest request =
        RestoreDatabaseRequest.newBuilder()
            .setParent(InstanceName.of(projectId, instanceId).toString())
            .setDatabaseId(restoreToDatabaseId)
            .setBackup(backupName.toString()).build();
    OperationFuture<com.google.spanner.admin.database.v1.Database, RestoreDatabaseMetadata> op =
        dbAdminClient.restoreDatabaseAsync(request);
    // Wait until the database has been restored.
    com.google.spanner.admin.database.v1.Database db = op.get();
    // Get the restore info.
    RestoreInfo restoreInfo = db.getRestoreInfo();
    BackupInfo backupInfo = restoreInfo.getBackupInfo();

    System.out.println(
        "Restored database ["
            + db.getName()
            + "] from ["
            + restoreInfo.getBackupInfo().getBackup()
            + "] with version time [" + backupInfo.getVersionTime() + "]");
  } catch (ExecutionException e) {
    throw SpannerExceptionFactory.newSpannerException(e.getCause());
  } catch (InterruptedException e) {
    throw SpannerExceptionFactory.propagateInterrupt(e);
  }
}
```

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` javascript
// Imports the Google Cloud client library and precise date library
const {Spanner} = require('@google-cloud/spanner');
const {PreciseDate} = require('@google-cloud/precise-date');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';
// const backupId = 'my-backup';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner Database Admin Client object
const databaseAdminClient = spanner.getDatabaseAdminClient();

// Restore the database
console.log(
  `Restoring database ${databaseAdminClient.databasePath(
    projectId,
    instanceId,
    databaseId,
  )} from backup ${backupId}.`,
);
const [restoreOperation] = await databaseAdminClient.restoreDatabase({
  parent: databaseAdminClient.instancePath(projectId, instanceId),
  databaseId: databaseId,
  backup: databaseAdminClient.backupPath(projectId, instanceId, backupId),
});

// Wait for restore to complete
console.log('Waiting for database restore to complete...');
await restoreOperation.promise();

console.log('Database restored from backup.');
const [metadata] = await databaseAdminClient.getDatabase({
  name: databaseAdminClient.databasePath(projectId, instanceId, databaseId),
});
console.log(
  `Database ${metadata.restoreInfo.backupInfo.sourceDatabase} was restored ` +
    `to ${databaseId} from backup ${metadata.restoreInfo.backupInfo.backup} ` +
    'with version time ' +
    `${new PreciseDate(
      metadata.restoreInfo.backupInfo.versionTime,
    ).toISOString()}.`,
);
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\RestoreDatabaseRequest;

/**
 * Restore a database from a backup.
 * Example:
 * ```
 * restore_backup($projectId, $instanceId, $databaseId, $backupId);
 * ```
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 * @param string $backupId The Spanner backup ID.
 */
function restore_backup(
    string $projectId,
    string $instanceId,
    string $databaseId,
    string $backupId
): void {
    $databaseAdminClient = new DatabaseAdminClient();

    $backupName = DatabaseAdminClient::backupName($projectId, $instanceId, $backupId);
    $instanceName = DatabaseAdminClient::instanceName($projectId, $instanceId);

    $request = new RestoreDatabaseRequest([
        'parent' => $instanceName,
        'database_id' => $databaseId,
        'backup' => $backupName
    ]);

    $operationResponse = $databaseAdminClient->restoreDatabase($request);
    $operationResponse->pollUntilComplete();

    $database = $operationResponse->operationSucceeded() ? $operationResponse->getResult() : null;
    $restoreInfo = $database->getRestoreInfo();
    $backupInfo = $restoreInfo->getBackupInfo();
    $sourceDatabase = $backupInfo->getSourceDatabase();
    $sourceBackup = $backupInfo->getBackup();
    $versionTime = $backupInfo->getVersionTime()->getSeconds();
    printf(
        'Database %s restored from backup %s with version time %s' . PHP_EOL,
        $sourceDatabase, $sourceBackup, $versionTime
    );
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def restore_database(instance_id, new_database_id, backup_id):
    """Restores a database from a backup."""
    from google.cloud.spanner_admin_database_v1 import RestoreDatabaseRequest

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    # Start restoring an existing backup to a new database.
    request = RestoreDatabaseRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        database_id=new_database_id,
        backup=database_admin_api.backup_path(
            spanner_client.project, instance_id, backup_id
        ),
    )
    operation = database_admin_api.restore_database(request)

    # Wait for restore operation to complete.
    db = operation.result(1600)

    # Newly created database has restore information.
    restore_info = db.restore_info
    print(
        "Database {} restored to {} from backup {} with version time {}.".format(
            restore_info.backup_info.source_database,
            new_database_id,
            restore_info.backup_info.backup,
            restore_info.backup_info.version_time,
        )
    )
```

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID of where to restore"
# backup_id = "Your Spanner backup ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin

instance_path = database_admin_client.instance_path project: project_id, instance: instance_id

db_path = database_admin_client.database_path project: project_id,
                                              instance: instance_id,
                                              database: database_id

backup_path = database_admin_client.backup_path project: project_id,
                                                instance: instance_id,
                                                backup: backup_id

job = database_admin_client.restore_database parent: instance_path,
                                             database_id: database_id,
                                             backup: backup_path

puts "Waiting for restore backup operation to complete"

job.wait_until_done!

database = database_admin_client.get_database name: db_path
restore_info = database.restore_info
puts "Database #{restore_info.backup_info.source_database} was restored to #{database_id} from backup #{restore_info.backup_info.backup} with version time #{restore_info.backup_info.version_time}"
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
