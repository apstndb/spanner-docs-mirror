List all create backup operations running on the database and list the percentage complete for each operation.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Manage backups](/spanner/docs/backup/manage-backups)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

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

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
