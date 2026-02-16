List all operations running on the database and list the percentage complete for each operation.

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
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
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void listDatabaseOperations(
    InstanceAdminClient instanceAdminClient,
    DatabaseAdminClient dbAdminClient,
    InstanceId instanceId) {
  Instance instance = instanceAdminClient.getInstance(instanceId.getInstance());
  // Get optimize restored database operations.
  Timestamp last24Hours = Timestamp.ofTimeSecondsAndNanos(TimeUnit.SECONDS.convert(
      TimeUnit.HOURS.convert(Timestamp.now().getSeconds(), TimeUnit.SECONDS) - 24,
      TimeUnit.HOURS), 0);
  String filter = String.format("(metadata.@type:type.googleapis.com/"
                  + "google.spanner.admin.database.v1.OptimizeRestoredDatabaseMetadata) AND "
                  + "(metadata.progress.start_time > \"%s\")", last24Hours);
  for (Operation op : instance.listDatabaseOperations(Options.filter(filter)).iterateAll()) {
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
// const instanceId = 'my-instance';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner Database Admin Client object
const databaseAdminClient = spanner.getDatabaseAdminClient();

// List database operations
try {
  const [databaseOperations] =
    await databaseAdminClient.listDatabaseOperations({
      parent: databaseAdminClient.instancePath(projectId, instanceId),
      filter:
        '(metadata.@type:type.googleapis.com/google.spanner.admin.database.v1.OptimizeRestoredDatabaseMetadata)',
    });
  console.log('Optimize Database Operations:');
  databaseOperations.forEach(databaseOperation => {
    const metadata =
      protos.google.spanner.admin.database.v1.OptimizeRestoredDatabaseMetadata.decode(
        databaseOperation.metadata.value,
      );
    console.log(
      `Database ${metadata.name} restored from backup is ` +
        `${metadata.progress.progressPercent}% optimized.`,
    );
  });
} catch (err) {
  console.error('ERROR:', err);
}
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\ListDatabaseOperationsRequest;
use Google\Cloud\Spanner\Admin\Database\V1\OptimizeRestoredDatabaseMetadata;

/**
 * List all optimize restored database operations in an instance.
 * Example:
 * ```
 * list_database_operations($instanceId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 */
function list_database_operations(string $projectId, string $instanceId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $parent = DatabaseAdminClient::instanceName($projectId, $instanceId);

    $filter = '(metadata.@type:type.googleapis.com/' .
                'google.spanner.admin.database.v1.OptimizeRestoredDatabaseMetadata)';
    $operations = $databaseAdminClient->listDatabaseOperations(
        new ListDatabaseOperationsRequest([
            'parent' => $parent,
            'filter' => $filter
        ])
    );

    foreach ($operations->iterateAllElements() as $operation) {
        $obj = new OptimizeRestoredDatabaseMetadata();
        $meta = $operation->getMetadata()->unpack($obj);
        $progress = $meta->getProgress()->getProgressPercent();
        $dbName = basename($meta->getName());
        printf('Database %s restored from backup is %d%% optimized.' . PHP_EOL, $dbName, $progress);
    }
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def list_database_operations(instance_id):
    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    # List the progress of restore.
    filter_ = (
        "(metadata.@type:type.googleapis.com/"
        "google.spanner.admin.database.v1.OptimizeRestoredDatabaseMetadata)"
    )
    request = spanner_database_admin.ListDatabaseOperationsRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        filter=filter_,
    )
    operations = database_admin_api.list_database_operations(request)
    for op in operations:
        metadata = protobuf_helpers.from_any_pb(
            spanner_database_admin.OptimizeRestoredDatabaseMetadata, op.metadata
        )
        print(
            "Database {} restored from backup is {}% optimized.".format(
                metadata.name, metadata.progress.progress_percent
            )
        )
```

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` ruby
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
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
