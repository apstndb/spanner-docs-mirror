---
name: documents/docs.cloud.google.com/spanner/docs/samples/spanner-list-backup-operations
uri: https://docs.cloud.google.com/spanner/docs/samples/spanner-list-backup-operations
title: List backup operations
description: List all create backup operations running on the database and list the percentage complete for each operation.
data_source: docs.cloud.google.com
---

List all create backup operations running on the database and list the percentage complete for each operation.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Manage backups](https://docs.cloud.google.com/spanner/docs/backup/manage-backups)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    use Google\Cloud\Spanner\SpannerClient;
    
    /**
     * List all create backup operations in an instance.
     * Optionally passing the backupId will also list the
     * copy backup operations on the backup.
     *
     * @param string $instanceId The Spanner instance ID.
     * @param string $databaseId The Spanner database ID.
     * @param string $backupId The Spanner backup ID whose copy operations need to be listed.
     */
    function list_backup_operations(
        string $instanceId,
        string $databaseId,
        ?string $backupId = null
    ): void {
        $spanner = new SpannerClient();
        $instance = $spanner->instance($instanceId);
    
        // List the CreateBackup operations.
        $filter = '(metadata.@type:type.googleapis.com/' .
                  'google.spanner.admin.database.v1.CreateBackupMetadata) AND ' . "(metadata.database:$databaseId)";
    
        // See https://cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#listbackupoperationsrequest
        // for the possible filter values
        $operations = $instance->backupOperations(['filter' => $filter]);
    
        foreach ($operations as $operation) {
            if (!$operation->done()) {
                $meta = $operation->info()['metadata'];
                $backupName = basename($meta['name']);
                $dbName = basename($meta['database']);
                $progress = $meta['progress']['progressPercent'];
                printf('Backup %s on database %s is %d%% complete.' . PHP_EOL, $backupName, $dbName, $progress);
            }
        }
    
        if (is_null($backupId)) {
            return;
        }
    
        // List copy backup operations
        $filter = '(metadata.@type:type.googleapis.com/' .
                  'google.spanner.admin.database.v1.CopyBackupMetadata) AND ' . "(metadata.source_backup:$backupId)";
    
        $operations = $instance->backupOperations(['filter' => $filter]);
    
        foreach ($operations as $operation) {
            if (!$operation->done()) {
                $meta = $operation->info()['metadata'];
                $backupName = basename($meta['name']);
                $progress = $meta['progress']['progressPercent'];
                printf('Copy Backup %s on source backup %s is %d%% complete.' . PHP_EOL, $backupName, $backupId, $progress);
            }
        }
    }

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    def list_backup_operations(instance_id, database_id, backup_id):
        spanner_client = spanner.Client()
        instance = spanner_client.instance(instance_id)
    
        # List the CreateBackup operations.
        filter_ = (
            "(metadata.@type:type.googleapis.com/"
            "google.spanner.admin.database.v1.CreateBackupMetadata) "
            "AND (metadata.database:{})"
        ).format(database_id)
        operations = instance.list_backup_operations(filter_=filter_)
        for op in operations:
            metadata = op.metadata
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
        operations = instance.list_backup_operations(filter_=filter_)
        for op in operations:
            metadata = op.metadata
            print(
                "Backup {} on source backup {}: {}% complete.".format(
                    metadata.name,
                    metadata.source_backup,
                    metadata.progress.progress_percent,
                )
            )

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=cloudspanner) .
