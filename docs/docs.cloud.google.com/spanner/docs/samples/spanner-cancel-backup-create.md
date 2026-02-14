Cancel a create backup database operation.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Manage backups](/spanner/docs/backup/manage-backups)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void CreateBackupAndCancel(
    google::cloud::spanner_admin::DatabaseAdminClient client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id, std::string const& backup_id,
    google::cloud::spanner::Timestamp expire_time) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  google::spanner::admin::database::v1::CreateBackupRequest request;
  request.set_parent(database.instance().FullName());
  request.set_backup_id(backup_id);
  request.mutable_backup()->set_database(database.FullName());
  *request.mutable_backup()->mutable_expire_time() =
      expire_time.get<google::protobuf::Timestamp>().value();
  auto f = client.CreateBackup(request);
  f.cancel();
  auto backup = f.get();
  if (backup) {
    auto status = client.DeleteBackup(backup->name());
    if (!status.ok()) throw std::move(status);
    std::cout << "Backup " << backup->name() << " was deleted.\n";
  } else {
    std::cout << "CreateBackup operation was cancelled with the message '"
              << backup.status().message() << "'.\n";
  }
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using Google.LongRunning;
using Google.Protobuf.WellKnownTypes;
using System;

public class CancelBackupOperationSample
{
    public Operation<Backup, CreateBackupMetadata> CancelBackupOperation(string projectId, string instanceId, string databaseId, string backupId)
    {
        // Create the DatabaseAdminClient instance.
        DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();

        // Initialize backup request parameters.
        Backup backup = new Backup
        {
            DatabaseAsDatabaseName = DatabaseName.FromProjectInstanceDatabase(projectId, instanceId, databaseId),
            ExpireTime = DateTime.UtcNow.AddDays(14).ToTimestamp()
        };
        InstanceName parentAsInstanceName = InstanceName.FromProjectInstance(projectId, instanceId);

        // Make the CreateBackup request.
        Operation<Backup, CreateBackupMetadata> operation = databaseAdminClient.CreateBackup(parentAsInstanceName, backup, backupId);

        // Cancel the operation.
        operation.Cancel();

        // Poll until the long-running operation is completed in case the backup was
        // created before the operation was cancelled.
        Console.WriteLine("Waiting for the operation to finish.");
        Operation<Backup, CreateBackupMetadata> completedOperation = operation.PollUntilCompleted();

        if (completedOperation.IsFaulted)
        {
            Console.WriteLine($"Create backup operation cancelled: {operation.Name}");
        }
        else
        {
            Console.WriteLine("The backup was created before the operation was cancelled. Backup needs to be deleted.");
            BackupName backupAsBackupName = BackupName.FromProjectInstanceBackup(projectId, instanceId, backupId);
            databaseAdminClient.DeleteBackup(backupAsBackupName);
        }

        return completedOperation;
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
 "time"

 longrunning "cloud.google.com/go/longrunning/autogen/longrunningpb"
 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
 pbt "github.com/golang/protobuf/ptypes/timestamp"
 "google.golang.org/grpc/codes"
 "google.golang.org/grpc/status"
)

func cancelBackup(ctx context.Context, w io.Writer, db, backupID string) error {
 matches := regexp.MustCompile("^(.+)/databases/(.+)$").FindStringSubmatch(db)
 if matches == nil || len(matches) != 3 {
     return fmt.Errorf("cancelBackup: invalid database id %q", db)
 }

 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return fmt.Errorf("cancelBackup.NewDatabaseAdminClient: %w", err)
 }
 defer adminClient.Close()

 expireTime := time.Now().AddDate(0, 0, 14)
 // Create a backup.
 req := adminpb.CreateBackupRequest{
     Parent:   matches[1],
     BackupId: backupID,
     Backup: &adminpb.Backup{
         Database:   db,
         ExpireTime: &pbt.Timestamp{Seconds: expireTime.Unix(), Nanos: int32(expireTime.Nanosecond())},
     },
 }
 op, err := adminClient.CreateBackup(ctx, &req)
 if err != nil {
     return fmt.Errorf("cancelBackup.CreateBackup: %w", err)
 }

 // Cancel backup creation.
 err = adminClient.LROClient.CancelOperation(ctx, &longrunning.CancelOperationRequest{Name: op.Name()})
 if err != nil {
     return fmt.Errorf("cancelBackup.CancelOperation: %w", err)
 }

 // Cancel operations are best effort so either it will complete or be
 // cancelled.
 backup, err := op.Wait(ctx)
 if err != nil {
     if waitStatus, ok := status.FromError(err); !ok || waitStatus.Code() != codes.Canceled {
         return fmt.Errorf("cancelBackup.Wait: %w", err)
     }
 } else {
     // Backup was completed before it could be cancelled so delete the
     // unwanted backup.
     err = adminClient.DeleteBackup(ctx, &adminpb.DeleteBackupRequest{Name: backup.Name})
     if err != nil {
         return fmt.Errorf("cancelBackup.DeleteBackup: %w", err)
     }
 }

 fmt.Fprintf(w, "Backup cancelled.\n")
 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void cancelCreateBackup(
    DatabaseAdminClient dbAdminClient, String projectId, String instanceId,
    String databaseId, String backupId) {
  // Set expire time to 14 days from now.
  Timestamp expireTime =
      Timestamp.newBuilder().setSeconds(TimeUnit.MILLISECONDS.toSeconds((
          System.currentTimeMillis() + TimeUnit.DAYS.toMillis(14)))).build();
  BackupName backupName = BackupName.of(projectId, instanceId, backupId);
  Backup backup = Backup.newBuilder()
      .setName(backupName.toString())
      .setDatabase(DatabaseName.of(projectId, instanceId, databaseId).toString())
      .setExpireTime(expireTime).build();

  try {
    // Start the creation of a backup.
    System.out.println("Creating backup [" + backupId + "]...");
    OperationFuture<Backup, CreateBackupMetadata> op = dbAdminClient.createBackupAsync(
        InstanceName.of(projectId, instanceId), backup, backupId);

    // Try to cancel the backup operation.
    System.out.println("Cancelling create backup operation for [" + backupId + "]...");
    dbAdminClient.getOperationsClient().cancelOperation(op.getName());

    // Get a polling future for the running operation. This future will regularly poll the server
    // for the current status of the backup operation.
    RetryingFuture<OperationSnapshot> pollingFuture = op.getPollingFuture();

    // Wait for the operation to finish.
    // isDone will return true when the operation is complete, regardless of whether it was
    // successful or not.
    while (!pollingFuture.get().isDone()) {
      System.out.println("Waiting for the cancelled backup operation to finish...");
      Thread.sleep(TimeUnit.MILLISECONDS.convert(5, TimeUnit.SECONDS));
    }
    if (pollingFuture.get().getErrorCode() == null) {
      // Backup was created before it could be cancelled. Delete the backup.
      dbAdminClient.deleteBackup(backupName);
      System.out.println("Backup operation for [" + backupId
          + "] successfully finished before it could be cancelled");
    } else if (pollingFuture.get().getErrorCode().getCode() == StatusCode.Code.CANCELLED) {
      System.out.println("Backup operation for [" + backupId + "] successfully cancelled");
    }
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
const {Spanner, protos} = require('@google-cloud/spanner');
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

// Creates a new backup of the database
try {
  console.log(
    `Creating backup of database ${databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    )}.`,
  );

  // Expire backup one day in the future
  const expireTime = Date.now() + 1000 * 60 * 60 * 24;
  const [operation] = await databaseAdminClient.createBackup({
    parent: databaseAdminClient.instancePath(projectId, instanceId),
    backupId: backupId,
    backup: (protos.google.spanner.admin.database.v1.Backup = {
      database: databaseAdminClient.databasePath(
        projectId,
        instanceId,
        databaseId,
      ),
      expireTime: Spanner.timestamp(expireTime).toStruct(),
      name: databaseAdminClient.backupPath(projectId, instanceId, backupId),
    }),
  });

  // Cancel the backup
  await operation.cancel();

  console.log('Backup cancelled.');
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Delete backup in case it got created before the cancel operation
  await databaseAdminClient.deleteBackup({
    name: databaseAdminClient.backupPath(projectId, instanceId, backupId),
  });
  // Close the spanner client when finished.
  // The databaseAdminClient does not require explicit closure. The closure of the Spanner client will automatically close the databaseAdminClient.
  spanner.close();
}
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\ApiCore\ApiException;
use Google\Cloud\Spanner\Admin\Database\V1\Backup;
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\CreateBackupRequest;
use Google\Cloud\Spanner\Admin\Database\V1\DeleteBackupRequest;
use Google\Cloud\Spanner\Admin\Database\V1\GetBackupRequest;
use Google\Protobuf\Timestamp;

/**
 * Cancel a backup operation.
 * Example:
 * ```
 * cancel_backup($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function cancel_backup(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseFullName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);
    $instanceFullName = DatabaseAdminClient::instanceName($projectId, $instanceId);
    $expireTime = new Timestamp();
    $expireTime->setSeconds((new \DateTime('+14 days'))->getTimestamp());
    $backupId = uniqid('backup-' . $databaseId . '-cancel');
    $request = new CreateBackupRequest([
        'parent' => $instanceFullName,
        'backup_id' => $backupId,
        'backup' => new Backup([
            'database' => $databaseFullName,
            'expire_time' => $expireTime
        ])
    ]);

    $operation = $databaseAdminClient->createBackup($request);
    $operation->cancel();

    // Cancel operations are always successful regardless of whether the operation is
    // still in progress or is complete.
    printf('Cancel backup operation complete.' . PHP_EOL);

    // Operation may succeed before cancel() has been called. So we need to clean up created backup.
    try {
        $request = new GetBackupRequest();
        $request->setName($databaseAdminClient->backupName($projectId, $instanceId, $backupId));
        $info = $databaseAdminClient->getBackup($request);
    } catch (ApiException $ex) {
        return;
    }
    $databaseAdminClient->deleteBackup(new DeleteBackupRequest([
        'name' => $databaseAdminClient->backupName($projectId, $instanceId, $backupId)
    ]));
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def cancel_backup(instance_id, database_id, backup_id):
    from google.cloud.spanner_admin_database_v1.types import backup as backup_pb

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    expire_time = datetime.utcnow() + timedelta(days=30)

    # Create a backup.
    request = backup_pb.CreateBackupRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        backup_id=backup_id,
        backup=backup_pb.Backup(
            database=database_admin_api.database_path(
                spanner_client.project, instance_id, database_id
            ),
            expire_time=expire_time,
        ),
    )

    operation = database_admin_api.create_backup(request)
    # Cancel backup creation.
    operation.cancel()

    # Cancel operations are the best effort so either it will complete or
    # be cancelled.
    while not operation.done():
        time.sleep(300)  # 5 mins

    try:
        database_admin_api.get_backup(
            backup_pb.GetBackupRequest(
                name=database_admin_api.backup_path(
                    spanner_client.project, instance_id, backup_id
                ),
            )
        )
    except NotFound:
        print("Backup creation was successfully cancelled.")
        return
    print("Backup was created before the cancel completed.")
    database_admin_api.delete_backup(
        backup_pb.DeleteBackupRequest(
            name=database_admin_api.backup_path(
                spanner_client.project, instance_id, backup_id
            ),
        )
    )
    print("Backup deleted.")
```

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"
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

expire_time = Time.now + (14 * 24 * 3600) # 14 days from now

job = database_admin_client.create_backup parent: instance_path,
                                          backup_id: backup_id,
                                          backup: {
                                            database: db_path,
                                              expire_time: expire_time
                                          }

puts "Backup operation in progress"

job.cancel
job.wait_until_done!

begin
  backup = database_admin_client.get_backup name: backup_path
  database_admin_client.delete_backup name: backup_path if backup
rescue StandardError
  nil # no cleanup needed when a backup is not created
end
puts "#{backup_id} creation job cancelled"
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
