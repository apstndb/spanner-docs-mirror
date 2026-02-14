Update a backup by retrieving the expiry time of a backup and extending it.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Manage backups](/spanner/docs/backup/manage-backups)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
