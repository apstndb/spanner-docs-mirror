This page offers information about Spanner backup operations. For more information about backups, see [Backups overview](/spanner/docs/backup) .

## Before you begin

  - To get the permissions that you need to manage backups, ask your administrator to grant you the following IAM roles on the instance:
    
      - Create, view, update, and delete backups: [Cloud Spanner Backup Admin](/iam/docs/roles-permissions/spanner#spanner.backupAdmin) ( `  roles/spanner.backupAdmin  ` )
      - Create and view backups: [Cloud Spanner Backup Writer](/iam/docs/roles-permissions/spanner#spanner.backupWriter) ( `  roles/spanner.backupWriter  ` )
    
    **Note:** [IAM basic roles](/iam/docs/roles-overview#basic) might also contain permissions to manage backups. You shouldn't grant basic roles in a production environment, but you can grant them in a development or test environment.

  - The Google Cloud CLI examples on this page make the following assumptions:
    
      - You have already set up [gcloud CLI](/sdk/docs) for use with Spanner. If you are new to using gcloud CLI with Spanner, see [Getting started with Spanner using gcloud CLI](/spanner/docs/getting-started/gcloud) .
    
      - You have configured gcloud CLI with your project. For example:
        
        ``` text
        gcloud config set core/project PROJECT_ID
        ```

## Copy a backup

### Console

1.  In the Google Cloud console, go to the Spanner **Instances** page.

2.  Click the instance that contains the database that you want to copy.

3.  Click the database.

4.  In the navigation pane, click **Backup/Restore** .

5.  In the **Backups** table, select **Actions** for your backup and click **Copy** .

6.  Fill out the form by choosing a destination instance, providing a name, and selecting an expiration date for the backup copy.

7.  Click **Copy** .

To check the progress of a copy operation, see [Check the operation progress](#check-operation-progress) .

If the operation takes too long, you can cancel it. For more information, see [Cancel a long-running instance operation](/spanner/docs/manage-and-observe-long-running-operations#cancel_a_long-running_instance_operation) .

**Note:** If you are copying to a different instance, the copy backup operation appears in the destination instance containing the copied backup, not the instance containing the source backup. The Google Cloud console doesn't support copying a backup to an instance in a different project. To copy a backup to a different project, use the gcloud CLI, client libraries, [REST APIs](/spanner/docs/reference/rest/v1/projects.instances.backups) , or [RPC APIs](/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.Backup) .

### gcloud

You can copy a backup to a different instance in the same project, or to a different instance in a different project.

### Copy a backup in the same project

If you choose to copy the backup to a different instance in the same project, you must create a new instance (or have one ready) for the copied backup. You can't create a new instance as part of the backup copy operation. Also, the expiration time of the backup must be at least six hours from the time the current copy request is processed and at most 366 days after the `  create_time  ` of the source backup.

Before using any of the command data below, make the following replacements:

  - PROJECT\_ID : the project ID.
  - SOURCE\_INSTANCE\_ID : the source Spanner instance ID.
  - SOURCE\_DATABASE\_ID : the source Spanner database ID.
  - SOURCE\_BACKUP\_NAME : the Spanner backup name.
  - DESTINATION\_INSTANCE\_ID : the target Spanner instance ID.
  - DESTINATION\_BACKUP\_NAME : the destination Spanner backup name.
  - EXPIRATION\_DATE : the expiration date time stamp.
  - ENCRYPTION\_TYPE : the encryption type of backup created. Valid values are `  USE_CONFIG_DEFAULT_OR_BACKUP_ENCRYPTION  ` , `  GOOGLE_DEFAULT_ENCRYPTION  ` , or `  CUSTOMER_MANAGED_ENCRYPTION  ` . If you use `  CUSTOMER_MANAGED_ENCRYPTION  ` , you must specify a `  kmsKeyName  ` .

Execute the following command:

#### Linux, macOS, or Cloud Shell

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups copy \
--source-instance=SOURCE_INSTANCE_ID \
--source-backup=SOURCE_BACKUP_NAME \
--destination-instance=DESTINATION_INSTANCE_ID \
--destination-backup=DESTINATION_BACKUP_NAME \
--expiration-date=EXPIRATION_DATE \
--encryption-type=ENCRYPTION_TYPE
```

#### Windows (PowerShell)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups copy `
--source-instance=SOURCE_INSTANCE_ID `
--source-backup=SOURCE_BACKUP_NAME `
--destination-instance=DESTINATION_INSTANCE_ID `
--destination-backup=DESTINATION_BACKUP_NAME `
--expiration-date=EXPIRATION_DATE `
--encryption-type=ENCRYPTION_TYPE
```

#### Windows (cmd.exe)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups copy ^
--source-instance=SOURCE_INSTANCE_ID ^
--source-backup=SOURCE_BACKUP_NAME ^
--destination-instance=DESTINATION_INSTANCE_ID ^
--destination-backup=DESTINATION_BACKUP_NAME ^
--expiration-date=EXPIRATION_DATE ^
--encryption-type=ENCRYPTION_TYPE
```

You should receive a response similar to the following:

``` text
createTime: '2022-03-29T22:06:05.905823Z'
database: projects/PROJECT_ID/instances/SOURCE_INSTANCE_ID/databases/SOURCE_DATABASE_ID
databaseDialect: GOOGLE_STANDARD_SQL
encryptionInfo:
encryptionType: GOOGLE_DEFAULT_ENCRYPTION
expireTime: '2022-03-30T10:49:41Z'
maxExpireTime: '2023-03-17T20:46:33.479336Z'
name: projects/PROJECT_ID/instances/DESTINATION_INSTANCE_ID/backups/DESTINATION_BACKUP_NAME
sizeBytes: '7957667'
state: READY
versionTime: '2022-03-16T20:46:33.479336Z'
```

### Copy a backup in a different project

If you choose to copy the backup to a different project, you must have another project with its own instance ready for the copied backup. You can't create a new project as part of the backup copy operation. Also, the expiration time of the backup must be at least six hours from the time the current copy request is processed and at most 366 days after the source backup `  create_time  ` .

Before using any of the command data below, make the following replacements:

  - SOURCE\_PROJECT\_ID : the source project ID.
  - SOURCE\_INSTANCE\_ID : the source Spanner instance ID.
  - SOURCE\_DATABASE\_ID : the source Spanner database ID.
  - SOURCE\_BACKUP\_NAME : the Spanner backup name.
  - DESTINATION\_PROJECT\_ID : the destination project ID.
  - DESTINATION\_INSTANCE\_ID : the target Spanner instance ID.
  - DESTINATION\_BACKUP\_NAME : the destination Spanner backup name.
  - EXPIRATION\_DATE : the expiration date time stamp.
  - ENCRYPTION\_TYPE : the encryption type of backup created. Valid values are `  USE_CONFIG_DEFAULT_OR_BACKUP_ENCRYPTION  ` , `  GOOGLE_DEFAULT_ENCRYPTION  ` , or `  CUSTOMER_MANAGED_ENCRYPTION  ` . If you use `  CUSTOMER_MANAGED_ENCRYPTION  ` , you must specify a `  kmsKeyName  ` .

Execute the following command:

#### Linux, macOS, or Cloud Shell

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups copy \
--source-backup=projects/SOURCE_PROJECT_ID/instances/SOURCE_INSTANCE_ID/backups/SOURCE_BACKUP_NAME \
--destination-backup=projects/DESTINATION_PROJECT_ID/instances/DESTINATION_INSTANCE_ID/backups/DESTINATION_BACKUP_NAME \
--expiration-date=EXPIRATION_DATE \
--encryption-type=ENCRYPTION_TYPE
```

#### Windows (PowerShell)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups copy `
--source-backup=projects/SOURCE_PROJECT_ID/instances/SOURCE_INSTANCE_ID/backups/SOURCE_BACKUP_NAME `
--destination-backup=projects/DESTINATION_PROJECT_ID/instances/DESTINATION_INSTANCE_ID/backups/DESTINATION_BACKUP_NAME `
--expiration-date=EXPIRATION_DATE `
--encryption-type=ENCRYPTION_TYPE
```

#### Windows (cmd.exe)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups copy ^
--source-backup=projects/SOURCE_PROJECT_ID/instances/SOURCE_INSTANCE_ID/backups/SOURCE_BACKUP_NAME ^
--destination-backup=projects/DESTINATION_PROJECT_ID/instances/DESTINATION_INSTANCE_ID/backups/DESTINATION_BACKUP_NAME ^
--expiration-date=EXPIRATION_DATE ^
--encryption-type=ENCRYPTION_TYPE
```

You should receive a response similar to the following:

``` text
createTime: '2022-03-29T22:06:05.905823Z'
database: projects/SOURCE_PROJECT_ID/instances/SOURCE_INSTANCE_ID/databases/SOURCE_DATABASE_ID
databaseDialect: GOOGLE_STANDARD_SQL
encryptionInfo:
encryptionType: GOOGLE_DEFAULT_ENCRYPTION
expireTime: '2022-03-30T10:49:41Z'
maxExpireTime: '2023-03-17T20:46:33.479336Z'
name: projects/DESTINATION_PROJECT_ID/instances/DESTINATION_INSTANCE_ID/backups/DESTINATION_BACKUP_NAME
sizeBytes: '7957667'
state: READY
versionTime: '2022-03-16T20:46:33.479336Z'
```

To check the progress of a copy operation, see [Check the operation progress](#check-operation-progress) .

### Client libraries

The following code sample copies an existing backup. You can copy the backup into an instance in a different region or project. Once complete, the sample retrieves and prints some information about the newly created copied backup, such as its name, size, backup state, and [`  version_time  `](/spanner/docs/reference/rest/v1/projects.instances.backups#Backup.FIELDS.version_time) .

### C++

``` cpp
void CopyBackup(google::cloud::spanner_admin::DatabaseAdminClient client,
                std::string const& src_project_id,
                std::string const& src_instance_id,
                std::string const& src_backup_id,
                std::string const& dst_project_id,
                std::string const& dst_instance_id,
                std::string const& dst_backup_id,
                google::cloud::spanner::Timestamp expire_time) {
  google::cloud::spanner::Backup source(
      google::cloud::spanner::Instance(src_project_id, src_instance_id),
      src_backup_id);
  google::cloud::spanner::Instance dst_in(dst_project_id, dst_instance_id);
  auto copy_backup =
      client
          .CopyBackup(dst_in.FullName(), dst_backup_id, source.FullName(),
                      expire_time.get<google::protobuf::Timestamp>().value())
          .get();
  if (!copy_backup) throw std::move(copy_backup).status();
  std::cout << "Copy Backup " << copy_backup->name()  //
            << " of " << source.FullName()            //
            << " of size " << copy_backup->size_bytes() << " bytes as of "
            << google::cloud::spanner::MakeTimestamp(
                   copy_backup->version_time())
                   .value()
            << " was created at "
            << google::cloud::spanner::MakeTimestamp(copy_backup->create_time())
                   .value()
            << ".\n";
}
```

### C\#

``` csharp
using Google.Api.Gax;
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using Google.Protobuf.WellKnownTypes;
using System;

public class CopyBackupSample
{
    public Backup CopyBackup(
        string sourceProjectId, string sourceInstanceId, string sourceBackupId,
        string targetProjectId, string targetInstanceId, string targetBackupId,
        DateTimeOffset expireTime)
    {
        DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();

        var request = new CopyBackupRequest
        {
            SourceBackupAsBackupName = new BackupName(sourceProjectId, sourceInstanceId, sourceBackupId),
            ParentAsInstanceName = new InstanceName(targetProjectId, targetInstanceId),
            BackupId = targetBackupId,
            ExpireTime = Timestamp.FromDateTimeOffset(expireTime)
        };

        var response = databaseAdminClient.CopyBackup(request);
        Console.WriteLine("Waiting for the operation to finish.");
        var completedResponse = response.PollUntilCompleted(new PollSettings(Expiration.FromTimeout(TimeSpan.FromMinutes(15)), TimeSpan.FromMinutes(2)));

        if (completedResponse.IsFaulted)
        {
            Console.WriteLine($"Error while creating backup: {completedResponse.Exception}");
            throw completedResponse.Exception;
        }

        Backup backup = completedResponse.Result;

        Console.WriteLine($"Backup created successfully.");
        Console.WriteLine($"Backup with Id {sourceBackupId} has been copied from {sourceProjectId}/{sourceInstanceId} to {targetProjectId}/{targetInstanceId} Backup {targetBackupId}");
        Console.WriteLine($"Backup {backup.Name} of size {backup.SizeBytes} bytes was created at {backup.CreateTime} from {backup.Database} and is in state {backup.State} and has version time {backup.VersionTime}");

        return backup;
    }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"
 "time"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
 pbt "github.com/golang/protobuf/ptypes/timestamp"
)

// copyBackup copies an existing backup to a given instance in same or different region, or in same or different project.
func copyBackup(w io.Writer, instancePath string, copyBackupId string, sourceBackupPath string) error {
 // instancePath := "projects/my-project/instances/my-instance"
 // copyBackupId := "my-copy-backup"
 // sourceBackupPath := "projects/my-project/instances/my-instance/backups/my-source-backup"

 // Add timeout to context.
 ctx, cancel := context.WithTimeout(context.Background(), time.Hour)
 defer cancel()

 // Instantiate database admin client.
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return fmt.Errorf("database.NewDatabaseAdminClient: %w", err)
 }
 defer adminClient.Close()

 expireTime := time.Now().AddDate(0, 0, 14)

 // Instantiate the request for performing copy backup operation.
 copyBackupReq := adminpb.CopyBackupRequest{
     Parent:       instancePath,
     BackupId:     copyBackupId,
     SourceBackup: sourceBackupPath,
     ExpireTime:   &pbt.Timestamp{Seconds: expireTime.Unix(), Nanos: int32(expireTime.Nanosecond())},
 }

 // Start copying the backup.
 copyBackupOp, err := adminClient.CopyBackup(ctx, &copyBackupReq)
 if err != nil {
     return fmt.Errorf("adminClient.CopyBackup: %w", err)
 }

 // Wait for copy backup operation to complete.
 fmt.Fprintf(w, "Waiting for backup copy %s/backups/%s to complete...\n", instancePath, copyBackupId)
 copyBackup, err := copyBackupOp.Wait(ctx)
 if err != nil {
     return fmt.Errorf("copyBackup.Wait: %w", err)
 }

 // Check if long-running copyBackup operation is completed.
 if !copyBackupOp.Done() {
     return fmt.Errorf("backup %v could not be copied to %v", sourceBackupPath, copyBackupId)
 }

 // Get the name, create time, version time and backup size.
 copyBackupCreateTime := time.Unix(copyBackup.CreateTime.Seconds, int64(copyBackup.CreateTime.Nanos))
 copyBackupVersionTime := time.Unix(copyBackup.VersionTime.Seconds, int64(copyBackup.VersionTime.Nanos))
 fmt.Fprintf(w,
     "Backup %s of size %d bytes was created at %s with version time %s\n",
     copyBackup.Name,
     copyBackup.SizeBytes,
     copyBackupCreateTime.Format(time.RFC3339),
     copyBackupVersionTime.Format(time.RFC3339))

 return nil
}
```

### Java

``` java
import com.google.cloud.Timestamp;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerException;
import com.google.cloud.spanner.SpannerExceptionFactory;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.spanner.admin.database.v1.Backup;
import com.google.spanner.admin.database.v1.BackupName;
import com.google.spanner.admin.database.v1.InstanceName;
import java.time.Instant;
import java.time.OffsetDateTime;
import java.time.ZoneId;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

public class CopyBackupSample {

  static void copyBackup() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String sourceBackupId = "my-backup";
    String destinationBackupId = "my-destination-backup";

    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {
      copyBackup(databaseAdminClient, projectId, instanceId, sourceBackupId, destinationBackupId);
    }
  }

  static void copyBackup(
      DatabaseAdminClient databaseAdminClient,
      String projectId,
      String instanceId,
      String sourceBackupId,
      String destinationBackupId) {

    Timestamp expireTime =
        Timestamp.ofTimeMicroseconds(
            TimeUnit.MICROSECONDS.convert(
                System.currentTimeMillis() + TimeUnit.DAYS.toMillis(14),
                TimeUnit.MILLISECONDS));

    // Initiate the request which returns an OperationFuture.
    System.out.println("Copying backup [" + destinationBackupId + "]...");
    Backup destinationBackup;
    try {
      // Creates a copy of an existing backup.
      // Wait for the backup operation to complete.
      destinationBackup = databaseAdminClient.copyBackupAsync(
          InstanceName.of(projectId, instanceId), destinationBackupId,
          BackupName.of(projectId, instanceId, sourceBackupId), expireTime.toProto()).get();
      System.out.println("Copied backup [" + destinationBackup.getName() + "]");
    } catch (ExecutionException e) {
      throw (SpannerException) e.getCause();
    } catch (InterruptedException e) {
      throw SpannerExceptionFactory.propagateInterrupt(e);
    }
    // Load the metadata of the new backup from the server.
    destinationBackup = databaseAdminClient.getBackup(destinationBackup.getName());
    System.out.println(
        String.format(
            "Backup %s of size %d bytes was copied at %s for version of database at %s",
            destinationBackup.getName(),
            destinationBackup.getSizeBytes(),
            OffsetDateTime.ofInstant(
                Instant.ofEpochSecond(destinationBackup.getCreateTime().getSeconds(),
                    destinationBackup.getCreateTime().getNanos()),
                ZoneId.systemDefault()),
            OffsetDateTime.ofInstant(
                Instant.ofEpochSecond(destinationBackup.getVersionTime().getSeconds(),
                    destinationBackup.getVersionTime().getNanos()),
                ZoneId.systemDefault())));
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const instanceId = 'my-instance';
// const backupId = 'my-backup',
// const sourceBackupPath = 'projects/my-project-id/instances/my-source-instance/backups/my-source-backup',
// const projectId = 'my-project-id';

// Imports the Google Cloud Spanner client library
const {Spanner} = require('@google-cloud/spanner');
const {PreciseDate} = require('@google-cloud/precise-date');

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner Database Admin Client object
const databaseAdminClient = spanner.getDatabaseAdminClient();

async function spannerCopyBackup() {
  // Expire copy backup 14 days in the future
  const expireTime = Spanner.timestamp(
    Date.now() + 1000 * 60 * 60 * 24 * 14,
  ).toStruct();

  // Copy the source backup
  try {
    console.log(`Creating copy of the source backup ${sourceBackupPath}.`);
    const [operation] = await databaseAdminClient.copyBackup({
      parent: databaseAdminClient.instancePath(projectId, instanceId),
      sourceBackup: sourceBackupPath,
      backupId: backupId,
      expireTime: expireTime,
    });

    console.log(
      `Waiting for backup copy ${databaseAdminClient.backupPath(
        projectId,
        instanceId,
        backupId,
      )} to complete...`,
    );
    await operation.promise();

    // Verify the copy backup is ready
    const [copyBackup] = await databaseAdminClient.getBackup({
      name: databaseAdminClient.backupPath(projectId, instanceId, backupId),
    });

    if (copyBackup.state === 'READY') {
      console.log(
        `Backup copy ${copyBackup.name} of size ` +
          `${copyBackup.sizeBytes} bytes was created at ` +
          `${new PreciseDate(copyBackup.createTime).toISOString()} ` +
          'with version time ' +
          `${new PreciseDate(copyBackup.versionTime).toISOString()}`,
      );
    } else {
      console.error('ERROR: Copy of backup is not ready.');
    }
  } catch (err) {
    console.error('ERROR:', err);
  }
}
spannerCopyBackup();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\CopyBackupRequest;
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Protobuf\Timestamp;

/**
 * Create a copy backup from another source backup.
 * Example:
 * ```
 * copy_backup($projectId, $destInstanceId, $destBackupId, $sourceInstanceId, $sourceBackupId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $destInstanceId The Spanner instance ID where the copy backup will reside.
 * @param string $destBackupId The Spanner backup ID of the new backup to be created.
 * @param string $sourceInstanceId The Spanner instance ID of the source backup.
 * @param string $sourceBackupId The Spanner backup ID of the source.
 */
function copy_backup(
    string $projectId,
    string $destInstanceId,
    string $destBackupId,
    string $sourceInstanceId,
    string $sourceBackupId
): void {
    $databaseAdminClient = new DatabaseAdminClient();

    $destInstanceFullName = DatabaseAdminClient::instanceName($projectId, $destInstanceId);
    $expireTime = new Timestamp();
    $expireTime->setSeconds((new \DateTime('+8 hours'))->getTimestamp());
    $sourceBackupFullName = DatabaseAdminClient::backupName($projectId, $sourceInstanceId, $sourceBackupId);
    $request = new CopyBackupRequest([
        'source_backup' => $sourceBackupFullName,
        'parent' => $destInstanceFullName,
        'backup_id' => $destBackupId,
        'expire_time' => $expireTime
    ]);

    $operationResponse = $databaseAdminClient->copyBackup($request);
    $operationResponse->pollUntilComplete();

    if ($operationResponse->operationSucceeded()) {
        $destBackupInfo = $operationResponse->getResult();
        printf(
            'Backup %s of size %d bytes was copied at %d from the source backup %s' . PHP_EOL,
            basename($destBackupInfo->getName()),
            $destBackupInfo->getSizeBytes(),
            $destBackupInfo->getCreateTime()->getSeconds(),
            $sourceBackupId
        );
        printf('Version time of the copied backup: %d' . PHP_EOL, $destBackupInfo->getVersionTime()->getSeconds());
    } else {
        $error = $operationResponse->getError();
        printf('Backup not created due to error: %s.' . PHP_EOL, $error->getMessage());
    }
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def copy_backup(instance_id, backup_id, source_backup_path):
    """Copies a backup."""

    from google.cloud.spanner_admin_database_v1.types import backup as backup_pb

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    # Create a backup object and wait for copy backup operation to complete.
    expire_time = datetime.utcnow() + timedelta(days=14)
    request = backup_pb.CopyBackupRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        backup_id=backup_id,
        source_backup=source_backup_path,
        expire_time=expire_time,
    )

    operation = database_admin_api.copy_backup(request)

    # Wait for backup operation to complete.
    copy_backup = operation.result(2100)

    # Verify that the copy backup is ready.
    assert copy_backup.state == backup_pb.Backup.State.READY

    print(
        "Backup {} of size {} bytes was created at {} with version time {}".format(
            copy_backup.name,
            copy_backup.size_bytes,
            copy_backup.create_time,
            copy_backup.version_time,
        )
    )
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "The ID of the destination instance that will contain the backup copy"
# backup_id = "The ID of the backup copy"
# source_backup = "The source backup to be copied"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin

instance_path = database_admin_client.instance_path project: project_id, instance: instance_id
backup_path = database_admin_client.backup_path project: project_id,
                                                instance: instance_id,
                                                backup: backup_id
source_backup = database_admin_client.backup_path project: project_id,
                                                  instance: instance_id,
                                                  backup: source_backup_id

expire_time = Time.now + (14 * 24 * 3600) # 14 days from now

job = database_admin_client.copy_backup parent: instance_path,
                                        backup_id: backup_id,
                                        source_backup: source_backup,
                                        expire_time: expire_time

puts "Copy backup operation in progress"

job.wait_until_done!

backup = database_admin_client.get_backup name: backup_path
puts "Backup #{backup_id} of size #{backup.size_bytes} bytes was copied at #{backup.create_time} from #{source_backup} for version #{backup.version_time}"
```

## Check the operation progress

### Console

1.  In the Google Cloud console, go to the Spanner **Instances** page.

2.  Click the instance that contains the database that you want to view the backup operation for.

3.  Click the database.

4.  In the navigation pane, click **Operations** . The **Operations** page shows a list of running operations.

### gcloud

Use [`  gcloud spanner operations describe  `](/sdk/gcloud/reference/spanner/operations/describe) to check the progress of an operation.

1.  Get the operation ID:
    
    Before using any of the command data below, make the following replacements:
    
      - INSTANCE\_NAME : the Spanner instance name.
      - DATABASE\_NAME : the Spanner database name.
    
    Execute the following command:
    
    #### Linux, macOS, or Cloud Shell
    
    **Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .
    
    ``` text
    gcloud spanner operations list --instance=INSTANCE_NAME \
    --database=DATABASE_NAME --type=backup
    ```
    
    #### Windows (PowerShell)
    
    **Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .
    
    ``` text
    gcloud spanner operations list --instance=INSTANCE_NAME `
    --database=DATABASE_NAME --type=backup
    ```
    
    #### Windows (cmd.exe)
    
    **Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .
    
    ``` text
    gcloud spanner operations list --instance=INSTANCE_NAME ^
    --database=DATABASE_NAME --type=backup
    ```
    
    You should receive a response similar to the following:
    
    ``` text
    OPERATION_ID     DONE  @TYPE                 BACKUP          SOURCE_DATABASE       START_TIME                   END_TIME
    _auto_op_123456  True  CreateBackupMetadata  example-db-backup-7  example-db       2020-02-04T02:12:38.075515Z  2020-02-04T02:22:40.581170Z
    _auto_op_234567  True  CreateBackupMetadata  example-db-backup-6  example-db       2020-02-04T02:05:43.920377Z  2020-02-04T02:07:59.089820Z
    ```
    
    Usage notes:
    
      - To limit the list, specify the `  --filter  ` flag. For example:
        
          - `  --filter="metadata.name:example-db"  ` only lists the operations on a specific database.
          - `  --filter="error:*"  ` only lists the backup operations that failed.
        
        For information on filter syntax, see [`  gcloud topic filters  `](/sdk/gcloud/reference/topic/filters) . For information on filtering backup operations, see the `  filter  ` field in [`  ListBackupOperationsRequest  `](/spanner/docs/reference/rpc/google.spanner.admin.database.v1#listbackupoperationsrequest) .
    
      - The `  --type  ` flag isn't case sensitive.

2.  Run `  gcloud spanner operations describe  ` :
    
    Before using any of the command data below, make the following replacements:
    
      - OPERATION\_ID : the operation ID.
      - INSTANCE\_NAME : the Spanner instance name.
      - DATABASE\_NAME : the Spanner database name.
    
    Execute the following command:
    
    #### Linux, macOS, or Cloud Shell
    
    **Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .
    
    ``` text
    gcloud spanner operations describe OPERATION_ID \
    --instance=INSTANCE_NAME \
    --backup=BACKUP_NAME \
    ```
    
    #### Windows (PowerShell)
    
    **Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .
    
    ``` text
    gcloud spanner operations describe OPERATION_ID `
    --instance=INSTANCE_NAME `
    --backup=BACKUP_NAME `
    ```
    
    #### Windows (cmd.exe)
    
    **Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .
    
    ``` text
    gcloud spanner operations describe OPERATION_ID ^
    --instance=INSTANCE_NAME ^
    --backup=BACKUP_NAME ^
    ```
    
    You should receive a response similar to the following:
    
    ``` text
    done: true
    metadata:
    ...
    progress:
    - endTime: '2022-03-01T00:28:06.691403Z'
      progressPercent: 100
      startTime: '2022-03-01T00:28:04.221401Z'
    - endTime: '2022-03-01T00:28:17.624588Z'
      startTime: '2022-03-01T00:28:06.691403Z'
      progressPercent: 100
    ...
    ```
    
    The `  progress  ` section in the output shows the percentage of the operation that's complete.
    
    If the operation takes too long, you can cancel it. For more information, see [Cancel a long-running backup operation](/spanner/docs/manage-and-observe-long-running-operations#cancel_a_long-running_backup_operation) .

### Client libraries

The following code sample lists all of the ongoing operations for creating backups (operations with [`  CreateBackupMetadata  `](/spanner/docs/reference/rest/v1/CreateBackupMetadata) ) and copying backups (operations with [`  CopyBackupMetadata  `](/spanner/docs/reference/rest/v1/CopyBackupMetadata) ) filtered by a given database.

For information on filtering syntax, see the `  filter  ` parameter in [`  backupOperations.list  `](/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list#query-parameters) .

### C++

``` cpp
void ListBackupOperations(
    google::cloud::spanner_admin::DatabaseAdminClient client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id, std::string const& backup_id) {
  google::cloud::spanner::Instance in(project_id, instance_id);
  google::cloud::spanner::Database database(in, database_id);
  google::cloud::spanner::Backup backup(in, backup_id);

  google::spanner::admin::database::v1::ListBackupOperationsRequest request;
  request.set_parent(in.FullName());

  request.set_filter(std::string("(metadata.@type=type.googleapis.com/") +
                     "google.spanner.admin.database.v1.CreateBackupMetadata)" +
                     " AND (metadata.database=" + database.FullName() + ")");
  for (auto& operation : client.ListBackupOperations(request)) {
    if (!operation) throw std::move(operation).status();
    google::spanner::admin::database::v1::CreateBackupMetadata metadata;
    operation->metadata().UnpackTo(&metadata);
    std::cout << "Backup " << metadata.name() << " of database "
              << metadata.database() << " is "
              << metadata.progress().progress_percent() << "% complete.\n";
  }

  request.set_filter(std::string("(metadata.@type:type.googleapis.com/") +
                     "google.spanner.admin.database.v1.CopyBackupMetadata)" +
                     " AND (metadata.source_backup=" + backup.FullName() + ")");
  for (auto& operation : client.ListBackupOperations(request)) {
    if (!operation) throw std::move(operation).status();
    google::spanner::admin::database::v1::CopyBackupMetadata metadata;
    operation->metadata().UnpackTo(&metadata);
    std::cout << "Copy " << metadata.name() << " of backup "
              << metadata.source_backup() << " is "
              << metadata.progress().progress_percent() << "% complete.\n";
  }
}
```

### C\#

To list all the create backup operations:

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using Google.LongRunning;
using System;
using System.Collections.Generic;

public class ListBackupOperationsSample
{
    public IEnumerable<Operation> ListBackupOperations(string projectId, string instanceId, string databaseId)
    {
        // Create the DatabaseAdminClient instance.
        DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();

        var filter = $"(metadata.@type:type.googleapis.com/google.spanner.admin.database.v1.CreateBackupMetadata) AND (metadata.database:{databaseId})";
        ListBackupOperationsRequest request = new ListBackupOperationsRequest
        {
            ParentAsInstanceName = InstanceName.FromProjectInstance(projectId, instanceId),
            Filter = filter
        };

        // List the create backup operations on the database.
        var backupOperations = databaseAdminClient.ListBackupOperations(request);

        foreach (var operation in backupOperations)
        {
            CreateBackupMetadata metadata = operation.Metadata.Unpack<CreateBackupMetadata>();
            Console.WriteLine($"Backup {metadata.Name} on " + $"database {metadata.Database} is " + $"{metadata.Progress.ProgressPercent}% complete");
        }

        return backupOperations;
    }
}
```

To list all the copy backup operations:

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using Google.LongRunning;
using System;
using System.Collections.Generic;

public class ListCopyBackupOperationsSample
{
    public IEnumerable<Operation> ListCopyBackupOperations(string projectId, string instanceId, string databaseId, string backupId)
    {
        // Create the DatabaseAdminClient instance.
        DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();

        var filter = $"(metadata.@type:type.googleapis.com/google.spanner.admin.database.v1.CopyBackupMetadata) AND (metadata.source_backup:{backupId})";
        ListBackupOperationsRequest request = new ListBackupOperationsRequest
        {
            ParentAsInstanceName = InstanceName.FromProjectInstance(projectId, instanceId),
            Filter = filter
        };

        // List the copy backup operations on the database.
        var backupOperations = databaseAdminClient.ListBackupOperations(request);

        foreach (var operation in backupOperations)
        {
            CopyBackupMetadata metadata = operation.Metadata.Unpack<CopyBackupMetadata>();
            Console.WriteLine($"Backup {metadata.Name} from source backup {metadata.SourceBackup} is {metadata.Progress.ProgressPercent}% complete");
        }

        return backupOperations;
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
 "github.com/golang/protobuf/ptypes"
 "google.golang.org/api/iterator"
)

// listBackupOperations lists the backup operations that are pending or have completed/failed/cancelled within the last 7 days.
func listBackupOperations(w io.Writer, db string, backupId string) error {
 // db := "projects/my-project/instances/my-instance/databases/my-database"
 // backupID := "my-backup"

 ctx := context.Background()

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
 // List the CreateBackup operations.
 filter := fmt.Sprintf("(metadata.@type:type.googleapis.com/google.spanner.admin.database.v1.CreateBackupMetadata) AND (metadata.database:%s)", db)
 iter := adminClient.ListBackupOperations(ctx, &adminpb.ListBackupOperationsRequest{
     Parent: instanceName,
     Filter: filter,
 })
 for {
     resp, err := iter.Next()
     if err == iterator.Done {
         break
     }
     if err != nil {
         return err
     }
     metadata := &adminpb.CreateBackupMetadata{}
     if err := ptypes.UnmarshalAny(resp.Metadata, metadata); err != nil {
         return err
     }
     fmt.Fprintf(w, "Backup %s on database %s is %d%% complete.\n",
         metadata.Name,
         metadata.Database,
         metadata.Progress.ProgressPercent,
     )
 }

 // List the CopyBackup operations.
 filter = fmt.Sprintf("(metadata.@type:type.googleapis.com/google.spanner.admin.database.v1.CopyBackupMetadata) AND (metadata.source_backup:%s)", backupId)
 iter = adminClient.ListBackupOperations(ctx, &adminpb.ListBackupOperationsRequest{
     Parent: instanceName,
     Filter: filter,
 })
 for {
     resp, err := iter.Next()
     if err == iterator.Done {
         break
     }
     if err != nil {
         return err
     }
     metadata := &adminpb.CopyBackupMetadata{}
     if err := ptypes.UnmarshalAny(resp.Metadata, metadata); err != nil {
         return err
     }
     fmt.Fprintf(w, "Backup %s copied from %s is %d%% complete.\n",
         metadata.Name,
         metadata.SourceBackup,
         metadata.Progress.ProgressPercent,
     )
 }

 return nil
}
```

### Java

``` java
static void listBackupOperations(
    DatabaseAdminClient databaseAdminClient,
    String projectId, String instanceId,
    String databaseId, String backupId) {
  InstanceName instanceName = InstanceName.of(projectId, instanceId);
  // Get 'CreateBackup' operations for the sample database.
  String filter =
      String.format(
          "(metadata.@type:type.googleapis.com/"
              + "google.spanner.admin.database.v1.CreateBackupMetadata) "
              + "AND (metadata.database:%s)",
          DatabaseName.of(projectId, instanceId, databaseId).toString());
  ListBackupOperationsRequest listBackupOperationsRequest =
      ListBackupOperationsRequest.newBuilder()
          .setParent(instanceName.toString()).setFilter(filter).build();
  ListBackupOperationsPagedResponse createBackupOperations
      = databaseAdminClient.listBackupOperations(listBackupOperationsRequest);
  System.out.println("Create Backup Operations:");
  for (Operation op : createBackupOperations.iterateAll()) {
    try {
      CreateBackupMetadata metadata = op.getMetadata().unpack(CreateBackupMetadata.class);
      System.out.println(
          String.format(
              "Backup %s on database %s pending: %d%% complete",
              metadata.getName(),
              metadata.getDatabase(),
              metadata.getProgress().getProgressPercent()));
    } catch (InvalidProtocolBufferException e) {
      // The returned operation does not contain CreateBackupMetadata.
      System.err.println(e.getMessage());
    }
  }
  // Get copy backup operations for the sample database.
  filter = String.format(
      "(metadata.@type:type.googleapis.com/"
          + "google.spanner.admin.database.v1.CopyBackupMetadata) "
          + "AND (metadata.source_backup:%s)",
      BackupName.of(projectId, instanceId, backupId).toString());
  listBackupOperationsRequest =
      ListBackupOperationsRequest.newBuilder()
          .setParent(instanceName.toString()).setFilter(filter).build();
  ListBackupOperationsPagedResponse copyBackupOperations =
      databaseAdminClient.listBackupOperations(listBackupOperationsRequest);
  System.out.println("Copy Backup Operations:");
  for (Operation op : copyBackupOperations.iterateAll()) {
    try {
      CopyBackupMetadata copyBackupMetadata =
          op.getMetadata().unpack(CopyBackupMetadata.class);
      System.out.println(
          String.format(
              "Copy Backup %s on backup %s pending: %d%% complete",
              copyBackupMetadata.getName(),
              copyBackupMetadata.getSourceBackup(),
              copyBackupMetadata.getProgress().getProgressPercent()));
    } catch (InvalidProtocolBufferException e) {
      // The returned operation does not contain CopyBackupMetadata.
      System.err.println(e.getMessage());
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
// Imports the Google Cloud client library
const {Spanner, protos} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const databaseId = 'my-database';
// const backupId = 'my-backup';
// const instanceId = 'my-instance';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner Database Admin Client object
const databaseAdminClient = spanner.getDatabaseAdminClient();

// List create backup operations
try {
  const [backupOperations] = await databaseAdminClient.listBackupOperations({
    parent: databaseAdminClient.instancePath(projectId, instanceId),
    filter:
      '(metadata.@type:type.googleapis.com/google.spanner.admin.database.v1.CreateBackupMetadata) ' +
      `AND (metadata.database:${databaseId})`,
  });
  console.log('Create Backup Operations:');
  backupOperations.forEach(backupOperation => {
    const metadata =
      protos.google.spanner.admin.database.v1.CreateBackupMetadata.decode(
        backupOperation.metadata.value,
      );
    console.log(
      `Backup ${metadata.name} on database ${metadata.database} is ` +
        `${metadata.progress.progressPercent}% complete.`,
    );
  });
} catch (err) {
  console.error('ERROR:', err);
}

// List copy backup operations
try {
  console.log(
    '(metadata.@type:type.googleapis.com/google.spanner.admin.database.v1.CopyBackupMetadata) ' +
      `AND (metadata.source_backup:${backupId})`,
  );
  const [backupOperations] = await databaseAdminClient.listBackupOperations({
    parent: databaseAdminClient.instancePath(projectId, instanceId),
    filter:
      '(metadata.@type:type.googleapis.com/google.spanner.admin.database.v1.CopyBackupMetadata) ' +
      `AND (metadata.source_backup:${backupId})`,
  });
  console.log('Copy Backup Operations:');
  backupOperations.forEach(backupOperation => {
    const metadata =
      protos.google.spanner.admin.database.v1.CopyBackupMetadata.decode(
        backupOperation.metadata.value,
      );
    console.log(
      `Backup ${metadata.name} copied from source backup ${metadata.sourceBackup} is ` +
        `${metadata.progress.progressPercent}% complete.`,
    );
  });
} catch (err) {
  console.error('ERROR:', err);
}
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

``` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\CreateBackupMetadata;
use Google\Cloud\Spanner\Admin\Database\V1\CopyBackupMetadata;
use Google\Cloud\Spanner\Admin\Database\V1\ListBackupOperationsRequest;

/**
 * List all create backup operations in an instance.
 * Optionally passing the backupId will also list the
 * copy backup operations on the backup.
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 * @param string $backupId The Spanner backup ID whose copy operations need to be listed.
 */
function list_backup_operations(
    string $projectId,
    string $instanceId,
    string $databaseId,
    string $backupId
): void {
    $databaseAdminClient = new DatabaseAdminClient();

    $parent = DatabaseAdminClient::instanceName($projectId, $instanceId);

    // List the CreateBackup operations.
    $filterCreateBackup = '(metadata.@type:type.googleapis.com/' .
        'google.spanner.admin.database.v1.CreateBackupMetadata) AND ' . "(metadata.database:$databaseId)";

    // See https://cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#listbackupoperationsrequest
    // for the possible filter values
    $filterCopyBackup = sprintf('(metadata.@type:type.googleapis.com/' .
        'google.spanner.admin.database.v1.CopyBackupMetadata) AND ' . "(metadata.source_backup:$backupId)");
    $operations = $databaseAdminClient->listBackupOperations(
        new ListBackupOperationsRequest([
            'parent' => $parent,
            'filter' => $filterCreateBackup
        ])
    );

    foreach ($operations->iterateAllElements() as $operation) {
        $obj = new CreateBackupMetadata();
        $meta = $operation->getMetadata()->unpack($obj);
        $backupName = basename($meta->getName());
        $dbName = basename($meta->getDatabase());
        $progress = $meta->getProgress()->getProgressPercent();
        printf('Backup %s on database %s is %d%% complete.' . PHP_EOL, $backupName, $dbName, $progress);
    }

    $operations = $databaseAdminClient->listBackupOperations(
        new ListBackupOperationsRequest([
            'parent' => $parent,
            'filter' => $filterCopyBackup
        ])
    );

    foreach ($operations->iterateAllElements() as $operation) {
        $obj = new CopyBackupMetadata();
        $meta = $operation->getMetadata()->unpack($obj);
        $backupName = basename($meta->getName());
        $progress = $meta->getProgress()->getProgressPercent();
        printf('Copy Backup %s on source backup %s is %d%% complete.' . PHP_EOL, $backupName, $backupId, $progress);
    }
}
```

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def list_backup_operations(instance_id, database_id, backup_id):
    from google.cloud.spanner_admin_database_v1.types import backup as backup_pb

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    # List the CreateBackup operations.
    filter_ = (
        "(metadata.@type:type.googleapis.com/"
        "google.spanner.admin.database.v1.CreateBackupMetadata) "
        "AND (metadata.database:{})"
    ).format(database_id)
    request = backup_pb.ListBackupOperationsRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        filter=filter_,
    )
    operations = database_admin_api.list_backup_operations(request)
    for op in operations:
        metadata = protobuf_helpers.from_any_pb(
            backup_pb.CreateBackupMetadata, op.metadata
        )
        print(
            "Backup {} on database {}: {}% complete.".format(
                metadata.name, metadata.database, metadata.progress.progress_percent
            )
        )

    # List the CopyBackup operations.
    filter_ = (
        "(metadata.@type:type.googleapis.com/google.spanner.admin.database.v1.CopyBackupMetadata) "
        "AND (metadata.source_backup:{})"
    ).format(backup_id)
    request = backup_pb.ListBackupOperationsRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        filter=filter_,
    )
    operations = database_admin_api.list_backup_operations(request)
    for op in operations:
        metadata = protobuf_helpers.from_any_pb(
            backup_pb.CopyBackupMetadata, op.metadata
        )
        print(
            "Backup {} on source backup {}: {}% complete.".format(
                metadata.name,
                metadata.source_backup,
                metadata.progress.progress_percent,
            )
        )
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

To list all the create backup operations:

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin
instance_path = database_admin_client.instance_path project: project_id, instance: instance_id

jobs = database_admin_client.list_backup_operations parent: instance_path,
                                                    filter: "metadata.@type:type.googleapis.com/google.spanner.admin.database.v1.CreateBackupMetadata"
jobs.each do |job|
  if job.error?
    puts job.error
  else
    puts "Backup #{job.results.name} on database #{database_id} is #{job.metadata.progress.progress_percent}% complete"
  end
end
```

To list all the copy backup operations:

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# backup_id = "You Spanner backup ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin
instance_path = database_admin_client.instance_path project: project_id, instance: instance_id

filter = "(metadata.@type:type.googleapis.com/google.spanner.admin.database.v1.CopyBackupMetadata) AND (metadata.source_backup:#{backup_id})"

jobs = database_admin_client.list_backup_operations parent: instance_path,
                                                    filter: filter
jobs.each do |job|
  if job.error?
    puts job.error
  else
    puts "Backup #{job.results.name} on source backup #{backup_id} is #{job.metadata.progress.progress_percent}% complete"
  end
end
```

## Cancel a backup operation

### Console

The Google Cloud console doesn't support canceling a backup operation. However, you can cancel operations that take too long using the gcloud CLI, REST, or RPC API. For more information, see [Cancel a long-running instance operation](/spanner/docs/manage-and-observe-long-running-operations#cancel_a_long-running_instance_operation) .

### gcloud

1.  Get the operation ID:
    
    Before using any of the command data below, make the following replacements:
    
      - INSTANCE\_NAME : the Spanner instance name.
      - DATABASE\_NAME : the Spanner database name.
    
    Execute the following command:
    
    #### Linux, macOS, or Cloud Shell
    
    **Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .
    
    ``` text
    gcloud spanner operations list --instance=INSTANCE_NAME \
    --database=DATABASE_NAME --type=backup
    ```
    
    #### Windows (PowerShell)
    
    **Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .
    
    ``` text
    gcloud spanner operations list --instance=INSTANCE_NAME `
    --database=DATABASE_NAME --type=backup
    ```
    
    #### Windows (cmd.exe)
    
    **Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .
    
    ``` text
    gcloud spanner operations list --instance=INSTANCE_NAME ^
    --database=DATABASE_NAME --type=backup
    ```
    
    You should receive a response similar to the following:
    
    ``` text
    OPERATION_ID     DONE  @TYPE                 BACKUP          SOURCE_DATABASE       START_TIME                   END_TIME
    _auto_op_123456  True  CreateBackupMetadata  example-db-backup-7  example-db       2020-02-04T02:12:38.075515Z  2020-02-04T02:22:40.581170Z
    _auto_op_234567  True  CreateBackupMetadata  example-db-backup-6  example-db       2020-02-04T02:05:43.920377Z  2020-02-04T02:07:59.089820Z
    ```
    
    Usage notes:
    
      - To limit the list, specify the `  --filter  ` flag. For example:
        
          - `  --filter="metadata.name:example-db"  ` only lists the operations on a specific database.
          - `  --filter="error:*"  ` only lists the backup operations that failed.
        
        For information on filter syntax, see [`  gcloud topic filters  `](/sdk/gcloud/reference/topic/filters) . For information on filtering backup operations, see the `  filter  ` field in [`  ListBackupOperationsRequest  `](/spanner/docs/reference/rpc/google.spanner.admin.database.v1#listbackupoperationsrequest) .
    
      - The `  --type  ` flag isn't case sensitive.

2.  Use [`  gcloud spanner operations cancel  `](/sdk/gcloud/reference/spanner/operations/cancel) to cancel a backup operation.
    
    Before using any of the command data below, make the following replacements:
    
      - OPERATION\_ID : the operation ID.
      - INSTANCE\_NAME : the Spanner instance name.
      - DATABASE\_NAME : the Spanner database name.
      - BACKUP\_NAME : the Spanner backup name.
    
    Execute the following command:
    
    #### Linux, macOS, or Cloud Shell
    
    **Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .
    
    ``` text
    gcloud spanner operations cancel OPERATION_ID --instance=INSTANCE_NAME \
     --database=DATABASE_NAME --backup=BACKUP_NAME
    ```
    
    #### Windows (PowerShell)
    
    **Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .
    
    ``` text
    gcloud spanner operations cancel OPERATION_ID --instance=INSTANCE_NAME `
     --database=DATABASE_NAME --backup=BACKUP_NAME
    ```
    
    #### Windows (cmd.exe)
    
    **Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .
    
    ``` text
    gcloud spanner operations cancel OPERATION_ID --instance=INSTANCE_NAME ^
     --database=DATABASE_NAME --backup=BACKUP_NAME
    ```

### Client libraries

The following code sample creates a backup, cancels the backup operation, and then waits until the backup operation is `  done  ` . If the operation was successfully cancelled, it returns the `  cancelTime  ` and an error message. If the backup operation was completed before it was cancelled, the backup exists, and you can delete it.

### C++

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

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

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

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

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

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

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

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

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

## Get backup information

### Console

1.  In the Google Cloud console, go to the Spanner **Instances** page.

2.  Click the instance that contains the database you want to view the backup information for.

3.  Click the database to open its **Overview** page.

4.  In the navigation pane, click **Backup/Restore** . You can view backup information for your selected backup in the database.

### gcloud

To get information about a backup, use [`  gcloud spanner backups describe  `](/sdk/gcloud/reference/spanner/backups/describe) .

Before using any of the command data below, make the following replacements:

  - PROJECT\_ID : the project ID.
  - INSTANCE\_ID : the Spanner instance ID.
  - DATABASE\_ID : the Spanner database ID.
  - BACKUP\_NAME : the Spanner backup name.

Execute the following command:

#### Linux, macOS, or Cloud Shell

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups describe BACKUP_NAME --instance=INSTANCE_ID
```

#### Windows (PowerShell)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups describe BACKUP_NAME --instance=INSTANCE_ID
```

#### Windows (cmd.exe)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups describe BACKUP_NAME --instance=INSTANCE_ID
```

You should receive a response similar to the following:

``` text
createTime: '2020-02-04T02:05:43.920377Z'
database: projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID
expireTime: '2021-02-04T02:05:43.268327Z'
name: projects/PROJECT_ID/instances/INSTANCE_ID/backups/BACKUP_NAME
sizeBytes: '1000000000'
state: READY
```

### Client libraries

Client libraries don't support getting backup information for a single backup. However, you can list all backups and their information in an instance. For more information, see [List backups in an instance](#list-backups-in-instance) .

## List backups in an instance

### Console

1.  In the Google Cloud console, go to the Spanner **Instances** page.

2.  Click your instance to view all available backups and their information.

3.  In the navigation pane, click **Backup/Restore** .

### gcloud

To list all the backups in an instance, use [`  gcloud spanner backups list  `](/sdk/gcloud/reference/spanner/backups/list) .

Before using any of the command data below, make the following replacements:

  - INSTANCE\_ID : the Spanner instance ID.

Execute the following command:

#### Linux, macOS, or Cloud Shell

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups list --instance=INSTANCE_ID
```

#### Windows (PowerShell)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups list --instance=INSTANCE_ID
```

#### Windows (cmd.exe)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups list --instance=INSTANCE_ID
```

You should receive a response similar to the following:

``` text
  BACKUP               SOURCE_DATABASE  CREATION_TIME                EXPIRATION_TIME              STATE  BACKUP_SIZE_IN_BYTES  IN_USE_BY
  example-db-backup-6  example-db       2020-02-04T02:05:43.920377Z  2021-02-04T02:05:43.268327Z  CREATING
  example-db-backup-4  example-db       2020-02-04T01:21:20.873839Z  2021-02-04T01:21:20.530151Z  READY  32
  example-db-backup-3  example-db       2020-02-03T23:59:18.936433Z  2021-02-03T23:59:18.203083Z  READY  32
  example-db-backup-5  example-db       2020-02-03T23:48:06.259296Z  2021-02-03T23:48:05.830937Z  READY  32
  example-db-backup-2  example-db       2020-01-30T19:49:00.616338Z  2021-01-30T19:49:00.283917Z  READY  32
  example-db-backup-1  example-db       2020-01-30T19:47:09.492551Z  2021-01-30T19:47:09.097804Z  READY  32
```

To limit the list, specify the `  --filter  ` flag. For example, to filter the list to only include the backups that are still being created, add `  --filter="state:creating"  ` . For information about filter syntax, see [`  gcloud topic filters  `](/sdk/gcloud/reference/topic/filters) . For information on filtering backups, see the `  filter  ` field in [`  ListBackupsRequest  `](/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.ListBackupsRequest) .

### Client libraries

The following code sample lists the backups in a given instance.

You can filter the list of returned backups (for example, filtering by name, version time, or backup expiration time) by providing a filter expression. For information on filtering syntax, see the `  filter  ` parameter in [List Backups](/spanner/docs/reference/rest/v1/projects.instances.backups/list#query-parameters) .

### C++

``` cpp
void ListBackups(google::cloud::spanner_admin::DatabaseAdminClient client,
                 std::string const& project_id,
                 std::string const& instance_id) {
  google::cloud::spanner::Instance in(project_id, instance_id);
  std::cout << "All backups:\n";
  for (auto& backup : client.ListBackups(in.FullName())) {
    if (!backup) throw std::move(backup).status();
    std::cout << "Backup " << backup->name() << " on database "
              << backup->database() << " with size : " << backup->size_bytes()
              << " bytes.\n";
  }
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using System;
using System.Collections.Generic;
using System.Linq;

public class ListBackupsSample
{
    public IEnumerable<Backup> ListBackups(string projectId, string instanceId, string databaseId, string backupId)
    {
        // Create the DatabaseAdminClient instance.
        DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();

        InstanceName parentAsInstanceName = InstanceName.FromProjectInstance(projectId, instanceId);

        // List all backups.
        Console.WriteLine("All backups:");
        var allBackups = databaseAdminClient.ListBackups(parentAsInstanceName);
        PrintBackups(allBackups);

        ListBackupsRequest request = new ListBackupsRequest
        {
            ParentAsInstanceName = parentAsInstanceName,
        };

        // List backups containing backup name.
        Console.WriteLine($"Backups with backup name containing {backupId}:");
        request.Filter = $"name:{backupId}";
        var backupsWithName = databaseAdminClient.ListBackups(request);
        PrintBackups(backupsWithName);

        // List backups on a database containing name.
        Console.WriteLine($"Backups with database name containing {databaseId}:");
        request.Filter = $"database:{databaseId}";
        var backupsWithDatabaseName = databaseAdminClient.ListBackups(request);
        PrintBackups(backupsWithDatabaseName);

        // List backups that expire within 30 days.
        Console.WriteLine("Backups expiring within 30 days:");
        string expireTime = DateTime.UtcNow.AddDays(30).ToString("O");
        request.Filter = $"expire_time < \"{expireTime}\"";
        var expiringBackups = databaseAdminClient.ListBackups(request);
        PrintBackups(expiringBackups);

        // List backups with a size greater than 100 bytes.
        Console.WriteLine("Backups with size > 100 bytes:");
        request.Filter = "size_bytes > 100";
        var backupsWithSize = databaseAdminClient.ListBackups(request);
        PrintBackups(backupsWithSize);

        // List backups created in the last day that are ready.
        Console.WriteLine("Backups created within last day that are ready:");
        string createTime = DateTime.UtcNow.AddDays(-1).ToString("O");
        request.Filter = $"create_time >= \"{createTime}\" AND state:READY";
        var recentReadyBackups = databaseAdminClient.ListBackups(request);
        PrintBackups(recentReadyBackups);

        // List backups in pages of 500 elements each
        foreach (var page in databaseAdminClient.ListBackups(parentAsInstanceName, pageSize: 500).AsRawResponses())
        {
            PrintBackups(page);
        }

        return allBackups;
    }

    private static void PrintBackups(IEnumerable<Backup> backups)
    {
        // We print the first 5 elements each time for demonstration purposes.
        // You can print all backups in the sequence by removing the call to Take(5).
        // If the sequence has been returned by a paginated operation it will lazily
        // fetch elements in pages as needed.
        foreach (Backup backup in backups.Take(5))
        {
            Console.WriteLine($"Backup Name : {backup.Name}");
        };
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
 "time"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
 "google.golang.org/api/iterator"
)

func listBackups(ctx context.Context, w io.Writer, db, backupID string) error {
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

 printBackups := func(iter *database.BackupIterator) error {
     for {
         resp, err := iter.Next()
         if err == iterator.Done {
             return nil
         }
         if err != nil {
             return err
         }
         fmt.Fprintf(w, "Backup %s\n", resp.Name)
     }
 }

 var iter *database.BackupIterator
 var filter string
 // List all backups.
 iter = adminClient.ListBackups(ctx, &adminpb.ListBackupsRequest{
     Parent: instanceName,
 })
 if err := printBackups(iter); err != nil {
     return err
 }

 // List all backups that contain a name.
 iter = adminClient.ListBackups(ctx, &adminpb.ListBackupsRequest{
     Parent: instanceName,
     Filter: "name:" + backupID,
 })
 if err := printBackups(iter); err != nil {
     return err
 }

 // List all backups that expire before a timestamp.
 expireTime := time.Now().AddDate(0, 0, 30)
 filter = fmt.Sprintf(`expire_time < "%s"`, expireTime.Format(time.RFC3339))
 iter = adminClient.ListBackups(ctx, &adminpb.ListBackupsRequest{
     Parent: instanceName,
     Filter: filter,
 })
 if err := printBackups(iter); err != nil {
     return err
 }

 // List all backups for a database that contains a name.
 iter = adminClient.ListBackups(ctx, &adminpb.ListBackupsRequest{
     Parent: instanceName,
     Filter: "database:" + db,
 })
 if err := printBackups(iter); err != nil {
     return err
 }

 // List all backups with a size greater than some bytes.
 iter = adminClient.ListBackups(ctx, &adminpb.ListBackupsRequest{
     Parent: instanceName,
     Filter: "size_bytes > 100",
 })
 if err := printBackups(iter); err != nil {
     return err
 }

 // List backups that were created after a timestamp that are also ready.
 createTime := time.Now().AddDate(0, 0, -1)
 filter = fmt.Sprintf(
     `create_time >= "%s" AND state:READY`,
     createTime.Format(time.RFC3339),
 )
 iter = adminClient.ListBackups(ctx, &adminpb.ListBackupsRequest{
     Parent: instanceName,
     Filter: filter,
 })
 if err := printBackups(iter); err != nil {
     return err
 }

 // List backups with pagination.
 request := &adminpb.ListBackupsRequest{
     Parent:   instanceName,
     PageSize: 10,
 }
 for {
     iter = adminClient.ListBackups(ctx, request)
     if err := printBackups(iter); err != nil {
         return err
     }
     pageToken := iter.PageInfo().Token
     if pageToken == "" {
         break
     } else {
         request.PageToken = pageToken
     }
 }

 fmt.Fprintf(w, "Backups listed.\n")
 return nil
}
```

### Java

``` java
static void listBackups(
    DatabaseAdminClient dbAdminClient, String projectId,
    String instanceId, String databaseId, String backupId) {
  InstanceName instanceName = InstanceName.of(projectId, instanceId);
  // List all backups.
  System.out.println("All backups:");
  for (Backup backup : dbAdminClient.listBackups(
      instanceName.toString()).iterateAll()) {
    System.out.println(backup);
  }

  // List all backups with a specific name.
  System.out.println(
      String.format("All backups with backup name containing \"%s\":", backupId));
  ListBackupsRequest listBackupsRequest =
      ListBackupsRequest.newBuilder().setParent(instanceName.toString())
          .setFilter(String.format("name:%s", backupId)).build();
  for (Backup backup : dbAdminClient.listBackups(listBackupsRequest).iterateAll()) {
    System.out.println(backup);
  }

  // List all backups for databases whose name contains a certain text.
  System.out.println(
      String.format(
          "All backups for databases with a name containing \"%s\":", databaseId));
  listBackupsRequest =
      ListBackupsRequest.newBuilder().setParent(instanceName.toString())
          .setFilter(String.format("database:%s", databaseId)).build();
  for (Backup backup : dbAdminClient.listBackups(listBackupsRequest).iterateAll()) {
    System.out.println(backup);
  }

  // List all backups that expire before a certain time.
  com.google.cloud.Timestamp expireTime = com.google.cloud.Timestamp.ofTimeMicroseconds(
      TimeUnit.MICROSECONDS.convert(
          System.currentTimeMillis() + TimeUnit.DAYS.toMillis(30), TimeUnit.MILLISECONDS));

  System.out.println(String.format("All backups that expire before %s:", expireTime));
  listBackupsRequest =
      ListBackupsRequest.newBuilder().setParent(instanceName.toString())
          .setFilter(String.format("expire_time < \"%s\"", expireTime)).build();

  for (Backup backup : dbAdminClient.listBackups(listBackupsRequest).iterateAll()) {
    System.out.println(backup);
  }

  // List all backups with size greater than a certain number of bytes.
  listBackupsRequest =
      ListBackupsRequest.newBuilder().setParent(instanceName.toString())
          .setFilter("size_bytes > 100").build();

  System.out.println("All backups with size greater than 100 bytes:");
  for (Backup backup : dbAdminClient.listBackups(listBackupsRequest).iterateAll()) {
    System.out.println(backup);
  }

  // List all backups with a create time after a certain timestamp and that are also ready.
  com.google.cloud.Timestamp createTime = com.google.cloud.Timestamp.ofTimeMicroseconds(
      TimeUnit.MICROSECONDS.convert(
          System.currentTimeMillis() - TimeUnit.DAYS.toMillis(1), TimeUnit.MILLISECONDS));

  System.out.println(
      String.format(
          "All databases created after %s and that are ready:", createTime.toString()));
  listBackupsRequest =
      ListBackupsRequest.newBuilder().setParent(instanceName.toString())
          .setFilter(String.format(
              "create_time >= \"%s\" AND state:READY", createTime.toString())).build();
  for (Backup backup : dbAdminClient.listBackups(listBackupsRequest).iterateAll()) {
    System.out.println(backup);
  }

  // List backups using pagination.
  System.out.println("All backups, listed using pagination:");
  listBackupsRequest =
      ListBackupsRequest.newBuilder().setParent(instanceName.toString()).setPageSize(10).build();
  while (true) {
    ListBackupsPagedResponse response = dbAdminClient.listBackups(listBackupsRequest);
    for (Backup backup : response.getPage().iterateAll()) {
      System.out.println(backup);
    }
    String nextPageToken = response.getNextPageToken();
    if (!Strings.isNullOrEmpty(nextPageToken)) {
      listBackupsRequest = listBackupsRequest.toBuilder().setPageToken(nextPageToken).build();
    } else {
      break;
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

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

try {
  // Get the parent(instance) of the database
  const parent = databaseAdminClient.instancePath(projectId, instanceId);

  // List all backups
  const [allBackups] = await databaseAdminClient.listBackups({
    parent: parent,
  });

  console.log('All backups:');
  allBackups.forEach(backups => {
    if (backups.name) {
      const backup = backups.name;
      const delimiter =
        'projects/' + projectId + '/instances/' + instanceId + '/backups/';
      const result = backup.substring(delimiter.length);
      console.log(result);
    }
  });

  // List backups filtered by backup name
  const [backupsByName] = await databaseAdminClient.listBackups({
    parent: parent,
    filter: `Name:${backupId}`,
  });
  console.log('Backups matching backup name:');
  backupsByName.forEach(backup => {
    if (backup.name) {
      const backupName = backup.name;
      const delimiter =
        'projects/' + projectId + '/instances/' + instanceId + '/backups/';
      const result = backupName.substring(delimiter.length);
      console.log(result);
    }
  });

  // List backups expiring within 30 days
  const expireTime = new Date();
  expireTime.setDate(expireTime.getDate() + 30);
  const [backupsByExpiry] = await databaseAdminClient.listBackups({
    parent: parent,
    filter: `expire_time < "${expireTime.toISOString()}"`,
  });
  console.log('Backups expiring within 30 days:');
  backupsByExpiry.forEach(backup => {
    if (backup.name) {
      const backupName = backup.name;
      const delimiter =
        'projects/' + projectId + '/instances/' + instanceId + '/backups/';
      const result = backupName.substring(delimiter.length);
      console.log(result);
    }
  });

  // List backups filtered by database name
  const [backupsByDbName] = await databaseAdminClient.listBackups({
    parent: parent,
    filter: `Database:${databaseId}`,
  });
  console.log('Backups matching database name:');
  backupsByDbName.forEach(backup => {
    if (backup.name) {
      const backupName = backup.name;
      const delimiter =
        'projects/' + projectId + '/instances/' + instanceId + '/backups/';
      const result = backupName.substring(delimiter.length);
      console.log(result);
    }
  });

  // List backups filtered by backup size
  const [backupsBySize] = await databaseAdminClient.listBackups({
    parent: parent,
    filter: 'size_bytes > 100',
  });
  console.log('Backups filtered by size:');
  backupsBySize.forEach(backup => {
    if (backup.name) {
      const backupName = backup.name;
      const delimiter =
        'projects/' + projectId + '/instances/' + instanceId + '/backups/';
      const result = backupName.substring(delimiter.length);
      console.log(result);
    }
  });

  // List backups that are ready that were created after a certain time
  const createTime = new Date();
  createTime.setDate(createTime.getDate() - 1);
  const [backupsByCreateTime] = await databaseAdminClient.listBackups({
    parent: parent,
    filter: `(state:READY) AND (create_time >= "${createTime.toISOString()}")`,
  });
  console.log('Ready backups filtered by create time:');
  backupsByCreateTime.forEach(backup => {
    if (backup.name) {
      const backupName = backup.name;
      const delimiter =
        'projects/' + projectId + '/instances/' + instanceId + '/backups/';
      const result = backupName.substring(delimiter.length);
      console.log(result);
    }
  });

  // List backups using pagination
  console.log('Get backups paginated:');
  const [backups] = await databaseAdminClient.listBackups({
    parent: parent,
    pageSize: 3,
  });
  backups.forEach(backup => {
    if (backup.name) {
      const backupName = backup.name;
      const delimiter =
        'projects/' + projectId + '/instances/' + instanceId + '/backups/';
      const result = backupName.substring(delimiter.length);
      console.log(result);
    }
  });
} catch (err) {
  console.error('ERROR:', err);
}
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\ListBackupsRequest;

/**
 * List backups in an instance.
 * Example:
 * ```
 * list_backups($projectId, $instanceId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 */
function list_backups(string $projectId, string $instanceId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $parent = DatabaseAdminClient::instanceName($projectId, $instanceId);

    // List all backups.
    print('All backups:' . PHP_EOL);
    $request = new ListBackupsRequest([
        'parent' => $parent
    ]);
    $backups = $databaseAdminClient->listBackups($request)->iterateAllElements();
    foreach ($backups as $backup) {
        print('  ' . basename($backup->getName()) . PHP_EOL);
    }

    // List all backups that contain a name.
    $backupName = 'backup-test-';
    print("All backups with name containing \"$backupName\":" . PHP_EOL);
    $filter = "name:$backupName";
    $request = new ListBackupsRequest([
        'parent' => $parent,
        'filter' => $filter
    ]);
    $backups = $databaseAdminClient->listBackups($request)->iterateAllElements();
    foreach ($backups as $backup) {
        print('  ' . basename($backup->getName()) . PHP_EOL);
    }

    // List all backups for a database that contains a name.
    $databaseId = 'test-';
    print("All backups for a database which name contains \"$databaseId\":" . PHP_EOL);
    $filter = "database:$databaseId";
    $request = new ListBackupsRequest([
        'parent' => $parent,
        'filter' => $filter
    ]);
    $backups = $databaseAdminClient->listBackups($request)->iterateAllElements();
    foreach ($backups as $backup) {
        print('  ' . basename($backup->getName()) . PHP_EOL);
    }

    // List all backups that expire before a timestamp.
    $expireTime = (new \DateTime('+30 days'))->format('c');
    print("All backups that expire before $expireTime:" . PHP_EOL);
    $filter = "expire_time < \"$expireTime\"";
    $request = new ListBackupsRequest([
        'parent' => $parent,
        'filter' => $filter
    ]);
    $backups = $databaseAdminClient->listBackups($request)->iterateAllElements();
    foreach ($backups as $backup) {
        print('  ' . basename($backup->getName()) . PHP_EOL);
    }

    // List all backups with a size greater than some bytes.
    $size = 500;
    print("All backups with size greater than $size bytes:" . PHP_EOL);
    $filter = "size_bytes > $size";
    $request = new ListBackupsRequest([
        'parent' => $parent,
        'filter' => $filter
    ]);
    $backups = $databaseAdminClient->listBackups($request)->iterateAllElements();
    foreach ($backups as $backup) {
        print('  ' . basename($backup->getName()) . PHP_EOL);
    }

    // List backups that were created after a timestamp that are also ready.
    $createTime = (new \DateTime('-1 day'))->format('c');
    print("All backups created after $createTime:" . PHP_EOL);
    $filter = "create_time >= \"$createTime\" AND state:READY";
    $request = new ListBackupsRequest([
        'parent' => $parent,
        'filter' => $filter
    ]);
    $backups = $databaseAdminClient->listBackups($request)->iterateAllElements();
    foreach ($backups as $backup) {
        print('  ' . basename($backup->getName()) . PHP_EOL);
    }

    // List backups with pagination.
    print('All backups with pagination:' . PHP_EOL);
    $request = new ListBackupsRequest([
        'parent' => $parent,
        'page_size' => 2
    ]);
    $pages = $databaseAdminClient->listBackups($request)->iteratePages();
    foreach ($pages as $pageNumber => $page) {
        print("All backups, page $pageNumber:" . PHP_EOL);
        foreach ($page as $backup) {
            print('  ' . basename($backup->getName()) . PHP_EOL);
        }
    }
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def list_backups(instance_id, database_id, backup_id):
    from google.cloud.spanner_admin_database_v1.types import backup as backup_pb

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    # List all backups.
    print("All backups:")
    request = backup_pb.ListBackupsRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        filter="",
    )
    operations = database_admin_api.list_backups(request)
    for backup in operations:
        print(backup.name)

    # List all backups that contain a name.
    print('All backups with backup name containing "{}":'.format(backup_id))
    request = backup_pb.ListBackupsRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        filter="name:{}".format(backup_id),
    )
    operations = database_admin_api.list_backups(request)
    for backup in operations:
        print(backup.name)

    # List all backups for a database that contains a name.
    print('All backups with database name containing "{}":'.format(database_id))
    request = backup_pb.ListBackupsRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        filter="database:{}".format(database_id),
    )
    operations = database_admin_api.list_backups(request)
    for backup in operations:
        print(backup.name)

    # List all backups that expire before a timestamp.
    expire_time = datetime.utcnow().replace(microsecond=0) + timedelta(days=30)
    print(
        'All backups with expire_time before "{}-{}-{}T{}:{}:{}Z":'.format(
            *expire_time.timetuple()
        )
    )
    request = backup_pb.ListBackupsRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        filter='expire_time < "{}-{}-{}T{}:{}:{}Z"'.format(*expire_time.timetuple()),
    )
    operations = database_admin_api.list_backups(request)
    for backup in operations:
        print(backup.name)

    # List all backups with a size greater than some bytes.
    print("All backups with backup size more than 100 bytes:")
    request = backup_pb.ListBackupsRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        filter="size_bytes > 100",
    )
    operations = database_admin_api.list_backups(request)
    for backup in operations:
        print(backup.name)

    # List backups that were created after a timestamp that are also ready.
    create_time = datetime.utcnow().replace(microsecond=0) - timedelta(days=1)
    print(
        'All backups created after "{}-{}-{}T{}:{}:{}Z" and are READY:'.format(
            *create_time.timetuple()
        )
    )
    request = backup_pb.ListBackupsRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        filter='create_time >= "{}-{}-{}T{}:{}:{}Z" AND state:READY'.format(
            *create_time.timetuple()
        ),
    )
    operations = database_admin_api.list_backups(request)
    for backup in operations:
        print(backup.name)

    print("All backups with pagination")
    # If there are multiple pages, additional ``ListBackup``
    # requests will be made as needed while iterating.
    paged_backups = set()
    request = backup_pb.ListBackupsRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        page_size=2,
    )
    operations = database_admin_api.list_backups(request)
    for backup in operations:
        paged_backups.add(backup.name)
    for backup in paged_backups:
        print(backup)
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# backup_id = "Your Spanner database backup ID"
# database_id = "Your Spanner databaseID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin
instance_path = database_admin_client.instance_path project: project_id, instance: instance_id

puts "All backups"
database_admin_client.list_backups(parent: instance_path).each do |backup|
  puts backup.name
end

puts "All backups with backup name containing \"#{backup_id}\":"
database_admin_client.list_backups(parent: instance_path, filter: "name:#{backup_id}").each do |backup|
  puts backup.name
end

puts "All backups for databases with a name containing \"#{database_id}\":"
database_admin_client.list_backups(parent: instance_path, filter: "database:#{database_id}").each do |backup|
  puts backup.name
end

puts "All backups that expire before a timestamp:"
expire_time = Time.now + (30 * 24 * 3600) # 30 days from now
database_admin_client.list_backups(parent: instance_path, filter: "expire_time < \"#{expire_time.iso8601}\"").each do |backup|
  puts backup.name
end

puts "All backups with a size greater than 500 bytes:"
database_admin_client.list_backups(parent: instance_path, filter: "size_bytes >= 500").each do |backup|
  puts backup.name
end

puts "All backups that were created after a timestamp that are also ready:"
create_time = Time.now - (24 * 3600) # From 1 day ago
database_admin_client.list_backups(parent: instance_path, filter: "create_time >= \"#{create_time.iso8601}\" AND state:READY").each do |backup|
  puts backup.name
end

puts "All backups with pagination:"
list = database_admin_client.list_backups parent: instance_path, page_size: 5
list.each do |backup|
  puts backup.name
end
```

## Update the backup expiration period

### Console

1.  Go to the Spanner Instances page in the Google Cloud console.

2.  Click the instance containing the database to open its **Overview** page.

3.  Click the database to open its **Overview** page.

4.  In the navigation pane, click **Backup/Restore** .

5.  Click the **Actions** button for your selected backup, then select **Update metadata** .

6.  Select the new expiration date.

7.  Click **Update** .

### gcloud

To update a backup's expiration period date, use [`  gcloud spanner backups update-metadata  `](/sdk/gcloud/reference/spanner/backups/update-metadata) :

Before using any of the command data below, make the following replacements:

  - PROJECT\_ID : the project ID.
  - BACKUP\_ID : the Spanner backup ID.
  - INSTANCE\_ID : the Spanner instance ID.
  - EXPIRATION\_DATE : the expiration date time stamp.
  - DATABASE\_ID : the Spanner database ID.

Execute the following command:

#### Linux, macOS, or Cloud Shell

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups update-metadata BACKUP_ID \
--instance=INSTANCE_ID \
--expiration-date=EXPIRATION_DATE
```

#### Windows (PowerShell)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups update-metadata BACKUP_ID `
--instance=INSTANCE_ID `
--expiration-date=EXPIRATION_DATE
```

#### Windows (cmd.exe)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups update-metadata BACKUP_ID ^
--instance=INSTANCE_ID ^
--expiration-date=EXPIRATION_DATE
```

You should receive a response similar to the following:

``` text
createTime: '2020-02-04T02:05:43.920377Z'
database: projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID
expireTime: '2020-05-05T00:00:00Z'
name: projects/PROJECT_ID/instances/INSTANCE_ID/backups/BACKUP_ID
sizeBytes: '1000000000'
state: READY
```

### Client libraries

The following code sample retrieves the expiry time of a backup and extends it.

### C++

``` cpp
void UpdateBackup(google::cloud::spanner_admin::DatabaseAdminClient client,
                  std::string const& project_id, std::string const& instance_id,
                  std::string const& backup_id,
                  absl::Duration expiry_extension) {
  google::cloud::spanner::Backup backup_name(
      google::cloud::spanner::Instance(project_id, instance_id), backup_id);
  auto backup = client.GetBackup(backup_name.FullName());
  if (!backup) throw std::move(backup).status();
  auto expire_time =
      google::cloud::spanner::MakeTimestamp(backup->expire_time())
          .value()
          .get<absl::Time>()
          .value();
  expire_time += expiry_extension;
  auto max_expire_time =
      google::cloud::spanner::MakeTimestamp(backup->max_expire_time())
          .value()
          .get<absl::Time>()
          .value();
  if (expire_time > max_expire_time) expire_time = max_expire_time;
  google::spanner::admin::database::v1::UpdateBackupRequest request;
  request.mutable_backup()->set_name(backup_name.FullName());
  *request.mutable_backup()->mutable_expire_time() =
      google::cloud::spanner::MakeTimestamp(expire_time)
          .value()
          .get<google::protobuf::Timestamp>()
          .value();
  request.mutable_update_mask()->add_paths("expire_time");
  backup = client.UpdateBackup(request);
  if (!backup) throw std::move(backup).status();
  std::cout
      << "Backup " << backup->name() << " updated to expire at "
      << google::cloud::spanner::MakeTimestamp(backup->expire_time()).value()
      << ".\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Protobuf.WellKnownTypes;
using System;

public class UpdateBackupSample
{
    public Backup UpdateBackup(string projectId, string instanceId, string backupId)
    {
        // Create the DatabaseAdminClient instance.
        DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();

        // Retrieve existing backup.
        BackupName backupName = BackupName.FromProjectInstanceBackup(projectId, instanceId, backupId);
        Backup backup = databaseAdminClient.GetBackup(backupName);

        // Add 1 hour to the existing ExpireTime.
        backup.ExpireTime = backup.ExpireTime.ToDateTime().AddHours(1).ToTimestamp();

        UpdateBackupRequest backupUpdateRequest = new UpdateBackupRequest
        {
            UpdateMask = new FieldMask
            {
                Paths = { "expire_time" }
            },
            Backup = backup
        };

        // Make the UpdateBackup requests.
        var updatedBackup = databaseAdminClient.UpdateBackup(backupUpdateRequest);

        Console.WriteLine($"Updated Backup ExpireTime: {updatedBackup.ExpireTime}");

        return updatedBackup;
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
 "time"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
 "google.golang.org/genproto/protobuf/field_mask"
 "google.golang.org/protobuf/types/known/timestamppb"
)

// updateBackup updates the expiration time of a pending or completed backup.
func updateBackup(w io.Writer, db string, backupID string) error {
 // db := "projects/my-project/instances/my-instance/databases/my-database"
 // backupID := "my-backup"

 // Add timeout to context.
 ctx, cancel := context.WithTimeout(context.Background(), time.Hour)
 defer cancel()

 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 matches := regexp.MustCompile("^(.*)/databases/(.*)$").FindStringSubmatch(db)
 if matches == nil || len(matches) != 3 {
     return fmt.Errorf("invalid database id %s", db)
 }
 backupName := matches[1] + "/backups/" + backupID

 // Get the backup instance.
 backup, err := adminClient.GetBackup(ctx, &adminpb.GetBackupRequest{Name: backupName})
 if err != nil {
     return err
 }

 // Expire time must be within 366 days of the create time of the backup.
 maxExpireTime := time.Unix(backup.MaxExpireTime.Seconds, int64(backup.MaxExpireTime.Nanos))
 expireTime := time.Unix(backup.ExpireTime.Seconds, int64(backup.ExpireTime.Nanos)).AddDate(0, 0, 30)

 // Ensure that new expire time is less than the max expire time.
 if expireTime.After(maxExpireTime) {
     expireTime = maxExpireTime
 }
 expireTimepb := timestamppb.New(expireTime)

 // Make the update backup request.
 _, err = adminClient.UpdateBackup(ctx, &adminpb.UpdateBackupRequest{
     Backup: &adminpb.Backup{
         Name:       backupName,
         ExpireTime: expireTimepb,
     },
     UpdateMask: &field_mask.FieldMask{Paths: []string{"expire_time"}},
 })
 if err != nil {
     return err
 }

 fmt.Fprintf(w, "Updated backup %s with expire time %s\n", backupName, expireTime)

 return nil
}
```

### Java

``` java
static void updateBackup(DatabaseAdminClient dbAdminClient, String projectId,
    String instanceId, String backupId) {
  BackupName backupName = BackupName.of(projectId, instanceId, backupId);

  // Get current backup metadata.
  Backup backup = dbAdminClient.getBackup(backupName);
  // Add 30 days to the expire time.
  // Expire time must be within 366 days of the create time of the backup.
  Timestamp currentExpireTime = backup.getExpireTime();
  com.google.cloud.Timestamp newExpireTime =
      com.google.cloud.Timestamp.ofTimeMicroseconds(
          TimeUnit.SECONDS.toMicros(currentExpireTime.getSeconds())
              + TimeUnit.NANOSECONDS.toMicros(currentExpireTime.getNanos())
              + TimeUnit.DAYS.toMicros(30L));

  // New Expire Time must be less than Max Expire Time
  newExpireTime =
      newExpireTime.compareTo(com.google.cloud.Timestamp.fromProto(backup.getMaxExpireTime()))
          < 0 ? newExpireTime : com.google.cloud.Timestamp.fromProto(backup.getMaxExpireTime());

  System.out.println(String.format(
      "Updating expire time of backup [%s] to %s...",
      backupId.toString(),
      java.time.OffsetDateTime.ofInstant(
          Instant.ofEpochSecond(newExpireTime.getSeconds(),
              newExpireTime.getNanos()), ZoneId.systemDefault())));

  // Update expire time.
  backup = backup.toBuilder().setExpireTime(newExpireTime.toProto()).build();
  dbAdminClient.updateBackup(backup,
      FieldMask.newBuilder().addAllPaths(Lists.newArrayList("expire_time")).build());
  System.out.println("Updated backup [" + backupId + "]");
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
// Imports the Google Cloud client library and precise date library
const {Spanner, protos} = require('@google-cloud/spanner');
const {PreciseDate} = require('@google-cloud/precise-date');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const backupId = 'my-backup';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner Database Admin Client object
const databaseAdminClient = spanner.getDatabaseAdminClient();

// Read backup metadata and update expiry time
try {
  const [metadata] = await databaseAdminClient.getBackup({
    name: databaseAdminClient.backupPath(projectId, instanceId, backupId),
  });

  const currentExpireTime = metadata.expireTime;
  const maxExpireTime = metadata.maxExpireTime;
  const wantExpireTime = new PreciseDate(currentExpireTime);
  wantExpireTime.setDate(wantExpireTime.getDate() + 1);

  // New expire time should be less than the max expire time
  const min = (currentExpireTime, maxExpireTime) =>
    currentExpireTime < maxExpireTime ? currentExpireTime : maxExpireTime;
  const newExpireTime = new PreciseDate(min(wantExpireTime, maxExpireTime));
  console.log(
    `Backup ${backupId} current expire time: ${Spanner.timestamp(
      currentExpireTime,
    ).toISOString()}`,
  );
  console.log(
    `Updating expire time to ${Spanner.timestamp(
      newExpireTime,
    ).toISOString()}`,
  );

  await databaseAdminClient.updateBackup({
    backup: {
      name: databaseAdminClient.backupPath(projectId, instanceId, backupId),
      expireTime: Spanner.timestamp(newExpireTime).toStruct(),
    },
    updateMask: (protos.google.protobuf.FieldMask = {
      paths: ['expire_time'],
    }),
  });
  console.log('Expire time updated.');
} catch (err) {
  console.error('ERROR:', err);
}
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Backup;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateBackupRequest;
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Protobuf\Timestamp;

/**
 * Update the backup expire time.
 * Example:
 * ```
 * update_backup($projectId, $instanceId, $backupId);
 * ```
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $backupId The Spanner backup ID.
 */
function update_backup(string $projectId, string $instanceId, string $backupId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $backupName = DatabaseAdminClient::backupName($projectId, $instanceId, $backupId);
    $newExpireTime = new Timestamp();
    $newExpireTime->setSeconds((new \DateTime('+30 days'))->getTimestamp());
    $request = new UpdateBackupRequest([
        'backup' => new Backup([
            'name' => $backupName,
            'expire_time' => $newExpireTime
        ]),
        'update_mask' => new \Google\Protobuf\FieldMask(['paths' => ['expire_time']])
    ]);

    $info = $databaseAdminClient->updateBackup($request);
    printf('Backup %s new expire time: %d' . PHP_EOL, basename($info->getName()), $info->getExpireTime()->getSeconds());
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def update_backup(instance_id, backup_id):
    from google.cloud.spanner_admin_database_v1.types import backup as backup_pb

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    backup = database_admin_api.get_backup(
        backup_pb.GetBackupRequest(
            name=database_admin_api.backup_path(
                spanner_client.project, instance_id, backup_id
            ),
        )
    )

    # Expire time must be within 366 days of the create time of the backup.
    old_expire_time = backup.expire_time
    # New expire time should be less than the max expire time
    new_expire_time = min(backup.max_expire_time, old_expire_time + timedelta(days=30))
    database_admin_api.update_backup(
        backup_pb.UpdateBackupRequest(
            backup=backup_pb.Backup(name=backup.name, expire_time=new_expire_time),
            update_mask={"paths": ["expire_time"]},
        )
    )
    print(
        "Backup {} expire time was updated from {} to {}.".format(
            backup.name, old_expire_time, new_expire_time
        )
    )
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# backup_id = "Your Spanner backup ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin
instance_path = database_admin_client.instance_path project: project_id, instance: instance_id
backup_path = database_admin_client.backup_path project: project_id,
                                                instance: instance_id,
                                                backup: backup_id
backup = database_admin_client.get_backup name: backup_path
backup.expire_time = Time.now + (60 * 24 * 3600) # Extending the expiry time by 60 days from now.
database_admin_client.update_backup backup: backup,
                                    update_mask: { paths: ["expire_time"] }

puts "Expiration time updated: #{backup.expire_time}"
```

## Delete a backup

When you delete a backup, Spanner releases the storage and all other resources associated with that backup.

If you delete a backup that's still being created, Spanner also cancels the long-running backup operation.

Deleting an incremental backup might not free up storage if a younger incremental backup depends on it. For more information, see [Backups overview](/spanner/docs/backup#incremental-backups) .

### Console

1.  Go to the Spanner Instances page in the Google Cloud console.

2.  Click the instance containing the database to open its **Overview** page.

3.  Click the database to open its **Overview** page.

4.  In the navigation pane, click **Backup/Restore** .

5.  Click the **Actions** button for your selected backup, then select **Delete** .

6.  Type the backup ID.

7.  Click **Delete** .

### gcloud

To delete a backup, use [`  gcloud spanner backups delete  `](/sdk/gcloud/reference/spanner/backups/delete) .

Before using any of the command data below, make the following replacements:

  - INSTANCE\_ID : the Spanner instance ID.
  - BACKUP\_NAME : the Spanner backup name.

Execute the following command:

#### Linux, macOS, or Cloud Shell

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups delete BACKUP_NAME --instance=INSTANCE_ID
```

#### Windows (PowerShell)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups delete BACKUP_NAME --instance=INSTANCE_ID
```

#### Windows (cmd.exe)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backups delete BACKUP_NAME --instance=INSTANCE_ID
```

You should receive a response similar to the following:

``` text
You are about to delete backup BACKUP_NAME

Do you want to continue (Y/n)?  Y

Deleted backup BACKUP_NAME.
```

### Client libraries

The following code sample deletes a backup and verifies that it was deleted.

### C++

``` cpp
void DeleteBackup(google::cloud::spanner_admin::DatabaseAdminClient client,
                  std::string const& project_id, std::string const& instance_id,
                  std::string const& backup_id) {
  google::cloud::spanner::Backup backup(
      google::cloud::spanner::Instance(project_id, instance_id), backup_id);
  auto status = client.DeleteBackup(backup.FullName());
  if (!status.ok()) throw std::move(status);
  std::cout << "Backup " << backup.FullName() << " was deleted.\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using System;

public class DeleteBackupSample
{
    public void DeleteBackup(string projectId, string instanceId, string backupId)
    {
        // Create the DatabaseAdminClient instance.
        DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();

        // Make the DeleteBackup request.
        BackupName backupName = BackupName.FromProjectInstanceBackup(projectId, instanceId, backupId);
        databaseAdminClient.DeleteBackup(backupName);

        Console.WriteLine("Backup deleted successfully.");
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

func deleteBackup(ctx context.Context, w io.Writer, db, backupID string) error {
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 matches := regexp.MustCompile("^(.*)/databases/(.*)$").FindStringSubmatch(db)
 if matches == nil || len(matches) != 3 {
     return fmt.Errorf("Invalid database id %s", db)
 }
 backupName := matches[1] + "/backups/" + backupID
 // Delete the backup.
 err = adminClient.DeleteBackup(ctx, &adminpb.DeleteBackupRequest{Name: backupName})
 if err != nil {
     return err
 }
 fmt.Fprintf(w, "Deleted backup %s\n", backupID)
 return nil
}
```

### Java

``` java
static void deleteBackup(DatabaseAdminClient dbAdminClient,
    String project, String instance, String backupId) {
  BackupName backupName = BackupName.of(project, instance, backupId);

  // Delete the backup.
  System.out.println("Deleting backup [" + backupId + "]...");
  dbAdminClient.deleteBackup(backupName);
  // Verify that the backup is deleted.
  try {
    dbAdminClient.getBackup(backupName);
  } catch (NotFoundException e) {
    if (e.getStatusCode().getCode() == Code.NOT_FOUND) {
      System.out.println("Deleted backup [" + backupId + "]");
    } else {
      System.out.println("Delete backup [" + backupId + "] failed");
      throw new RuntimeException("Delete backup [" + backupId + "] failed", e);
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

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

// Delete the backup
console.log(`Deleting backup ${backupId}.`);
await databaseAdminClient.deleteBackup({
  name: databaseAdminClient.backupPath(projectId, instanceId, backupId),
});
console.log('Backup deleted.');

// Verify backup no longer exists
try {
  await databaseAdminClient.getBackup({
    name: databaseAdminClient.backupPath(projectId, instanceId, backupId),
  });
  console.error('Error: backup still exists.');
} catch (err) {
  console.log('Backup deleted.');
}
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\DeleteBackupRequest;

/**
 * Delete a backup.
 * Example:
 * ```
 * delete_backup($projectId, $instanceId, $backupId);
 * ```
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $backupId The Spanner backup ID.
 */
function delete_backup(string $projectId, string $instanceId, string $backupId): void
{
    $databaseAdminClient = new DatabaseAdminClient();

    $backupName = DatabaseAdminClient::backupName($projectId, $instanceId, $backupId);

    $request = new DeleteBackupRequest();
    $request->setName($backupName);
    $databaseAdminClient->deleteBackup($request);

    print("Backup $backupName deleted" . PHP_EOL);
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def delete_backup(instance_id, backup_id):
    from google.cloud.spanner_admin_database_v1.types import backup as backup_pb

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api
    backup = database_admin_api.get_backup(
        backup_pb.GetBackupRequest(
            name=database_admin_api.backup_path(
                spanner_client.project, instance_id, backup_id
            ),
        )
    )

    # Wait for databases that reference this backup to finish optimizing.
    while backup.referencing_databases:
        time.sleep(30)
        backup = database_admin_api.get_backup(
            backup_pb.GetBackupRequest(
                name=database_admin_api.backup_path(
                    spanner_client.project, instance_id, backup_id
                ),
            )
        )

    # Delete the backup.
    database_admin_api.delete_backup(backup_pb.DeleteBackupRequest(name=backup.name))

    # Verify that the backup is deleted.
    try:
        backup = database_admin_api.get_backup(
            backup_pb.GetBackupRequest(name=backup.name)
        )
    except NotFound:
        print("Backup {} has been deleted.".format(backup.name))
        return
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# backup_id = "Your Spanner backup ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin
instance_path = database_admin_client.instance_path project: project_id, instance: instance_id
backup_path = database_admin_client.backup_path project: project_id,
                                                instance: instance_id,
                                                backup: backup_id

database_admin_client.delete_backup name: backup_path
puts "Backup #{backup_id} deleted"
```

## What's next

  - To learn more about backups, see [Backups overview](/spanner/docs/backup) .

  - To create backups, see [Create backups](/spanner/docs/backup/create-backups) .
