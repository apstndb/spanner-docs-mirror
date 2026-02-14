This page describes Spanner restoration operations and explains how to restore a database. For more information about database restoration in Spanner, see [Restore overview](/spanner/docs/backup/restore-backup-overview) .

You can restore a Spanner database by using the following:

  - In the Google Cloud console.
  - Using the Google Cloud CLI.
  - Using the client libraries.
  - Using the [REST](/spanner/docs/reference/rest/v1/projects.instances.backups) or [RPC](/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.Backup) APIs.

## Before you begin

  - The gcloud CLI examples on this page make the following assumptions:
    
      - You have already set up [gcloud CLI](/sdk/docs) for use with Spanner. If you are new to using gcloud CLI with Spanner, see [Getting started with Spanner using gcloud CLI](/spanner/docs/getting-started/gcloud) .
      - You have configured gcloud CLI with your project. For example:
    
    <!-- end list -->
    
    ``` text
      gcloud config set core/project PROJECT_ID
      
    ```
    
    ``` text
    *   You have an instance named `test-instance` and a database named
        `example-db`.
    ```

  - To get the permissions that you need to restore a database from a backup, ask your administrator to grant you the following IAM roles on the instance:
    
      - All: [Cloud Spanner Restore Admin](/iam/docs/roles-permissions/spanner#spanner.restoreAdmin) ( `  roles/spanner.restoreAdmin  ` )
    
    **Note:** [IAM basic roles](/iam/docs/roles-overview#basic) might also contain permissions to restore a database from a backup. You shouldn't grant basic roles in a production environment, but you can grant them in a development or test environment.

## Restore a database from a backup

### Console

1.  Go to the Spanner Instances page in the Google Cloud console.

<!-- end list -->

1.  Click the instance containing the database to open its **Overview** page.

2.  Click the database to open its **Overview** page.

3.  In the navigation pane, click **Backup/Restore** .

4.  Click the **Actions** button for your selected backup, then select **Restore** .

5.  Fill out the form and click the **Restore** button.

To check the progress of the operation, see the progress indicator, as shown in the operations page:

If the operation takes too long, you can cancel it. For more information, see [Cancel a long-running instance operation](/spanner/docs/manage-and-observe-long-running-operations#cancel_a_long-running_instance_operation) .

**Note:** If you are restoring a database from a backup or a copied backup, the restore operation appears in the instance containing the restored database, not the instance containing the backup.

### gcloud

To restore a database, use [`  gcloud spanner databases restore  `](/sdk/gcloud/reference/spanner/databases/restore) :

``` text
gcloud spanner databases restore --async \
--destination-instance=test-instance --destination-database=example-db-restored \
--source-instance=test-instance --source-backup=example-db-backup-6 \
--encryption_type=google-managed-encryption
```

Usage notes:

  - The command returns immediately because of the `  --async  ` flag. Without the flag, the command will wait for the restore operation to complete.
  - If the source and destination instances are different, they must have the same [instance configuration](/spanner/docs/instances#configuration) .
  - If the destination database already exists, the operation will fail.
  - The possible values for `  encryption_type  ` are `  USE_DATABASE_ENCRYPTION  ` , `  GOOGLE_DEFAULT_ENCRYPTION  ` , or `  CUSTOMER_MANAGED_ENCRYPTION  ` . If you use `  CUSTOMER_MANAGED_ENCRYPTION  ` , you must specify a `  kmsKeyName  ` .

### Client libraries

The following code sample restores a database from a given backup and waits for the restore operation (an operation with [`  RestoreDatabaseMetadata  `](/spanner/docs/reference/rest/v1/RestoreDatabaseMetadata) ) to complete. The restored database is created in the same instance as the backup. Once complete, the sample retrieves and prints some restore information from the database.

### C++

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

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

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

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

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

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

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

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

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
