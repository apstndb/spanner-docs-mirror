---
name: documents/docs.cloud.google.com/spanner/docs/samples/spanner-delete-backup
uri: https://docs.cloud.google.com/spanner/docs/samples/spanner-delete-backup
title: Delete backup
description: Delete a database backup.
data_source: docs.cloud.google.com
---

Delete a database backup.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Manage backups](https://docs.cloud.google.com/spanner/docs/backup/manage-backups)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    void DeleteBackup(google::cloud::spanner_admin::DatabaseAdminClient client,
                      std::string const& project_id, std::string const& instance_id,
                      std::string const& backup_id) {
      google::cloud::spanner::Backup backup(
          google::cloud::spanner::Instance(project_id, instance_id), backup_id);
      auto status = client.DeleteBackup(backup.FullName());
      if (!status.ok()) throw std::move(status);
      std::cout << "Backup " << backup.FullName() << " was deleted.\n";
    }

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    use Google\Cloud\Spanner\SpannerClient;
    
    /**
     * Delete a backup.
     * Example:
     * ```
     * delete_backup($instanceId, $backupId);
     * ```
     * @param string $instanceId The Spanner instance ID.
     * @param string $backupId The Spanner backup ID.
     */
    function delete_backup(string $instanceId, string $backupId): void
    {
        $spanner = new SpannerClient();
        $instance = $spanner->instance($instanceId);
        $backup = $instance->backup($backupId);
        $backupName = $backup->name();
        $backup->delete();
        print("Backup $backupName deleted" . PHP_EOL);
    }

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    def delete_backup(instance_id, backup_id):
        spanner_client = spanner.Client()
        instance = spanner_client.instance(instance_id)
        backup = instance.backup(backup_id)
        backup.reload()
    
        # Wait for databases that reference this backup to finish optimizing.
        while backup.referencing_databases:
            time.sleep(30)
            backup.reload()
    
        # Delete the backup.
        backup.delete()
    
        # Verify that the backup is deleted.
        assert backup.exists() is False
        print("Backup {} has been deleted.".format(backup.name))

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
    
    database_admin_client.delete_backup name: backup_path
    puts "Backup #{backup_id} deleted"

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=cloudspanner) .
