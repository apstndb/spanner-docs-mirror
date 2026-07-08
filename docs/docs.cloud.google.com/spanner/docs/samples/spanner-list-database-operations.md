---
name: documents/docs.cloud.google.com/spanner/docs/samples/spanner-list-database-operations
uri: https://docs.cloud.google.com/spanner/docs/samples/spanner-list-database-operations
title: List database operations
description: List all operations running on the database and list the percentage complete for each operation.
data_source: docs.cloud.google.com
---

List all operations running on the database and list the percentage complete for each operation.

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    void ListDatabaseOperations(
        google::cloud::spanner_admin::DatabaseAdminClient client,
        std::string const& project_id, std::string const& instance_id) {
      google::cloud::spanner::Instance in(project_id, instance_id);
      google::spanner::admin::database::v1::ListDatabaseOperationsRequest request;
      request.set_parent(in.FullName());
      request.set_filter(
          "(metadata.@type:type.googleapis.com/"
          "google.spanner.admin.database.v1.OptimizeRestoredDatabaseMetadata)");
      for (auto& operation : client.ListDatabaseOperations(request)) {
        if (!operation) throw std::move(operation).status();
        google::spanner::admin::database::v1::OptimizeRestoredDatabaseMetadata
            metadata;
        operation->metadata().UnpackTo(&metadata);
        std::cout << "Database " << metadata.name() << " restored from backup is "
                  << metadata.progress().progress_percent() << "% optimized.\n";
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
    using System.Linq;
    
    public class ListDatabaseOperationsSample
    {
        public IEnumerable<Operation> ListDatabaseOperations(string projectId, string instanceId)
        {
            // Create the DatabaseAdminClient instance.
            DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();
    
            var filter = "(metadata.@type:type.googleapis.com/google.spanner.admin.database.v1.OptimizeRestoredDatabaseMetadata)";
    
            ListDatabaseOperationsRequest request = new ListDatabaseOperationsRequest
            {
                ParentAsInstanceName = InstanceName.FromProjectInstance(projectId, instanceId),
                Filter = filter
            };
    
            // List the optimize restored databases operations on the instance.
            var operations = databaseAdminClient.ListDatabaseOperations(request);
    
            // We print the first 5 elements for demonstration purposes.
            // You can print all operations in the sequence by removing the call to Take(5).
            // The sequence will lazily fetch elements in pages as needed.
            foreach (var operation in operations.Take(5))
            {
                OptimizeRestoredDatabaseMetadata metadata =
                    operation.Metadata.Unpack<OptimizeRestoredDatabaseMetadata>();
                Console.WriteLine(
                    $"Database {metadata.Name} restored from backup is {metadata.Progress.ProgressPercent}% optimized.");
            }
    
            return operations;
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
    
    func listDatabaseOperations(ctx context.Context, w io.Writer, db string) error {
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
     // List the databases that are being optimized after a restore operation.
     filter := "(metadata.@type:type.googleapis.com/google.spanner.admin.database.v1.OptimizeRestoredDatabaseMetadata)"
     iter := adminClient.ListDatabaseOperations(ctx, &adminpb.ListDatabaseOperationsRequest{
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
         metadata := &adminpb.OptimizeRestoredDatabaseMetadata{}
         if err := ptypes.UnmarshalAny(resp.Metadata, metadata); err != nil {
             return err
         }
         fmt.Fprintf(w, "Database %s restored from backup is %d%% optimized.\n",
             metadata.Name,
             metadata.Progress.ProgressPercent,
         )
     }
     return nil
    }

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    static void listDatabaseOperations(
        DatabaseAdminClient dbAdminClient, String projectId, String instanceId) {
      // Get optimize restored database operations.
      com.google.cloud.Timestamp last24Hours = com.google.cloud.Timestamp.ofTimeSecondsAndNanos(
          TimeUnit.SECONDS.convert(
              TimeUnit.HOURS.convert(com.google.cloud.Timestamp.now().getSeconds(), TimeUnit.SECONDS)
                  - 24,
              TimeUnit.HOURS), 0);
      String filter = String.format("(metadata.@type:type.googleapis.com/"
          + "google.spanner.admin.database.v1.OptimizeRestoredDatabaseMetadata) AND "
          + "(metadata.progress.start_time > \"%s\")", last24Hours);
      ListDatabaseOperationsRequest listDatabaseOperationsRequest =
          ListDatabaseOperationsRequest.newBuilder()
              .setParent(com.google.spanner.admin.instance.v1.InstanceName.of(
                  projectId, instanceId).toString()).setFilter(filter).build();
      ListDatabaseOperationsPagedResponse pagedResponse
          = dbAdminClient.listDatabaseOperations(listDatabaseOperationsRequest);
      for (Operation op : pagedResponse.iterateAll()) {
        try {
          OptimizeRestoredDatabaseMetadata metadata =
              op.getMetadata().unpack(OptimizeRestoredDatabaseMetadata.class);
          System.out.println(String.format(
              "Database %s restored from backup is %d%% optimized",
              metadata.getName(),
              metadata.getProgress().getProgressPercent()));
        } catch (InvalidProtocolBufferException e) {
          // The returned operation does not contain OptimizeRestoredDatabaseMetadata.
          System.err.println(e.getMessage());
        }
      }
    }

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    use Google\Cloud\Spanner\SpannerClient;
    
    /**
     * List all optimize restored database operations in an instance.
     * Example:
     * ```
     * list_database_operations($instanceId);
     * ```
     *
     * @param string $instanceId The Spanner instance ID.
     */
    function list_database_operations(string $instanceId): void
    {
        $spanner = new SpannerClient();
        $instance = $spanner->instance($instanceId);
    
        // List the databases that are being optimized after a restore operation.
        $filter = '(metadata.@type:type.googleapis.com/' .
                  'google.spanner.admin.database.v1.OptimizeRestoredDatabaseMetadata)';
    
        $operations = $instance->databaseOperations(['filter' => $filter]);
    
        foreach ($operations as $operation) {
            if (!$operation->done()) {
                $meta = $operation->info()['metadata'];
                $dbName = basename($meta['name']);
                $progress = $meta['progress']['progressPercent'];
                printf('Database %s restored from backup is %d%% optimized.' . PHP_EOL, $dbName, $progress);
            }
        }
    }

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    def list_database_operations(instance_id):
        spanner_client = spanner.Client()
        instance = spanner_client.instance(instance_id)
    
        # List the progress of restore.
        filter_ = (
            "(metadata.@type:type.googleapis.com/"
            "google.spanner.admin.database.v1.OptimizeRestoredDatabaseMetadata)"
        )
        operations = instance.list_database_operations(filter_=filter_)
        for op in operations:
            print(
                "Database {} restored from backup is {}% optimized.".format(
                    op.metadata.name, op.metadata.progress.progress_percent
                )
            )

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    # project_id  = "Your Google Cloud project ID"
    # instance_id = "Your Spanner instance ID"
    
    require "google/cloud/spanner"
    require "google/cloud/spanner/admin/database"
    
    database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin
    instance_path = database_admin_client.instance_path project: project_id, instance: instance_id
    jobs = database_admin_client.list_database_operations parent: instance_path,
                                                          filter: "metadata.@type:type.googleapis.com/google.spanner.admin.database.v1.OptimizeRestoredDatabaseMetadata"
    
    jobs.each do |job|
      if job.error?
        puts job.error
      elsif job.results
        progress_percent = job.metadata.progress.progress_percent
        puts "Database #{job.results.name} restored from backup is #{progress_percent}% optimized"
      end
    end
    
    puts "List database operations with optimized database filter found #{jobs.count} jobs."

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=cloudspanner) .
