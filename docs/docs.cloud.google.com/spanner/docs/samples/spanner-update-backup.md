---
name: documents/docs.cloud.google.com/spanner/docs/samples/spanner-update-backup
uri: https://docs.cloud.google.com/spanner/docs/samples/spanner-update-backup
title: Update backup
description: Update a backup by retrieving the expiry time of a backup and extending it.
data_source: docs.cloud.google.com
---

Update a backup by retrieving the expiry time of a backup and extending it.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Manage backups](https://docs.cloud.google.com/spanner/docs/backup/manage-backups)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    use Google\Cloud\Spanner\SpannerClient;
    use DateTime;
    
    /**
     * Update the backup expire time.
     * Example:
     * ```
     * update_backup($instanceId, $backupId);
     * ```
     * @param string $instanceId The Spanner instance ID.
     * @param string $backupId The Spanner backup ID.
     */
    function update_backup(string $instanceId, string $backupId): void
    {
        $spanner = new SpannerClient();
        $instance = $spanner->instance($instanceId);
        $backup = $instance->backup($backupId);
        $backup->reload();
    
        $newExpireTime = new DateTime('+30 days');
        $maxExpireTime = new DateTime($backup->info()['maxExpireTime']);
        // The new expire time can't be greater than maxExpireTime for the backup.
        $newExpireTime = min($newExpireTime, $maxExpireTime);
    
        $backup->updateExpireTime($newExpireTime);
    
        printf('Backup %s new expire time: %s' . PHP_EOL, $backupId, $backup->info()['expireTime']);
    }

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    def update_backup(instance_id, backup_id):
        spanner_client = spanner.Client()
        instance = spanner_client.instance(instance_id)
        backup = instance.backup(backup_id)
        backup.reload()
    
        # Expire time must be within 366 days of the create time of the backup.
        old_expire_time = backup.expire_time
        # New expire time should be less than the max expire time
        new_expire_time = min(backup.max_expire_time, old_expire_time + timedelta(days=30))
        backup.update_expire_time(new_expire_time)
        print(
            "Backup {} expire time was updated from {} to {}.".format(
                backup.name, old_expire_time, new_expire_time
            )
        )

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=cloudspanner) .
