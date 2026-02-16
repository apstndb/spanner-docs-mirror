Create an instance.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Create and manage instances](/spanner/docs/create-manage-instances)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void CreateInstance(google::cloud::spanner_admin::InstanceAdminClient client,
                    std::string const& project_id,
                    std::string const& instance_id,
                    std::string const& display_name,
                    std::string const& config_id) {
  namespace spanner = ::google::cloud::spanner;
  spanner::Instance in(project_id, instance_id);

  auto project = google::cloud::Project(project_id);
  std::string config_name =
      project.FullName() + "/instanceConfigs/" + config_id;
  auto instance =
      client
          .CreateInstance(spanner::CreateInstanceRequestBuilder(in, config_name)
                              .SetDisplayName(display_name)
                              .SetNodeCount(1)
                              .SetLabels({{"cloud_spanner_samples", "true"}})
                              .Build())
          .get();
  if (!instance) throw std::move(instance).status();
  std::cout << "Created instance [" << in << "]:\n" << instance->DebugString();
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Api.Gax.ResourceNames;
using Google.Cloud.Spanner.Admin.Instance.V1;
using Google.Cloud.Spanner.Common.V1;
using Google.LongRunning;
using System;
using System.Threading.Tasks;

public class CreateInstanceAsyncSample
{
    public async Task<Instance> CreateInstanceAsync(
        string projectId,
        string instanceId,
        Instance.Types.Edition edition = Instance.Types.Edition.Standard)
    {
        // Create the InstanceAdminClient instance.
        InstanceAdminClient instanceAdminClient = await InstanceAdminClient.CreateAsync();

        // Initialize request parameters.
        Instance instance = new Instance
        {
            InstanceName = InstanceName.FromProjectInstance(projectId, instanceId),
            ConfigAsInstanceConfigName = InstanceConfigName.FromProjectInstanceConfig(projectId, "regional-us-central1"),
            DisplayName = "This is a display name.",
            NodeCount = 1,
            Labels =
            {
                { "cloud_spanner_samples", "true" },
            },
            Edition = edition,
        };
        ProjectName projectName = ProjectName.FromProject(projectId);

        // Make the CreateInstance request.
        Operation<Instance, CreateInstanceMetadata> response = await instanceAdminClient.CreateInstanceAsync(projectName, instanceId, instance);

        Console.WriteLine("Waiting for the operation to finish.");

        // Poll until the returned long-running operation is complete.
        Operation<Instance, CreateInstanceMetadata> completedResponse = await response.PollUntilCompletedAsync();

        if (completedResponse.IsFaulted)
        {
            Console.WriteLine($"Error while creating instance: {completedResponse.Exception}");
            throw completedResponse.Exception;
        }

        Console.WriteLine($"Instance created successfully.");

        return completedResponse.Result;
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

 instance "cloud.google.com/go/spanner/admin/instance/apiv1"
 "cloud.google.com/go/spanner/admin/instance/apiv1/instancepb"
)

func createInstance(w io.Writer, projectID, instanceID string) error {
 // projectID := "my-project-id"
 // instanceID := "my-instance"
 ctx := context.Background()
 instanceAdmin, err := instance.NewInstanceAdminClient(ctx)
 if err != nil {
     return err
 }
 defer instanceAdmin.Close()

 op, err := instanceAdmin.CreateInstance(ctx, &instancepb.CreateInstanceRequest{
     Parent:     fmt.Sprintf("projects/%s", projectID),
     InstanceId: instanceID,
     Instance: &instancepb.Instance{
         Config:      fmt.Sprintf("projects/%s/instanceConfigs/%s", projectID, "regional-us-central1"),
         DisplayName: instanceID,
         NodeCount:   1,
         Labels:      map[string]string{"cloud_spanner_samples": "true"},
         Edition:     instancepb.Instance_STANDARD,
     },
 })
 if err != nil {
     return fmt.Errorf("could not create instance %s: %w", fmt.Sprintf("projects/%s/instances/%s", projectID, instanceID), err)
 }
 // Wait for the instance creation to finish.
 i, err := op.Wait(ctx)
 if err != nil {
     return fmt.Errorf("waiting for instance creation to finish failed: %w", err)
 }
 // The instance may not be ready to serve yet.
 if i.State != instancepb.Instance_READY {
     fmt.Fprintf(w, "instance state is not READY yet. Got state %v\n", i.State)
 }
 fmt.Fprintf(w, "Created instance [%s]\n", instanceID)
 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.instance.v1.InstanceAdminClient;
import com.google.spanner.admin.instance.v1.CreateInstanceRequest;
import com.google.spanner.admin.instance.v1.Instance;
import com.google.spanner.admin.instance.v1.InstanceConfigName;
import com.google.spanner.admin.instance.v1.ProjectName;
import java.util.concurrent.ExecutionException;

class CreateInstanceExample {

  static void createInstance() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    createInstance(projectId, instanceId);
  }

  static void createInstance(String projectId, String instanceId) {
    // Set Instance configuration.
    int nodeCount = 2;
    String displayName = "Descriptive name";

    // Create an Instance object that will be used to create the instance.
    Instance instance =
        Instance.newBuilder()
            .setDisplayName(displayName)
            .setEdition(Instance.Edition.STANDARD)
            .setNodeCount(nodeCount)
            .setConfig(InstanceConfigName.of(projectId, "regional-us-east4").toString())
            .build();

    try (Spanner spanner =
            SpannerOptions.newBuilder().setProjectId(projectId).build().getService();
        InstanceAdminClient instanceAdminClient = spanner.createInstanceAdminClient()) {

      // Wait for the createInstance operation to finish.
      Instance createdInstance =
          instanceAdminClient
              .createInstanceAsync(
                  CreateInstanceRequest.newBuilder()
                      .setParent(ProjectName.of(projectId).toString())
                      .setInstanceId(instanceId)
                      .setInstance(instance)
                      .build())
              .get();
      System.out.printf("Instance %s was successfully created%n", createdInstance.getName());
    } catch (ExecutionException e) {
      System.out.printf(
          "Error: Creating instance %s failed with error message %s%n",
          instance.getName(), e.getMessage());
    } catch (InterruptedException e) {
      System.out.println("Error: Waiting for createInstance operation to finish was interrupted");
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

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const instanceAdminClient = await spanner.getInstanceAdminClient();
/**
 * TODO(developer): Uncomment the following lines before running the sample.
 **/
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';

// Creates a new instance
try {
  console.log(
    `Creating instance ${instanceAdminClient.instancePath(
      projectId,
      instanceId,
    )}.`,
  );
  const [operation] = await instanceAdminClient.createInstance({
    instanceId: instanceId,
    parent: instanceAdminClient.projectPath(projectId),
    instance: {
      config: instanceAdminClient.instanceConfigPath(
        projectId,
        'regional-us-central1',
      ),
      nodeCount: 1,
      displayName: 'Display name for the instance.',
      labels: {
        cloud_spanner_samples: 'true',
        created: Math.round(Date.now() / 1000).toString(), // current time
      },
      edition:
        protos.google.spanner.admin.instance.v1.Instance.Edition.STANDARD, //optional
    },
  });

  console.log(`Waiting for operation on ${instanceId} to complete...`);
  await operation.promise();

  console.log(`Created instance ${instanceId}.`);
} catch (err) {
  console.error('ERROR:', err);
}
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\Admin\Instance\V1\Client\InstanceAdminClient;
use Google\Cloud\Spanner\Admin\Instance\V1\CreateInstanceRequest;
use Google\Cloud\Spanner\Admin\Instance\V1\Instance;

/**
 * Creates an instance.
 * Example:
 * ```
 * create_instance($projectId, $instanceId);
 * ```
 *
 * @param string $projectId  The Spanner project ID.
 * @param string $instanceId The Spanner instance ID.
 */
function create_instance(string $projectId, string $instanceId): void
{
    $instanceAdminClient = new InstanceAdminClient();
    $parent = InstanceAdminClient::projectName($projectId);
    $instanceName = InstanceAdminClient::instanceName($projectId, $instanceId);
    $configName = $instanceAdminClient->instanceConfigName($projectId, 'regional-us-central1');
    $instance = (new Instance())
        ->setName($instanceName)
        ->setConfig($configName)
        ->setDisplayName('dispName')
        ->setNodeCount(1);

    $operation = $instanceAdminClient->createInstance(
        (new CreateInstanceRequest())
        ->setParent($parent)
        ->setInstanceId($instanceId)
        ->setInstance($instance)
    );

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf('Created instance %s' . PHP_EOL, $instanceId);
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def create_instance(instance_id):
    """Creates an instance."""
    from google.cloud.spanner_admin_instance_v1.types import spanner_instance_admin

    spanner_client = spanner.Client()

    config_name = "{}/instanceConfigs/regional-us-central1".format(
        spanner_client.project_name
    )

    operation = spanner_client.instance_admin_api.create_instance(
        parent=spanner_client.project_name,
        instance_id=instance_id,
        instance=spanner_instance_admin.Instance(
            config=config_name,
            display_name="This is a display name.",
            node_count=1,
            labels={
                "cloud_spanner_samples": "true",
                "sample_name": "snippets-create_instance-explicit",
                "created": str(int(time.time())),
            },
            edition=spanner_instance_admin.Instance.Edition.STANDARD,  # Optional
        ),
    )

    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)

    print("Created instance {}".format(instance_id))
```

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# instance_config_id = "Your Spanner InstanceConfig ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/instance"

instance_admin_client = Google::Cloud::Spanner::Admin::Instance.instance_admin

project_path = instance_admin_client.project_path project: project_id
instance_path = instance_admin_client.instance_path project: project_id, instance: instance_id
instance_config_path = instance_admin_client.instance_config_path project: project_id, instance_config: instance_config_id

job = instance_admin_client.create_instance parent: project_path,
                                            instance_id: instance_id,
                                            instance: { name: instance_path,
                                                        config: instance_config_path,
                                                        display_name: instance_id,
                                                        node_count: 2,
                                                        labels: { cloud_spanner_samples: "true" } }

puts "Waiting for create instance operation to complete"

job.wait_until_done!

if job.error?
  puts job.error
else
  puts "Created instance #{instance_id}"
end
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
