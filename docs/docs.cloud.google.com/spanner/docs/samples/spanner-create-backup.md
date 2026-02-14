Create database backup that can be used to restore the database.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Create backups](/spanner/docs/backup/create-backups)
  - [Recover data using point-in-time recovery (PITR)](/spanner/docs/use-pitr)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void CreateBackup(google::cloud::spanner_admin::DatabaseAdminClient client,
                  std::string const& project_id, std::string const& instance_id,
                  std::string const& database_id, std::string const& backup_id,
                  google::cloud::spanner::Timestamp expire_time,
                  google::cloud::spanner::Timestamp version_time) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  google::spanner::admin::database::v1::CreateBackupRequest request;
  request.set_parent(database.instance().FullName());
  request.set_backup_id(backup_id);
  request.mutable_backup()->set_database(database.FullName());
  *request.mutable_backup()->mutable_expire_time() =
      expire_time.get<google::protobuf::Timestamp>().value();
  *request.mutable_backup()->mutable_version_time() =
      version_time.get<google::protobuf::Timestamp>().value();
  auto backup = client.CreateBackup(request).get();
  if (!backup) throw std::move(backup).status();
  std::cout
      << "Backup " << backup->name() << " of " << backup->database()
      << " of size " << backup->size_bytes() << " bytes as of "
      << google::cloud::spanner::MakeTimestamp(backup->version_time()).value()
      << " was created at "
      << google::cloud::spanner::MakeTimestamp(backup->create_time()).value()
      << ".\n";
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

public class CreateBackupSample
{
    public Backup CreateBackup(string projectId, string instanceId, string databaseId, string backupId, DateTime versionTime)
    {
        // Create the DatabaseAdminClient instance.
        DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();

        // Initialize request parameters.
        Backup backup = new Backup
        {
            DatabaseAsDatabaseName = DatabaseName.FromProjectInstanceDatabase(projectId, instanceId, databaseId),
            ExpireTime = DateTime.UtcNow.AddDays(14).ToTimestamp(),
            VersionTime = versionTime.ToTimestamp(),
        };
        InstanceName instanceName = InstanceName.FromProjectInstance(projectId, instanceId);

        // Make the CreateBackup request.
        Operation<Backup, CreateBackupMetadata> response = databaseAdminClient.CreateBackup(instanceName, backup, backupId);

        Console.WriteLine("Waiting for the operation to finish.");

        // Poll until the returned long-running operation is complete.
        Operation<Backup, CreateBackupMetadata> completedResponse = response.PollUntilCompleted();

        if (completedResponse.IsFaulted)
        {
            Console.WriteLine($"Error while creating backup: {completedResponse.Exception}");
            throw completedResponse.Exception;
        }

        Console.WriteLine($"Backup created successfully.");

        // GetBackup to get more information about the created backup.
        BackupName backupName = BackupName.FromProjectInstanceBackup(projectId, instanceId, backupId);
        backup = databaseAdminClient.GetBackup(backupName);
        Console.WriteLine($"Backup {backup.Name} of size {backup.SizeBytes} bytes " +
                      $"was created at {backup.CreateTime} from {backup.Database} " +
                      $"and is in state {backup.State} " +
                      $"and has version time {backup.VersionTime}");
        return backup;
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

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
 pbt "github.com/golang/protobuf/ptypes/timestamp"
)

func createBackup(ctx context.Context, w io.Writer, db, backupID string, versionTime time.Time) error {
 // versionTime := time.Now().AddDate(0, 0, -1) // one day ago
 matches := regexp.MustCompile("^(.+)/databases/(.+)$").FindStringSubmatch(db)
 if matches == nil || len(matches) != 3 {
     return fmt.Errorf("createBackup: invalid database id %q", db)
 }

 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return fmt.Errorf("createBackup.NewDatabaseAdminClient: %w", err)
 }
 defer adminClient.Close()

 expireTime := time.Now().AddDate(0, 0, 14)
 // Create a backup.
 req := adminpb.CreateBackupRequest{
     Parent:   matches[1],
     BackupId: backupID,
     Backup: &adminpb.Backup{
         Database:    db,
         ExpireTime:  &pbt.Timestamp{Seconds: expireTime.Unix(), Nanos: int32(expireTime.Nanosecond())},
         VersionTime: &pbt.Timestamp{Seconds: versionTime.Unix(), Nanos: int32(versionTime.Nanosecond())},
     },
 }
 op, err := adminClient.CreateBackup(ctx, &req)
 if err != nil {
     return fmt.Errorf("createBackup.CreateBackup: %w", err)
 }
 // Wait for backup operation to complete.
 backup, err := op.Wait(ctx)
 if err != nil {
     return fmt.Errorf("createBackup.Wait: %w", err)
 }

 // Get the name, create time, version time and backup size.
 backupCreateTime := time.Unix(backup.CreateTime.Seconds, int64(backup.CreateTime.Nanos))
 backupVersionTime := time.Unix(backup.VersionTime.Seconds, int64(backup.VersionTime.Nanos))
 fmt.Fprintf(w,
     "Backup %s of size %d bytes was created at %s with version time %s\n",
     backup.Name,
     backup.SizeBytes,
     backupCreateTime.Format(time.RFC3339),
     backupVersionTime.Format(time.RFC3339))
 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void createBackup(DatabaseAdminClient dbAdminClient, String projectId, String instanceId,
    String databaseId, String backupId, Timestamp versionTime) {
  // Set expire time to 14 days from now.
  Timestamp expireTime =
      Timestamp.newBuilder().setSeconds(TimeUnit.MILLISECONDS.toSeconds((
          System.currentTimeMillis() + TimeUnit.DAYS.toMillis(14)))).build();
  BackupName backupName = BackupName.of(projectId, instanceId, backupId);
  Backup backup = Backup.newBuilder()
      .setName(backupName.toString())
      .setDatabase(DatabaseName.of(projectId, instanceId, databaseId).toString())
      .setExpireTime(expireTime).setVersionTime(versionTime).build();

  // Initiate the request which returns an OperationFuture.
  System.out.println("Creating backup [" + backupId + "]...");
  try {
    // Wait for the backup operation to complete.
    backup = dbAdminClient.createBackupAsync(
        InstanceName.of(projectId, instanceId), backup, backupId).get();
    System.out.println("Created backup [" + backup.getName() + "]");
  } catch (ExecutionException e) {
    throw SpannerExceptionFactory.asSpannerException(e);
  } catch (InterruptedException e) {
    throw SpannerExceptionFactory.propagateInterrupt(e);
  }

  // Reload the metadata of the backup from the server.
  backup = dbAdminClient.getBackup(backup.getName());
  System.out.println(
      String.format(
          "Backup %s of size %d bytes was created at %s for version of database at %s",
          backup.getName(),
          backup.getSizeBytes(),
          java.time.OffsetDateTime.ofInstant(
              Instant.ofEpochSecond(backup.getCreateTime().getSeconds(),
                  backup.getCreateTime().getNanos()), ZoneId.systemDefault()),
          java.time.OffsetDateTime.ofInstant(
              Instant.ofEpochSecond(backup.getVersionTime().getSeconds(),
                  backup.getVersionTime().getNanos()), ZoneId.systemDefault()))
  );
}
```

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` javascript
// Imports the Google Cloud client library and precise date library
const {Spanner, protos} = require('@google-cloud/spanner');
const {PreciseDate} = require('@google-cloud/precise-date');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';
// const backupId = 'my-backup';
// const versionTime = Date.now() - 1000 * 60 * 60 * 24; // One day ago

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

  // Expire backup 14 days in the future
  const expireTime = Date.now() + 1000 * 60 * 60 * 24 * 14;

  // Create a backup of the state of the database at the current time.
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
      versionTime: Spanner.timestamp(versionTime).toStruct(),
      name: databaseAdminClient.backupPath(projectId, instanceId, backupId),
    }),
  });

  console.log(
    `Waiting for backup ${databaseAdminClient.backupPath(
      projectId,
      instanceId,
      backupId,
    )} to complete...`,
  );
  await operation.promise();

  // Verify backup is ready
  const [backupInfo] = await databaseAdminClient.getBackup({
    name: databaseAdminClient.backupPath(projectId, instanceId, backupId),
  });
  if (backupInfo.state === 'READY') {
    console.log(
      `Backup ${backupInfo.name} of size ` +
        `${backupInfo.sizeBytes} bytes was created at ` +
        `${new PreciseDate(backupInfo.createTime).toISOString()} ` +
        'for version of database at ' +
        `${new PreciseDate(backupInfo.versionTime).toISOString()}`,
    );
  } else {
    console.error('ERROR: Backup is not ready.');
  }
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
use Google\Cloud\Spanner\Admin\Database\V1\Backup;
use Google\Cloud\Spanner\Admin\Database\V1\GetBackupRequest;
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\CreateBackupRequest;
use Google\Protobuf\Timestamp;

/**
 * Create a backup.
 * Example:
 * ```
 * create_backup($projectId, $instanceId, $databaseId, $backupId, $versionTime);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 * @param string $backupId The Spanner backup ID.
 * @param string $versionTime The version of the database to backup. Read more
 * at https://cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups#Backup.FIELDS.version_time
 */
function create_backup(
    string $projectId,
    string $instanceId,
    string $databaseId,
    string $backupId,
    string $versionTime = '-1hour'
): void {
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseFullName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);
    $instanceFullName = DatabaseAdminClient::instanceName($projectId, $instanceId);
    $timestamp = new Timestamp();
    $timestamp->setSeconds((new \DateTime($versionTime))->getTimestamp());
    $expireTime = new Timestamp();
    $expireTime->setSeconds((new \DateTime('+14 days'))->getTimestamp());
    $request = new CreateBackupRequest([
        'parent' => $instanceFullName,
        'backup_id' => $backupId,
        'backup' => new Backup([
            'database' => $databaseFullName,
            'expire_time' => $expireTime,
            'version_time' => $timestamp
        ])
    ]);

    $operation = $databaseAdminClient->createBackup($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    $request = new GetBackupRequest();
    $request->setName($databaseAdminClient->backupName($projectId, $instanceId, $backupId));
    $info = $databaseAdminClient->getBackup($request);
    printf(
        'Backup %s of size %d bytes was created at %d for version of database at %d' . PHP_EOL,
        basename($info->getName()),
        $info->getSizeBytes(),
        $info->getCreateTime()->getSeconds(),
        $info->getVersionTime()->getSeconds());
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def create_backup(instance_id, database_id, backup_id, version_time):
    """Creates a backup for a database."""

    from google.cloud.spanner_admin_database_v1.types import backup as backup_pb

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    # Create a backup
    expire_time = datetime.utcnow() + timedelta(days=14)

    request = backup_pb.CreateBackupRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        backup_id=backup_id,
        backup=backup_pb.Backup(
            database=database_admin_api.database_path(
                spanner_client.project, instance_id, database_id
            ),
            expire_time=expire_time,
            version_time=version_time,
        ),
    )

    operation = database_admin_api.create_backup(request)

    # Wait for backup operation to complete.
    backup = operation.result(2100)

    # Verify that the backup is ready.
    assert backup.state == backup_pb.Backup.State.READY

    print(
        "Backup {} of size {} bytes was created at {} for version of database at {}".format(
            backup.name, backup.size_bytes, backup.create_time, backup.version_time
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
# backup_id = "Your Spanner backup ID"
# version_time = Time.now - 60 * 60 * 24 # 1 day ago

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
                                              expire_time: expire_time,
                                              version_time: version_time
                                          }

puts "Backup operation in progress"

job.wait_until_done!

backup = database_admin_client.get_backup name: backup_path
puts "Backup #{backup_id} of size #{backup.size_bytes} bytes was created at #{backup.create_time} for version of database at #{backup.version_time}"
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
