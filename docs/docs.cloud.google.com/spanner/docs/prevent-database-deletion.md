This page describes how to protect Spanner databases from accidental deletion.

Spanner database deletion protection prevents the accidental deletion of existing databases by users or service accounts that have the necessary IAM permissions to delete the database. By enabling database deletion protection, you can safeguard databases that are important to your application and services. Use database deletion protection along with [point-in-time recovery](/spanner/docs/pitr) and [backup features](/spanner/docs/backup) to provide a comprehensive set of data protection capabilities for your Spanner databases.

By default, the deletion protection setting is disabled when you create your new database. You can [enable the deletion protection setting](#enable) after the database creation succeeds. Additionally, you can enable this setting on an existing database. If you want to protect multiple databases, enable the setting on each database individually. Enabling or disabling deletion protection doesn't have any performance impact on the database. If you need to delete a database that has database protection enabled, you need to disable the protection before you can delete the database.

## Limitations

You can't enable database deletion protection in the following scenarios:

  - If the database is being deleted.
  - If the database is being restored from a backup. (After the restore operation completes, you can enable database protection).

In addition, backups of a database and databases restored from a backup don't inherit the database deletion protection setting of their source database. After you restore a database from a backup, you must enable its database deletion protection separately.

If you delete your project, Spanner database deletion protection doesn't prevent the deletion of your database or instance. For more information about what happens when you delete your project, see [Shutting down (deleting) projects](/resource-manager/docs/creating-managing-projects#shutting_down_projects) .

## Access control with IAM

To enable the deletion protection setting of your database, you must have certain [IAM permissions](/spanner/docs/iam) .

You need to have the `  spanner.databases.update  ` permission to enable or disable database deletion protection. If you only need to view the status of your database configuration, you need to have the `  spanner.databases.list  ` or `  spanner.databases.get  ` permission. For information on how to grant Spanner IAM permissions, see [Apply IAM permissions](/spanner/docs/grant-permissions) .

If you have the predefined [Spanner Database Admin `  roles/spanner.databaseAdmin  `](/spanner/docs/iam#spanner.databaseAdmin) role for your database, you can update and enable database deletion protection.

You can enable the database deletion protection setting on an existing database to prevent the accidental deletion of the database.

## Enable database deletion protection

You can enable database deletion protection using the gcloud CLI, the client libraries, and the [REST](https://cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/patch) or [RPC](https://cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.UpdateDatabase) APIs. You can't enable database deletion protection using the Google Cloud console.

### gcloud

To enable the deletion protection setting of a database, run the following command:

``` text
  gcloud spanner databases update
  DATABASE_ID --instance=INSTANCE_ID
  --enable-drop-protection [--async]
```

The following options are required:

  - `  DATABASE_ID  `  
    ID of the database.
  - `  INSTANCE_ID  `  
    ID of the instance for the database.

The following options are optional:

  - `  --async  `  
    Return immediately, without waiting for the operation in progress to complete.

### Client libraries

### C++

``` cpp
void UpdateDatabase(google::cloud::spanner_admin::DatabaseAdminClient client,
                    std::string const& project_id,
                    std::string const& instance_id,
                    std::string const& database_id, bool drop_protection) {
  google::cloud::spanner::Database db(project_id, instance_id, database_id);
  google::spanner::admin::database::v1::Database database;
  database.set_name(db.FullName());
  database.set_enable_drop_protection(drop_protection);
  google::protobuf::FieldMask update_mask;
  update_mask.add_paths("enable_drop_protection");
  auto updated = client.UpdateDatabase(database, update_mask).get();
  if (!updated) throw std::move(updated).status();
  std::cout << "Database " << updated->name() << " successfully updated.\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using Google.Protobuf.WellKnownTypes;
using System;
using System.Threading.Tasks;

public class UpdateDatabaseAsyncSample
{
    public async Task<Database> UpdateDatabaseAsync(string projectId, string instanceId, string databaseId)
    {
        var databaseAdminClient = await DatabaseAdminClient.CreateAsync();
        var databaseName = DatabaseName.Format(projectId, instanceId, databaseId);
        Database databaseToUpdate = await databaseAdminClient.GetDatabaseAsync(databaseName);

        databaseToUpdate.EnableDropProtection = true;
        var updateDatabaseRequest = new UpdateDatabaseRequest()
        {
            Database = databaseToUpdate,
            UpdateMask = new FieldMask { Paths = { "enable_drop_protection" } }
        };

        var operation = await databaseAdminClient.UpdateDatabaseAsync(updateDatabaseRequest);

        // Wait until the operation has finished.
        Console.WriteLine("Waiting for the operation to finish.");
        var completedResponse = await operation.PollUntilCompletedAsync();

        if (completedResponse.IsFaulted)
        {
            Console.WriteLine($"Error while updating database {databaseId}: {completedResponse.Exception}");
            throw completedResponse.Exception;
        }

        Console.WriteLine($"Updated database {databaseId}.");

        // Return the updated database.
        return completedResponse.Result;
    }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
 "google.golang.org/genproto/protobuf/field_mask"
)

func updateDatabase(ctx context.Context, w io.Writer, db string) error {
 // db = `projects/<project>/instances/<instance-id>/database/<database-id>`
 // Instantiate database admin client.
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return fmt.Errorf("updateDatabase.NewDatabaseAdminClient: %w", err)
 }
 defer adminClient.Close()

 // Instantiate the request for performing update database operation.
 op, err := adminClient.UpdateDatabase(ctx, &adminpb.UpdateDatabaseRequest{
     Database: &adminpb.Database{
         Name:                 db,
         EnableDropProtection: true,
     },
     UpdateMask: &field_mask.FieldMask{
         Paths: []string{"enable_drop_protection"},
     },
 })
 if err != nil {
     return fmt.Errorf("updateDatabase.UpdateDatabase: %w", err)
 }

 // Wait for update database operation to complete.
 fmt.Fprintf(w, "Waiting for update database operation to complete [%s]\n", db)
 if _, err := op.Wait(ctx); err != nil {
     return fmt.Errorf("updateDatabase.Wait: %w", err)
 }
 fmt.Fprintf(w, "Updated database [%s]\n", db)
 return nil
}
```

### Java

``` java
import com.google.api.gax.longrunning.OperationFuture;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerExceptionFactory;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.common.collect.Lists;
import com.google.protobuf.FieldMask;
import com.google.spanner.admin.database.v1.Database;
import com.google.spanner.admin.database.v1.DatabaseName;
import com.google.spanner.admin.database.v1.UpdateDatabaseMetadata;
import com.google.spanner.admin.database.v1.UpdateDatabaseRequest;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class UpdateDatabaseSample {

  static void updateDatabase() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceId = "my-instance";
    final String databaseId = "my-database";

    updateDatabase(projectId, instanceId, databaseId);
  }

  static void updateDatabase(
      String projectId, String instanceId, String databaseId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {
      final Database database =
          Database.newBuilder()
              .setName(DatabaseName.of(projectId, instanceId, databaseId).toString())
              .setEnableDropProtection(true).build();
      final UpdateDatabaseRequest updateDatabaseRequest =
          UpdateDatabaseRequest.newBuilder()
              .setDatabase(database)
              .setUpdateMask(
                  FieldMask.newBuilder().addAllPaths(
                      Lists.newArrayList("enable_drop_protection")).build())
              .build();
      OperationFuture<Database, UpdateDatabaseMetadata> operation =
          databaseAdminClient.updateDatabaseAsync(updateDatabaseRequest);
      System.out.printf("Waiting for update operation for %s to complete...\n", databaseId);
      Database updatedDb = operation.get(5, TimeUnit.MINUTES);
      System.out.printf("Updated database %s.\n", updatedDb.getName());
    } catch (ExecutionException | TimeoutException e) {
      // If the operation failed during execution, expose the cause.
      throw SpannerExceptionFactory.asSpannerException(e.getCause());
    } catch (InterruptedException e) {
      // Throw when a thread is waiting, sleeping, or otherwise occupied,
      // and the thread is interrupted, either before or during the activity.
      throw SpannerExceptionFactory.propagateInterrupt(e);
    }
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
// const databaseId = 'my-database';
// const projectId = 'my-project-id';

// Imports the Google Cloud client library
const {Spanner, protos} = require('@google-cloud/spanner');

// creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

async function updateDatabase() {
  // Update the database metadata fields
  try {
    console.log(
      `Updating database ${databaseAdminClient.databasePath(
        projectId,
        instanceId,
        databaseId,
      )}.`,
    );
    const [operation] = await databaseAdminClient.updateDatabase({
      database: {
        name: databaseAdminClient.databasePath(
          projectId,
          instanceId,
          databaseId,
        ),
        enableDropProtection: true,
      },
      // updateMask contains the fields to be updated in database
      updateMask: (protos.google.protobuf.FieldMask = {
        paths: ['enable_drop_protection'],
      }),
    });
    console.log(
      `Waiting for update operation for ${databaseId} to complete...`,
    );
    await operation.promise();
    console.log(`Updated database ${databaseId}.`);
  } catch (err) {
    console.log('ERROR:', err);
  }
}
updateDatabase();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\Database;
use Google\Cloud\Spanner\Admin\Database\V1\GetDatabaseRequest;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseRequest;
use Google\Protobuf\FieldMask;

/**
 * Updates the drop protection setting for a database.
 * Example:
 * ```
 * update_database($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function update_database(string $projectId, string $instanceId, string $databaseId): void
{
    $newUpdateMaskField = new FieldMask([
        'paths' => ['enable_drop_protection']
    ]);
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseFullName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);
    $database = (new Database())
        ->setEnableDropProtection(true)
        ->setName($databaseFullName);

    printf('Updating database %s', $databaseId);
    $operation = $databaseAdminClient->updateDatabase((new UpdateDatabaseRequest())
        ->setDatabase($database)
        ->setUpdateMask($newUpdateMaskField));

    $operation->pollUntilComplete();

    $database = $databaseAdminClient->getDatabase(
        new GetDatabaseRequest(['name' => $databaseFullName])
    );
    printf(
        'Updated the drop protection for %s to %s' . PHP_EOL,
        $database->getName(),
        $database->getEnableDropProtection()
    );
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
def update_database(instance_id, database_id):
    """Updates the drop protection setting for a database."""
    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseRequest(
        database=spanner_database_admin.Database(
            name=database_admin_api.database_path(
                spanner_client.project, instance_id, database_id
            ),
            enable_drop_protection=True,
        ),
        update_mask={"paths": ["enable_drop_protection"]},
    )
    operation = database_admin_api.update_database(request=request)
    print(
        "Waiting for update operation for {}/databases/{} to complete...".format(
            database_admin_api.instance_path(spanner_client.project, instance_id),
            database_id,
        )
    )
    operation.result(OPERATION_TIMEOUT_SECONDS)

    print(
        "Updated database {}/databases/{}.".format(
            database_admin_api.instance_path(spanner_client.project, instance_id),
            database_id,
        )
    )
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
require "google/cloud/spanner/admin/database"

##
# This is a snippet for showcasing how to update database.
#
# @param project_id  [String] The ID of the Google Cloud project.
# @param instance_id [String] The ID of the spanner instance.
# @param database_id [String] The ID of the database.
#
def spanner_update_database project_id:, instance_id:, database_id:
  client = Google::Cloud::Spanner::Admin::Database.database_admin project_id: project_id
  db_path = client.database_path project: project_id, instance: instance_id, database: database_id
  database = client.get_database name: db_path

  puts "Updating database #{database.name}"
  database.enable_drop_protection = true
  job = client.update_database database: database, update_mask: { paths: ["enable_drop_protection"] }
  puts "Waiting for update operation for #{database.name} to complete..."
  job.wait_until_done!
  puts "Updated database #{database.name}"
end
```

## Check if a database has deletion protection enabled

You can determine if your database has its deletion protection enabled by viewing the database configuration.

### gcloud

To check if a database has deletion protection enabled, you can run the `  gcloud spanner databases describe  ` command to get detailed information about a database, or you can run the `  gcloud spanner databases list  ` to get detailed information about databases within an instance.

``` text
  gcloud spanner databases describe
  projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID
```

The following options are required:

  - `  PROJECT_ID  `  
    ID of the project for the database.
  - `  INSTANCE_ID  `  
    ID of the instance for the database.
  - `  DATABASE_ID  `  
    ID of the database.

If deletion protection is enabled, you'll see an `  enableDropProtection: true  ` parameter in the output.

## Disable database deletion protection

You can disable database deletion protection if a database no longer needs this protection or if you need to delete a database that has this setting enabled.

If you want to delete an instance that has one or more databases with deletion protection enabled, you must first disable the deletion protection on all databases in that instance before you can delete the instance.

### gcloud

To disable the deletion protection setting of a database, run the following command:

``` text
  gcloud spanner databases update
  DATABASE_ID --instance=INSTANCE_ID
  --no-enable-drop-protection [--async]
```

The following options are required:

  - `  DATABASE_ID  `  
    ID of the database.
  - `  INSTANCE_ID  `  
    ID of the instance for the database.

The following options are optional:

  - `  --async  `  
    Return immediately, without waiting for the operation in progress to complete.

## What's next

  - Learn how to [Create and manage databases](/spanner/docs/create-manage-databases) .
  - Learn more about how to [backup and restore a database](/spanner/docs/backup) .
  - Learn how to [make schema updates](/spanner/docs/schema-updates) .
