This page describes how to list, create, update, delete, and show the details of a Spanner [instance configuration](/spanner/docs/instance-configurations) . Some custom instance configuration functionalities are not available in the Google Cloud console UI. In those cases, use the Google Cloud CLI (gcloud) commands provided.

### List instance configurations

You can list all the available Spanner instance configurations with the [gcloud CLI](/spanner/docs/gcloud-spanner) and client libraries. To find a list of all Spanner instance configurations, see [Regional and multi-region configurations](/spanner/docs/instance-configurations) .

### gcloud

Run the `  gcloud spanner instance-configs list  ` command:

``` text
gcloud spanner instance-configs list
```

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` cpp
void ListInstanceConfigs(
    google::cloud::spanner_admin::InstanceAdminClient client,
    std::string const& project_id) {
  int count = 0;
  auto project = google::cloud::Project(project_id);
  for (auto& config : client.ListInstanceConfigs(project.FullName())) {
    if (!config) throw std::move(config).status();
    ++count;
    std::cout << "Instance config [" << count << "]:\n"
              << config->DebugString();
  }
  if (count == 0) {
    std::cout << "No instance configs found in project " << project_id << "\n";
  }
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` csharp
using Google.Api.Gax.ResourceNames;
using Google.Cloud.Spanner.Admin.Instance.V1;
using System;
using System.Collections.Generic;
using System.Linq;

public class ListInstanceConfigsSample
{
    public IEnumerable<InstanceConfig> ListInstanceConfigs(string projectId)
    {
        var instanceAdminClient = InstanceAdminClient.Create();
        var projectName = ProjectName.FromProject(projectId);
        var instanceConfigs = instanceAdminClient.ListInstanceConfigs(projectName);

        // We print the first 5 elements for demonstration purposes.
        // You can print all configs in the sequence by removing the call to Take(5).
        // The sequence will lazily fetch elements in pages as needed.
        foreach (var instanceConfig in instanceConfigs.Take(5))
        {
            Console.WriteLine($"Available leader options for instance config {instanceConfig.InstanceConfigName.InstanceConfigId}:");
            foreach (var leader in instanceConfig.LeaderOptions)
            {
                Console.WriteLine(leader);
            }

            Console.WriteLine($"Available optional replica for instance config {instanceConfig.InstanceConfigName.InstanceConfigId}:");
            foreach (var optionalReplica in instanceConfig.OptionalReplicas)
            {
                Console.WriteLine($"Replica type - {optionalReplica.Type}, default leader location - {optionalReplica.DefaultLeaderLocation}, location - {optionalReplica.Location}");
            }
        }
        return instanceConfigs;
    }
}
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` go
import (
 "context"
 "fmt"
 "io"

 instance "cloud.google.com/go/spanner/admin/instance/apiv1"
 "cloud.google.com/go/spanner/admin/instance/apiv1/instancepb"
 "google.golang.org/api/iterator"
)

// istInstanceConfigs gets available leader options for all instances
func listInstanceConfigs(w io.Writer, projectName string) error {
 // projectName = `projects/<project>
 ctx := context.Background()
 instanceAdmin, err := instance.NewInstanceAdminClient(ctx)
 if err != nil {
     return err
 }
 defer instanceAdmin.Close()

 request := &instancepb.ListInstanceConfigsRequest{
     Parent: projectName,
 }
 for {
     iter := instanceAdmin.ListInstanceConfigs(ctx, request)
     for {
         ic, err := iter.Next()
         if err == iterator.Done {
             break
         }
         if err != nil {
             return err
         }
         fmt.Fprintf(w, "Available leader options for instance config %s: %v\n", ic.Name, ic.LeaderOptions)
     }
     pageToken := iter.PageInfo().Token
     if pageToken == "" {
         break
     } else {
         request.PageToken = pageToken
     }
 }

 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.instance.v1.InstanceAdminClient;
import com.google.spanner.admin.instance.v1.InstanceConfig;
import com.google.spanner.admin.instance.v1.ProjectName;

public class ListInstanceConfigsSample {

  static void listInstanceConfigs() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    listInstanceConfigs(projectId);
  }

  static void listInstanceConfigs(String projectId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder()
            .setProjectId(projectId)
            .build()
            .getService();
        InstanceAdminClient instanceAdminClient = spanner.createInstanceAdminClient()) {
      final ProjectName projectName = ProjectName.of(projectId);
      for (InstanceConfig instanceConfig :
          instanceAdminClient.listInstanceConfigs(projectName).iterateAll()) {
        System.out.printf(
            "Available leader options for instance config %s: %s%n",
            instanceConfig.getName(),
            instanceConfig.getLeaderOptionsList()
        );
      }
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` javascript
/**
 * TODO(developer): Uncomment the following line before running the sample.
 */
// const projectId = 'my-project-id';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const instanceAdminClient = spanner.getInstanceAdminClient();

async function listInstanceConfigs() {
  // Lists all available instance configurations in the project.
  // See https://cloud.google.com/spanner/docs/instance-configurations#configuration for a list of all available
  // configurations.
  const [instanceConfigs] = await instanceAdminClient.listInstanceConfigs({
    parent: instanceAdminClient.projectPath(projectId),
  });
  console.log(`Available instance configs for project ${projectId}:`);
  instanceConfigs.forEach(instanceConfig => {
    console.log(
      `Available leader options for instance config ${
        instanceConfig.name
      } ('${instanceConfig.displayName}'): 
         ${instanceConfig.leaderOptions.join()}`,
    );
  });
}
listInstanceConfigs();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

```` php
use Google\Cloud\Spanner\Admin\Instance\V1\Client\InstanceAdminClient;
use Google\Cloud\Spanner\Admin\Instance\V1\ListInstanceConfigsRequest;

/**
 * Lists the available instance configurations.
 * Example:
 * ```
 * list_instance_configs();
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 */
function list_instance_configs(string $projectId): void
{
    $instanceAdminClient = new InstanceAdminClient();
    $projectName = InstanceAdminClient::projectName($projectId);
    $request = new ListInstanceConfigsRequest();
    $request->setParent($projectName);
    $resp = $instanceAdminClient->listInstanceConfigs($request);
    foreach ($resp as $element) {
        printf(
            'Available leader options for instance config %s: %s' . PHP_EOL,
            $element->getDisplayName(),
            implode(',', iterator_to_array($element->getLeaderOptions()))
        );
    }
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` python
def list_instance_config():
    """Lists the available instance configurations."""
    from google.cloud.spanner_admin_instance_v1.types import spanner_instance_admin

    spanner_client = spanner.Client()

    request = spanner_instance_admin.ListInstanceConfigsRequest(
        parent=spanner_client.project_name
    )
    for config in spanner_client.instance_admin_api.list_instance_configs(
        request=request
    ):
        print(
            "Available leader options for instance config {}: {}".format(
                config.name, config.leader_options
            )
        )
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` ruby
# project_id  = "Your Google Cloud project ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/instance"

instance_admin_client = Google::Cloud::Spanner::Admin::Instance.instance_admin

project_path = instance_admin_client.project_path project: project_id
configs = instance_admin_client.list_instance_configs parent: project_path

configs.each do |c|
  puts "Available leader options for instance config #{c.name} : #{c.leader_options}"
end
```

### Show instance configuration details

You can show the details of any instance configuration with the [gcloud CLI](/spanner/docs/gcloud-spanner) and client libraries. When you create a new [custom instance configuration](/spanner/docs/instance-configurations#configuration) , you can add any location listed under `  optionalReplicas  ` as an optional replica. If you don't see your desired read-only replica location, you can [request a new optional read-only replica region](https://docs.google.com/forms/d/e/1FAIpQLSfw9Rj4p4KA8oLu7MhIpSyPRd-4qxqazwsFZIY-_tkNrpWFcw/viewform) .

For more information, see [Create a custom instance configuration](/spanner/docs/create-manage-configurations#create-configuration) .

### gcloud

Run the `  gcloud spanner instance-configs describe  ` command:

``` text
gcloud spanner instance-configs describe INSTANCE-CONFIG
```

Provide the following value:

  - `  INSTANCE-CONFIG  `  
    The instance configuration, which defines the geographic location of the instance and affects how data is replicated. For example, `  eur6  ` or `  regional-us-central1  ` .

To show the details of the `  eur6  ` base configuration, run:

``` text
gcloud spanner instance-configs describe eur6
```

Here's an example output for the `  eur6  ` base configuration:

``` text
  configType: GOOGLE_MANAGED
  displayName: Europe (Netherlands, Frankfurt)
  freeInstanceAvailability: UNSUPPORTED
  leaderOptions:
  - europe-west3
  - europe-west4
  name: projects/cloud-spanner-demo/instanceConfigs/eur6
  optionalReplicas:
  - displayName: South Carolina
    labels:
      cloud.googleapis.com/country: US
      cloud.googleapis.com/location: us-east1
      cloud.googleapis.com/region: us-east1
    location: us-east1
    type: READ_ONLY
  - displayName: South Carolina
    labels:
      cloud.googleapis.com/country: US
      cloud.googleapis.com/location: us-east1
      cloud.googleapis.com/region: us-east1
    location: us-east1
    type: READ_ONLY
  replicas:
  - defaultLeaderLocation: true
    location: europe-west4
    type: READ_WRITE
  - location: europe-west4
    type: READ_WRITE
  - location: europe-west3
    type: READ_WRITE
  - location: europe-west3
    type: READ_WRITE
  - location: europe-west6
    type: WITNESS
```

Additional usage notes:

  - `  baseConfig  ` (for custom configurations only) points to the base instance configuration. Refer to [available regional configurations](/spanner/docs/instance-configurations#available-configurations-regional) and [available multi-region configurations](/spanner/docs/instance-configurations#available-configurations-multi-region) for lists of base instance configurations that can be used to create a custom instance configuration.
  - `  configType  ` indicates whether this instance configuration is a base instance configuration or a custom instance configuration.
  - `  etag  ` (for custom configurations only) is a base64-encoded string representation of the configuration. It is used for optimistic concurrency control.

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` cpp
void GetInstanceConfig(google::cloud::spanner_admin::InstanceAdminClient client,
                       std::string const& project_id,
                       std::string const& config_id) {
  auto project = google::cloud::Project(project_id);
  auto config = client.GetInstanceConfig(project.FullName() +
                                         "/instanceConfigs/" + config_id);
  if (!config) throw std::move(config).status();
  std::cout << "The instanceConfig " << config->name()
            << " exists and its metadata is:\n"
            << config->DebugString();
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` csharp
using Google.Cloud.Spanner.Admin.Instance.V1;
using System;
using System.Threading.Tasks;

public class GetInstanceConfigAsyncSample
{
    public async Task<InstanceConfig> GetInstanceConfigAsync(string projectId, string instanceConfigId)
    {
        var instanceAdminClient = await InstanceAdminClient.CreateAsync();
        var instanceConfigName = InstanceConfigName.FromProjectInstanceConfig(projectId, instanceConfigId);
        var instanceConfig = await instanceAdminClient.GetInstanceConfigAsync(instanceConfigName);

        Console.WriteLine($"Available leader options for instance config {instanceConfigName.InstanceConfigId}:");
        foreach (var leader in instanceConfig.LeaderOptions)
        {
            Console.WriteLine(leader);
        }
        return instanceConfig;
    }
}
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` go
import (
 "context"
 "fmt"
 "io"

 instance "cloud.google.com/go/spanner/admin/instance/apiv1"
 "cloud.google.com/go/spanner/admin/instance/apiv1/instancepb"
)

// getInstanceConfig gets available leader options
func getInstanceConfig(w io.Writer, instanceConfigName string) error {
 // defaultLeader = `nam3`
 ctx := context.Background()
 instanceAdmin, err := instance.NewInstanceAdminClient(ctx)
 if err != nil {
     return err
 }
 defer instanceAdmin.Close()

 ic, err := instanceAdmin.GetInstanceConfig(ctx, &instancepb.GetInstanceConfigRequest{
     Name: instanceConfigName,
 })

 if err != nil {
     return fmt.Errorf("could not get instance config %s: %w", instanceConfigName, err)
 }

 fmt.Fprintf(w, "Available leader options for instance config %s: %v", instanceConfigName, ic.LeaderOptions)

 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.instance.v1.InstanceAdminClient;
import com.google.spanner.admin.instance.v1.InstanceConfig;
import com.google.spanner.admin.instance.v1.InstanceConfigName;

public class GetInstanceConfigSample {

  static void getInstanceConfig() {
    // TODO(developer): Replace these variables before running the sample.
    final String projectId = "my-project";
    final String instanceConfigId = "nam6";
    getInstanceConfig(projectId, instanceConfigId);
  }

  static void getInstanceConfig(String projectId, String instanceConfigId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder()
            .setProjectId(projectId)
            .build()
            .getService();
        InstanceAdminClient instanceAdminClient = spanner.createInstanceAdminClient()) {
      final InstanceConfigName instanceConfigName = InstanceConfigName.of(projectId,
          instanceConfigId);

      final InstanceConfig instanceConfig =
          instanceAdminClient.getInstanceConfig(instanceConfigName.toString());

      System.out.printf(
          "Available leader options for instance config %s: %s%n",
          instanceConfig.getName(),
          instanceConfig.getLeaderOptionsList()
      );
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` javascript
/**
 * TODO(developer): Uncomment the following line before running the sample.
 */
// const projectId = 'my-project-id';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const instanceAdminClient = spanner.getInstanceAdminClient();

async function getInstanceConfig() {
  // Get the instance config for the multi-region North America 6 (NAM6).
  // See https://cloud.google.com/spanner/docs/instance-configurations#configuration for a list of all available
  // configurations.
  const [instanceConfig] = await instanceAdminClient.getInstanceConfig({
    name: instanceAdminClient.instanceConfigPath(projectId, 'nam6'),
  });
  console.log(
    `Available leader options for instance config ${instanceConfig.name} ('${
      instanceConfig.displayName
    }'): 
         ${instanceConfig.leaderOptions.join()}`,
  );
}
getInstanceConfig();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` php
use Google\Cloud\Spanner\Admin\Instance\V1\Client\InstanceAdminClient;
use Google\Cloud\Spanner\Admin\Instance\V1\GetInstanceConfigRequest;

/**
 * Gets the leader options for the instance configuration.
 *
 * @param string $projectId The Google Cloud Project ID.
 * @param string $instanceConfig The name of the instance configuration.
 */
function get_instance_config(string $projectId, string $instanceConfig): void
{
    $instanceAdminClient = new InstanceAdminClient();
    $instanceConfigName = InstanceAdminClient::instanceConfigName($projectId, $instanceConfig);

    $request = (new GetInstanceConfigRequest())
        ->setName($instanceConfigName);
    $configInfo = $instanceAdminClient->getInstanceConfig($request);

    printf('Available leader options for instance config %s: %s' . PHP_EOL,
        $instanceConfig,
        implode(',', array_keys(iterator_to_array($configInfo->getLeaderOptions())))
    );
}
```

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` python
def get_instance_config(instance_config):
    """Gets the leader options for the instance configuration."""
    spanner_client = spanner.Client()
    config_name = "{}/instanceConfigs/{}".format(
        spanner_client.project_name, instance_config
    )
    config = spanner_client.instance_admin_api.get_instance_config(name=config_name)
    print(
        "Available leader options for instance config {}: {}".format(
            instance_config, config.leader_options
        )
    )
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_config_id = "Spanner instance config ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/instance"

instance_admin_client = Google::Cloud::Spanner::Admin::Instance.instance_admin

instance_config_path = instance_admin_client.instance_config_path \
  project: project_id, instance_config: instance_config_id
config = instance_admin_client.get_instance_config name: instance_config_path

puts "Available leader options for instance config #{config.name} : #{config.leader_options}"
```

## Create a custom instance configuration

The instance configurations with predefined regions and replication topologies are called *base instance configurations* . You can't change the replication topology of base instance configurations.

You can create a *custom instance configuration* , which lets you add additional optional [read-only replicas](/spanner/docs/replication#read-only) to the predefined base instance configuration. This scales reads and supports low latency stale reads. The added read-only replica must be in a region that isn't part of the existing instance configuration. For a list of the optional read-only regions that you can use to create a custom instance configuration, see the Optional Region column under [Regional available configurations](/spanner/docs/instance-configurations#available-configurations-regional) and [Multi-region available configurations](/spanner/docs/instance-configurations#available-configurations-multi-region) .

You can't create a custom [dual-region instance configuration](/spanner/docs/instance-configurations#dual-region-configurations) . For more information about Spanner replication and replica types, see [Replication](/spanner/docs/replication) .

To create a custom instance configuration, you must have the `  spanner.instanceConfigs.create  ` permission. By default, roles that have the `  spanner.instances.create  ` permission will also have the `  spanner.instanceConfigs.create  ` permission.

### Console

You can't create a custom instance configuration using the Google Cloud console. To create an instance with read-only replicas, use the [gcloud CLI](/spanner/docs/create-manage-configurations#gcloud_2) or client libraries.

### gcloud

Use the `  gcloud spanner instance-configs create  ` command:

``` text
gcloud spanner instance-configs create CUSTOM-INSTANCE-CONFIG-ID  \
 --display-name=DISPLAY-NAME \
 --base-config=BASE-CONFIG \
 --labels=KEY=VALUE,[...] \
 --replicas=location=LOCATION,type=TYPE[:...]
```

You can use the `  --clone-config  ` flag as a convenient way to clone another base or custom instance configuration while also declaring the location and type of a specific custom replica.

``` text
  gcloud spanner instance-configs create CUSTOM-INSTANCE-CONFIG-ID  \
  --display-name=DISPLAY-NAME \
  --clone-config=INSTANCE-CONFIG \
  --labels=KEY=VALUE,[...] \
  --add-replicas=location=LOCATION,type=TYPE[:...] \
  --skip-replicas=location=LOCATION,type=TYPE[:...]
```

Provide the following values:

  - `  CUSTOM-INSTANCE-CONFIG-ID  `  
    A permanent identifier that is unique within your Google Cloud project. You can't change the instance configuration ID later. The `  custom-  ` prefix is required to avoid name conflicts with base instance configurations.
  - `  DISPLAY-NAME  `  
    The name to display for the custom instance configuration in the Google Cloud console.
  - `  BASE-CONFIG  `  
    The region name of the base instance configuration on which your custom instance configuration is based. For example, `  eur6  ` or `  regional-us-central1  ` .
  - `  LOCATION  `  
    The region name of the serving resources (replicas), for example, `  us-east1  ` . To find out what location names are accepted, run `  gcloud spanner instance-configs describe INSTANCE-CONFIG  ` and refer to the `  replicas  ` and `  optionalReplicas  ` lists.
  - `  TYPE  `  
    The type of replica. To find out what corresponding locations and replica types are accepted, run `  gcloud spanner instance-configs describe INSTANCE-CONFIG  ` and refer to the `  replicas  ` and `  optionalReplicas  ` lists. The types are one of the following:
      - READ\_ONLY
      - READ\_WRITE
      - WITNESS
    Items in the list are separated by ":".
    Unless the `  --[clone-config]  ` flag is used, all replica `  LOCATION  ` and `  TYPE  ` must be specified when creating a custom instance configuration, including the ones predefined in the base configuration. For more information, see the [gcloud instance-configs describe help-text](/sdk/gcloud/reference/spanner/instance-configs/create) .

If you choose to use the flags `  --clone-config  ` and `  --add-replicas  ` (only use `  --skip-replicas  ` if there are replicas you want to skip from being cloned), provide the following values:

  - `  --clone-config= INSTANCE-CONFIG  `
    
    Use this flag as a convenient way to clone another base or custom instance configuration while also declaring the location and type of a specific custom replica. Then use `  --add-replicas=location= LOCATION ,type= TYPE  ` to specify where you want to add your optional replica.
    
    For example, to create a custom instance configuration with two read-only replicas in `  us-east1  ` while copying all the other replica locations from the `  eur6  ` base instance configuration, run:
    
    ``` text
    gcloud spanner instance-configs create custom-eur6 --clone-config=eur6 \
    --add-replicas=location=us-east1,type=READ_ONLY:location=us-east1,type=READ_ONLY
    ```

  - `  --skip-replicas=location= LOCATION ,type= TYPE  `
    
    Use this flag to skip any replica from being cloned.
    
    For example, to create a custom instance configuration with one read-only replica in `  us-east4  ` while copying all the other replica locations from the `  nam3  ` base instance configuration except the read-only replica in `  us-central1  ` , run:
    
    ``` text
    gcloud spanner instance-configs create custom-nam3 --clone-config=nam3 \
      --add-replicas=location=us-east4,type=READ_ONLY \
      --skip-replicas=location=us-central1,type=READ_ONLY
    ```

The following flags and values are optional:

  - `  --labels= KEY=VALUE,[...]  `
    
    `  KEY  ` and `  VALUE  ` : A list of key and value pairs to add to your custom instance configuration.
    
    Keys must start with a lowercase character and contain only hyphens (-), underscores (\_), lowercase characters, and numbers. Values must contain only hyphens (-), underscores (\_), lowercase characters, and numbers.

  - `  --validate-only  `
    
    Use this flag to validate that the request will succeed before executing it.

For example, to create a custom instance configuration with the `  eur6  ` base configuration with one additional read-only replica in `  us-east1  ` , run:

``` text
gcloud spanner instance-configs create custom-eur6 \
  --display-name="Custom eur6" --clone-config=eur6 \
  --add-replicas=location=us-east1,type=READ_ONLY \
```

You can also create a custom instance configuration without the `  --clone-config  ` flag:

``` text
gcloud spanner instance-configs create custom-eur6 \
  --display-name="Custom eur6" --base-config=eur6 \
  --replicas=location=europe-west4,type=READ_WRITE:location=europe-west3,type=READ_WRITE:location=europe-west4,type=READ_WRITE:location=europe-west3,type=READ_WRITE:location=europe-west6,type=WITNESS:location=us-east1,type=READ_ONLY
```

You should see the following output:

``` text
Creating instance-config...done.
```

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` cpp
void CreateInstanceConfig(
    google::cloud::spanner_admin::InstanceAdminClient client,
    std::string const& project_id, std::string const& user_config_id,
    std::string const& base_config_id) {
  auto project = google::cloud::Project(project_id);
  auto base_config = client.GetInstanceConfig(
      project.FullName() + "/instanceConfigs/" + base_config_id);
  if (!base_config) throw std::move(base_config).status();
  if (base_config->optional_replicas().empty()) {
    throw std::runtime_error("No optional replicas in base config");
  }
  google::spanner::admin::instance::v1::CreateInstanceConfigRequest request;
  request.set_parent(project.FullName());
  request.set_instance_config_id(user_config_id);
  auto* request_config = request.mutable_instance_config();
  request_config->set_name(project.FullName() + "/instanceConfigs/" +
                           user_config_id);
  request_config->set_display_name("My instance config");
  // The user-managed instance config must contain all the replicas
  // of the base config plus at least one of the optional replicas.
  *request_config->mutable_replicas() = base_config->replicas();
  for (auto const& replica : base_config->optional_replicas()) {
    *request_config->add_replicas() = replica;
  }
  request_config->set_base_config(base_config->name());
  *request_config->mutable_leader_options() = base_config->leader_options();
  request.set_validate_only(false);
  auto user_config = client.CreateInstanceConfig(request).get();
  if (!user_config) throw std::move(user_config).status();
  std::cout << "Created instance config [" << user_config_id << "]:\n"
            << user_config->DebugString();
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` csharp
using System;
using System.Threading.Tasks;
using Google.Api.Gax.ResourceNames;
using Google.Cloud.Spanner.Admin.Instance.V1;

public class CreateInstanceConfigAsyncSample
{
    public async Task<InstanceConfig> CreateInstanceConfigAsync(string projectId, string baseInstanceConfigId, string customInstanceConfigId)
    {
        InstanceAdminClient instanceAdminClient = await InstanceAdminClient.CreateAsync();
        var instanceConfigName = new InstanceConfigName(projectId, baseInstanceConfigId);
        var instanceConfig = await instanceAdminClient.GetInstanceConfigAsync(instanceConfigName);

        var customInstanceConfigName = new InstanceConfigName(projectId, customInstanceConfigId);

        //Create a custom config.
        InstanceConfig userInstanceConfig = new InstanceConfig
        {
            DisplayName = "C# test custom instance config",
            ConfigType = InstanceConfig.Types.Type.UserManaged,
            BaseConfigAsInstanceConfigName = instanceConfigName,
            InstanceConfigName = customInstanceConfigName,
            //The replicas for the custom instance configuration must include all the replicas of the base
            //configuration, in addition to at least one from the list of optional replicas of the base
            //configuration.
            Replicas = { instanceConfig.Replicas },
            OptionalReplicas =
            {
                new ReplicaInfo
                {
                    Type = ReplicaInfo.Types.ReplicaType.ReadOnly,
                    Location = "us-east1",
                    DefaultLeaderLocation = false
                },
                //The replicas for the custom instance configuration must include all the replicas of the base
                //configuration, in addition to at least one from the list of optional replicas of the base
                //configuration.
                instanceConfig.OptionalReplicas
            }
        };

        var operationResult = await instanceAdminClient.CreateInstanceConfigAsync(new CreateInstanceConfigRequest
        {
            ParentAsProjectName = ProjectName.FromProject(projectId),
            InstanceConfig = userInstanceConfig,
            InstanceConfigId = customInstanceConfigId
        });

        var pollResult = await operationResult.PollUntilCompletedAsync();

        if (pollResult.IsFaulted)
        {
            throw pollResult.Exception;
        }

        Console.WriteLine($"Instance config created successfully");
        Console.WriteLine($"Available custom replication options for instance config {pollResult.Result.Name}\r\n{pollResult.Result.OptionalReplicas}");

        return pollResult.Result;
    }
}
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` go
import (
 "context"
 "fmt"
 "io"
 "time"

 instance "cloud.google.com/go/spanner/admin/instance/apiv1"
 "cloud.google.com/go/spanner/admin/instance/apiv1/instancepb"
)

// createInstanceConfig creates a custom spanner instance config
func createInstanceConfig(w io.Writer, projectID, userConfigID, baseConfigID string) error {
 // projectID := "my-project-id"
 // userConfigID := "custom-config", custom config names must start with the prefix “custom-”.
 // baseConfigID := "my-base-config"

 // Add timeout to context.
 ctx, cancel := context.WithTimeout(context.Background(), 10*time.Minute)
 defer cancel()

 adminClient, err := instance.NewInstanceAdminClient(ctx)
 if err != nil {
     return err
 }
 defer admiadminClient.ClosebaseConfig, err := adminClient.GetInstanceConfig(ctx, &instancepb.GetInstanceConfigRequest{
     Name: fmt.Sprintf("projects/%s/instanceConfigs/%s", projectID, baseConfigID),
 })
 if err != nil {
     return fmt.Errorf("createInstanceConfig.GetInstanceConfig: %w", err)
 }
 if baseConfig.OptionalReplicas == nil || len(baseConfig.OptionalReplicas) == 0 {
     return fmt.Errorf("CreateInstanceConfig expects base config with at least from the list of optional replicas")
 }
 op, err := adminClient.CreateInstanceConfig(ctx, &instancepb.CreateInstanceConfigRequest{
     Parent: fmt.Sprintf("projects/%s", projectID),
     // Custom config names must start with the prefix “custom-”.
     InstanceConfigId: userConfigID,
     InstanceConfig: &instancepb.InstanceConfig{
         Name:        fmt.Sprintf("projects/%s/instanceConfigs/%s", projectID, userConfigID),
         DisplayName: "custom-golang-samples",
         ConfigType:  instanceinstancepb.InstanceConfig_USER_MANAGEDThe replicas for the custom instance configuration must include all the replicas of the base
         // configuration, in addition to at least one from the list of optional replicas of the base
         // configuration.
         Replicas:   append(baseConfig.Replicas, baseConfig.OptionalReplicas...),
         BaseConfig: baseConfig.Name,
         Labels:     map[string]string{"go_cloud_spanner_samples": "true"},
     },
 })
 if err != nil {
     return err
 }
 fmt.Fprintf(w, "Waiting for create operation on projects/%s/instanceConfigs/%s to complete...\n", projectID, userConfigID)
 // Wait for the instance configuration creation to finish.
 i, err := op.Wait(ctx)
 if err != nil {
     return fmt.Errorf("Waiting for instance config creation to finish failed: %w", err)
 }
 // The instance configuration may not be ready to serve yet.
 if i.State != instanceinstancepb.InstanceConfig_READY.Fprintf(w, "InstanceConfig state is not READY yet. Got state %v\n", i.State)
 }
 fmt.Fprintf(w, "Created instance configuration [%s]\n", userConfigID)
 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.instance.v1.InstanceAdminClient;
import com.google.spanner.admin.instance.v1.CreateInstanceConfigRequest;
import com.google.spanner.admin.instance.v1.InstanceConfig;
import com.google.spanner.admin.instance.v1.InstanceConfigName;
import com.google.spanner.admin.instance.v1.ProjectName;
import com.google.spanner.admin.instance.v1.ReplicaInfo;
import java.util.List;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;
import java.util.stream.Collectors;
import java.util.stream.Stream;

class CreateInstanceConfigSample {

  static void createInstanceConfig() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String baseInstanceConfigId = "nam11";
    String instanceConfigId = "custom-instance-config4";

    createInstanceConfig(projectId, baseInstanceConfigId, instanceConfigId);
  }

  static void createInstanceConfig(
      String projectId, String baseInstanceConfigId, String instanceConfigId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder()
            .setProjectId(projectId)
            .build()
            .getService();
        InstanceAdminClient instanceAdminClient = spanner.createInstanceAdminClient()) {
      final InstanceConfigName baseInstanceConfigName = InstanceConfigName.of(projectId,
          baseInstanceConfigId);
      final InstanceConfig baseConfig =
          instanceAdminClient.getInstanceConfig(baseInstanceConfigName.toString());
      final InstanceConfigName instanceConfigName = InstanceConfigName.of(projectId,
          instanceConfigId);
      /**
       * The replicas for the custom instance configuration must include all the replicas of the
       * base configuration, in addition to at least one from the list of optional replicas of the
       * base configuration.
       */
      final List<ReplicaInfo> replicas =
          Stream.concat(baseConfig.getReplicasList().stream(),
              baseConfig.getOptionalReplicasList().stream().limit(1)).collect(Collectors.toList());
      final InstanceConfig instanceConfig =
          InstanceConfig.newBuilder().setName(instanceConfigName.toString())
              .setBaseConfig(baseInstanceConfigName.toString())
              .setDisplayName("Instance Configuration").addAllReplicas(replicas).build();
      final CreateInstanceConfigRequest createInstanceConfigRequest =
          CreateInstanceConfigRequest.newBuilder().setParent(ProjectName.of(projectId).toString())
              .setInstanceConfigId(instanceConfigId).setInstanceConfig(instanceConfig).build();
      try {
        System.out.printf("Waiting for create operation for %s to complete...\n",
            instanceConfigName);
        InstanceConfig instanceConfigResult =
            instanceAdminClient.createInstanceConfigAsync(
                createInstanceConfigRequest).get(5, TimeUnit.MINUTES);
        System.out.printf("Created instance configuration %s\n", instanceConfigResult.getName());
      } catch (ExecutionException | TimeoutException e) {
        System.out.printf(
            "Error: Creating instance configuration %s failed with error message %s\n",
            instanceConfig.getName(), e.getMessage());
      } catch (InterruptedException e) {
        System.out.println(
            "Error: Waiting for createInstanceConfig operation to finish was interrupted");
      }
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` javascript
/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const instanceConfigId = 'custom-my-instance-config-id'
// const baseInstanceConfigId = 'my-base-instance-config-id';
// const projectId = 'my-project-id';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const instanceAdminClient = spanner.getInstanceAdminClient();

// Creates a new instance config
async function createInstanceConfig() {
  const [baseInstanceConfig] = await instanceAdminClient.getInstanceConfig({
    name: instanceAdminClient.instanceConfigPath(
      projectId,
      baseInstanceConfigId,
    ),
  });
  try {
    console.log(
      `Creating instance config ${instanceAdminClient.instanceConfigPath(
        projectId,
        instanceConfigId,
      )}.`,
    );
    const [operation] = await instanceAdminClient.createInstanceConfig({
      instanceConfigId: instanceConfigId,
      parent: instanceAdminClient.projectPath(projectId),
      instanceConfig: {
        name: instanceAdminClient.instanceConfigPath(
          projectId,
          instanceConfigId,
        ),
        baseConfig: instanceAdminClient.instanceConfigPath(
          projectId,
          baseInstanceConfigId,
        ),
        displayName: instanceConfigId,
        replicas: baseInstanceConfig.replicas.concat(
          baseInstanceConfig.optionalReplicas[0],
        ),
      },
    });
    console.log(
      `Waiting for create operation for ${instanceConfigId} to complete...`,
    );
    await operation.promise();
    console.log(`Created instance config ${instanceConfigId}.`);
  } catch (err) {
    console.error(
      'ERROR: Creating instance config ',
      instanceConfigId,
      ' failed with error message ',
      err,
    );
  }
}
createInstanceConfig();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

```` php
use Google\Cloud\Spanner\Admin\Instance\V1\CreateInstanceConfigRequest;
use Google\Cloud\Spanner\Admin\Instance\V1\GetInstanceConfigRequest;
use Google\Cloud\Spanner\Admin\Instance\V1\InstanceConfig;
use Google\Cloud\Spanner\Admin\Instance\V1\Client\InstanceAdminClient;
use Google\Cloud\Spanner\Admin\Instance\V1\ReplicaInfo;

/**
 * Creates a customer managed instance configuration.
 * Example:
 * ```
 * create_instance_config($instanceConfigId);
 * ```
 *
 * @param string $projectId The Google Cloud Project ID.
 * @param string $instanceConfigId The customer managed instance configuration id. The id must start with 'custom-'.
 * @param string $baseConfigId Base configuration ID to be used for creation, e.g. nam11.
 */
function create_instance_config(string $projectId, string $instanceConfigId, string $baseConfigId): void
{
    $instanceAdminClient = new InstanceAdminClient();
    $projectName = InstanceAdminClient::projectName($projectId);
    $instanceConfigName = $instanceAdminClient->instanceConfigName(
        $projectId,
        $instanceConfigId
    );

    // Get a Google Managed instance configuration to use as the base for our custom instance configuration.
    $baseInstanceConfig = $instanceAdminClient->instanceConfigName(
        $projectId,
        $baseConfigId
    );

    $request = new GetInstanceConfigRequest(['name' => $baseInstanceConfig]);
    $baseInstanceConfigInfo = $instanceAdminClient->getInstanceConfig($request);

    $instanceConfig = (new InstanceConfig())
        ->setBaseConfig($baseInstanceConfig)
        ->setName($instanceConfigName)
        ->setDisplayName('My custom instance configuration')
        ->setLabels(['php-cloud-spanner-samples' => true])
        ->setReplicas(array_merge(
            iterator_to_array($baseInstanceConfigInfo->getReplicas()),
            [new ReplicaInfo([
            'location' => 'us-east1',
            'type' => ReplicaInfo\ReplicaType::READ_ONLY,
            'default_leader_location' => false
            ])]
        ));

    $request = new CreateInstanceConfigRequest([
        'parent' => $projectName,
        'instance_config' => $instanceConfig,
        'instance_config_id' => $instanceConfigId
    ]);
    $operation = $instanceAdminClient->createInstanceConfig($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf('Created instance configuration %s' . PHP_EOL, $instanceConfigId);
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` python
def create_instance_config(user_config_name, base_config_id):
    """Creates the new user-managed instance configuration using base instance config."""

    # user_config_name = `custom-nam11`
    # base_config_id = `projects/<project>/instanceConfigs/nam11`
    spanner_client = spanner.Client()
    base_config = spanner_client.instance_admin_api.get_instance_config(
        name=base_config_id
    )

    # The replicas for the custom instance configuration must include all the replicas of the base
    # configuration, in addition to at least one from the list of optional replicas of the base
    # configuration.
    replicas = []
    for replica in base_config.replicas:
        replicas.append(replica)
    replicas.append(base_config.optional_replicas[0])
    operation = spanner_client.instance_admin_api.create_instance_config(
        parent=spanner_client.project_name,
        instance_config_id=user_config_name,
        instance_config=spanner_instance_admin.InstanceConfig(
            name="{}/instanceConfigs/{}".format(
                spanner_client.project_name, user_config_name
            ),
            display_name="custom-python-samples",
            config_type=spanner_instance_admin.InstanceConfig.Type.USER_MANAGED,
            replicas=replicas,
            base_config=base_config.name,
            labels={"python_cloud_spanner_samples": "true"},
        ),
    )
    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)

    print("Created instance configuration {}".format(user_config_name))
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` ruby
require "google/cloud/spanner"
require "google/cloud/spanner/admin/instance"

def spanner_create_instance_config project_id:, user_config_name:, base_config_id:
  # project_id  = "Your Google Cloud project ID"
  # user_config_name = "Your custom instance configuration name, The name must start with 'custom-'"
  # base_config_id = "Base configuration ID to be used for creation, e.g projects/<project>/instanceConfigs/nam11"

  instance_admin_client = Google::Cloud::Spanner::Admin::Instance.instance_admin
  project_path = instance_admin_client.project_path project: project_id
  base_instance_config = instance_admin_client.get_instance_config name: base_config_id
  # The replicas for the custom instance configuration must include all the replicas of the base
  # configuration, in addition to at least one from the list of optional replicas of the base
  # configuration.
  custom_replicas = []
  base_instance_config.replicas.each do |replica|
    custom_replicas << replica
  end
  custom_replicas << base_instance_config.optional_replicas[0]
  custom_instance_config_id = instance_admin_client.instance_config_path \
    project: project_id, instance_config: user_config_name
  custom_instance_config = {
    name: custom_instance_config_id,
    display_name: "custom-ruby-samples",
    config_type: :USER_MANAGED,
    replicas: custom_replicas,
    base_config: base_config_id,
    labels: { ruby_cloud_spanner_samples: "true" }
  }
  request = {
    parent: project_path,
    # Custom config names must start with the prefix “custom-”.
    instance_config_id: user_config_name,
    instance_config: custom_instance_config
  }
  job = instance_admin_client.create_instance_config request

  puts "Waiting for create instance config operation to complete"

  job.wait_until_done!

  if job.error?
    puts job.error
  else
    puts "Created instance configuration #{user_config_name}"
  end
end
```

## Create an instance in a custom instance configuration

You can create an instance in a custom instance configuration.

### Console

To create an instance in a custom instance configuration, use the [gcloud CLI](/spanner/docs/create-manage-configurations#gcloud_2) or client libraries.

### gcloud

After you [create the custom instance configuration](#create-configuration) , follow the instructions provided in [Create an instance](/spanner/docs/create-manage-instances#gcloud) .

### C++

After you [create the custom instance configuration](#create-configuration) , follow the instructions provided in [Create an instance](/spanner/docs/create-manage-instances#c++) .

### C\#

After you [create the custom instance configuration](#create-configuration) , follow the instructions provided in [Create an instance](/spanner/docs/create-manage-instances#c) .

### Go

After you [create the custom instance configuration](#create-configuration) , follow the instructions provided in [Create an instance](/spanner/docs/create-manage-instances#go) .

### Java

After you [create the custom instance configuration](#create-configuration) , follow the instructions provided in [Create an instance](/spanner/docs/create-manage-instances#java) .

### Node.js

After you [create the custom instance configuration](#create-configuration) , follow the instructions provided in [Create an instance](/static/spanner/docs/create-manage-instances#node.js) .

### PHP

After you [create the custom instance configuration](#create-configuration) , follow the instructions provided in [Create an instance](/spanner/docs/create-manage-instances#php) .

### Python

After you [create the custom instance configuration](#create-configuration) , follow the instructions provided in [Create an instance](/spanner/docs/create-manage-instances#python) .

### Ruby

After you [create the custom instance configuration](#create-configuration) , follow the instructions provided in [Create an instance](/spanner/docs/create-manage-instances#ruby) .

## Update a custom instance configuration

You can change the display name and labels of a custom instance configuration.

You cannot change or update the replicas of your custom instance configuration. However, you can create a new custom instance configuration with additional replicas, then move your instance to the new custom instance configuration with your chosen additional replicas. For example, if your instance is in `  regional-us-central1  ` and you want to add a read-only replica `  us-west1  ` , then you need to create a new custom instance configuration with `  regional-us-central1  ` as the base configuration and add `  us-west1  ` as a read-only replica. Then [move your instance](/spanner/docs/move-instance) to this new custom instance configuration.

### gcloud

Use the `  gcloud spanner instance-configs update  ` command:

``` text
gcloud spanner instance-configs update CUSTOM-INSTANCE-CONFIG-ID \
  --display-name=NEW-DISPLAY-NAME \
  --update-labels=KEY=VALUE,[...], \
  --etag=ETAG
```

Provide the following values:

  - `  CUSTOM-INSTANCE-CONFIG-ID  `  
    A permanent identifier of your custom instance configuration. It will start with `  custom-  ` .
  - `  NEW-DISPLAY-NAME  `  
    The new name to display for the instance configuration in the Google Cloud console.
  - `  KEY  ` and `  VALUE  `  
    A list of key and value pairs to update.
    Keys must start with a lowercase character and contain only hyphens (-), underscores (\_), lowercase characters, and numbers. Values must contain only hyphens (-), underscores (\_), lowercase characters, and numbers.

The following flags and values are optional:

  - `  --etag= ETAG  ` : The `  ETAG  ` argument can be used to select and skip simultaneous updates in a read-modify-write scenario.
  - `  --validate-only  ` : Use this flag to validate that the request will succeed before executing it.

For example:

``` text
gcloud spanner instance-configs update custom-eur6 \
  --display-name="Customer managed europe replicas"
```

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` cpp
void UpdateInstanceConfig(
    google::cloud::spanner_admin::InstanceAdminClient client,
    std::string const& project_id, std::string const& config_id) {
  auto project = google::cloud::Project(project_id);
  auto config = client.GetInstanceConfig(project.FullName() +
                                         "/instanceConfigs/" + config_id);
  if (!config) throw std::move(config).status();
  google::spanner::admin::instance::v1::UpdateInstanceConfigRequest request;
  auto* request_config = request.mutable_instance_config();
  request_config->set_name(config->name());
  request_config->mutable_labels()->insert({"key", "value"});
  request.mutable_update_mask()->add_paths("labels");
  request_config->set_etag(config->etag());
  request.set_validate_only(false);
  auto updated_config = client.UpdateInstanceConfig(request).get();
  if (!updated_config) throw std::move(updated_config).status();
  std::cout << "Updated instance config [" << config_id << "]:\n"
            << updated_config->DebugString();
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Google.Cloud.Spanner.Admin.Instance.V1;
using Google.LongRunning;
using Google.Protobuf.WellKnownTypes;

public class UpdateInstanceConfigAsyncSample
{
    public async Task<Operation<InstanceConfig, UpdateInstanceConfigMetadata>> UpdateInstanceConfigAsync(string projectId, string instanceConfigId)
    {
        InstanceAdminClient instanceAdminClient = await InstanceAdminClient.CreateAsync();
        var instanceConfigName = new InstanceConfigName(projectId, instanceConfigId);

        var instanceConfig = new InstanceConfig();
        instanceConfig.InstanceConfigName  = instanceConfigName;

        instanceConfig.DisplayName = "New display name";
        instanceConfig.Labels.Add(new Dictionary<string, string>
        {
            {"cloud_spanner_samples","true"},
            {"updated","true"},
        });

        var updateInstanceConfigOperation = await instanceAdminClient.UpdateInstanceConfigAsync(new UpdateInstanceConfigRequest
        {
            InstanceConfig = instanceConfig,
            UpdateMask = new FieldMask
            {
                Paths = { "display_name", "labels" }
            }
        });

        updateInstanceConfigOperation = await updateInstanceConfigOperation.PollUntilCompletedAsync(); 

        if (updateInstanceConfigOperation.IsFaulted)
        {
            throw updateInstanceConfigOperation.Exception;
        }

        Console.WriteLine("Update Instance Config operation completed successfully.");
        return updateInstanceConfigOperation;
    }
}
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` go
import (
 "context"
 "fmt"
 "io"
 "time"

 instance "cloud.google.com/go/spanner/admin/instance/apiv1"
 "cloud.google.com/go/spanner/admin/instance/apiv1/instancepb"
 "google.golang.org/genproto/protobuf/field_mask"
)

// updateInstanceConfig updates the custom spanner instance config
func updateInstanceConfig(w io.Writer, projectID, userConfigID string) error {
 // projectID := "my-project-id"
 // userConfigID := "custom-config", custom config names must start with the prefix “custom-”.

 // Add timeout to context.
 ctx, cancel := context.WithTimeout(context.Background(), 10*time.Minute)
 defer cancel()

 adminClient, err := instance.NewInstanceAdminClient(ctx)
 if err != nil {
     return err
 }
 defer admiadminClient.Closeconfig, err := adminClient.GetInstanceConfig(ctx, &instancepb.GetInstanceConfigRequest{
     Name: fmt.Sprintf("projects/%s/instanceConfigs/%s", projectID, userConfigID),
 })
 if err != nil {
     return fmt.Errorf("updateInstanceConfig.GetInstanceConfig: %w", err)
 }
 config.DisplayName = "updated custom instance config"
 config.Labels["updated"] = "true"
 op, err := adminClient.UpdateInstanceConfig(ctx, &instancepb.UpdateInstanceConfigRequest{
     InstanceConfig: config,
     UpdateMask: &field_mask.FieldMask{
         Paths: []string{"display_name", "labels"},
     },
     ValidateOnly: false,
 })
 if err != nil {
     return err
 }
 fmt.Fprintf(w, "Waiting for update operation on %s to complete...\n", userConfigID)
 // Wait for the instance configuration creation to finish.
 i, err := op.Wait(ctx)
 if err != nil {
     return fmt.Errorf("Waiting for instance config creation to finish failed: %w", err)
 }
 // The instance configuration may not be ready to serve yet.
 if i.State != instinstancepb.InstanceConfig_READY fmt.Fprintf(w, "InstanceConfig state is not READY yet. Got state %v\n", i.State)
 }
 fmt.Fprintf(w, "Updated instance configuration [%s]\n", config.Name)
 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.instance.v1.InstanceAdminClient;
import com.google.common.collect.ImmutableList;
import com.google.protobuf.FieldMask;
import com.google.spanner.admin.instance.v1.InstanceConfig;
import com.google.spanner.admin.instance.v1.InstanceConfigName;
import com.google.spanner.admin.instance.v1.UpdateInstanceConfigRequest;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

class UpdateInstanceConfigSample {

  static void updateInstanceConfig() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceConfigId = "custom-instance-config";
    updateInstanceConfig(projectId, instanceConfigId);
  }

  static void updateInstanceConfig(String projectId, String instanceConfigId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder()
            .setProjectId(projectId)
            .build()
            .getService();
        InstanceAdminClient instanceAdminClient = spanner.createInstanceAdminClient()) {
      final InstanceConfigName instanceConfigName =
          InstanceConfigName.of(projectId, instanceConfigId);
      final InstanceConfig instanceConfig =
          InstanceConfig.newBuilder()
              .setName(instanceConfigName.toString())
              .setDisplayName("updated custom instance config")
              .putLabels("updated", "true").build();
      /**
       * The field mask must always be specified; this prevents any future
       * fields in [InstanceConfig][google.spanner.admin.instance.v1.InstanceConfig]
       * from being erased accidentally by clients that do not know about them.
       */
      final UpdateInstanceConfigRequest updateInstanceConfigRequest =
          UpdateInstanceConfigRequest.newBuilder()
              .setInstanceConfig(instanceConfig)
              .setUpdateMask(
                  FieldMask.newBuilder().addAllPaths(ImmutableList.of("display_name", "labels"))
                      .build()).build();
      try {
        System.out.printf("Waiting for update operation on %s to complete...\n",
            instanceConfigName);
        InstanceConfig instanceConfigResult =
            instanceAdminClient.updateInstanceConfigAsync(
                updateInstanceConfigRequest).get(5, TimeUnit.MINUTES);
        System.out.printf(
            "Updated instance configuration %s with new display name %s\n",
            instanceConfigResult.getName(), instanceConfig.getDisplayName());
      } catch (ExecutionException | TimeoutException e) {
        System.out.printf(
            "Error: Updating instance config %s failed with error message %s\n",
            instanceConfig.getName(), e.getMessage());
        e.printStackTrace();
      } catch (InterruptedException e) {
        System.out.println(
            "Error: Waiting for updateInstanceConfig operation to finish was interrupted");
      }
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` javascript
/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const instanceConfigId = 'custom-my-instance-config-id';
// const projectId = 'my-project-id';

// Imports the Google Cloud client library
const {Spanner, protos} = require('@google-cloud/spanner');

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const instanceAdminClient = spanner.getInstanceAdminClient();

async function updateInstanceConfig() {
  // Updates an instance config
  try {
    console.log(
      `Updating instance config ${instanceAdminClient.instanceConfigPath(
        projectId,
        instanceConfigId,
      )}.`,
    );
    const [operation] = await instanceAdminClient.updateInstanceConfig({
      instanceConfig: {
        name: instanceAdminClient.instanceConfigPath(
          projectId,
          instanceConfigId,
        ),
        displayName: 'updated custom instance config',
        labels: {
          updated: 'true',
          created: Math.round(Date.now() / 1000).toString(), // current time
        },
      },
      // Field mask specifying fields that should get updated in InstanceConfig
      // Only display_name and labels can be updated
      updateMask: (protos.google.protobuf.FieldMask = {
        paths: ['display_name', 'labels'],
      }),
    });
    console.log(
      `Waiting for update operation for ${instanceConfigId} to complete...`,
    );
    await operation.promise();
    console.log(`Updated instance config ${instanceConfigId}.`);
  } catch (err) {
    console.error(
      'ERROR: Updating instance config ',
      instanceConfigId,
      ' failed with error message ',
      err,
    );
  }
}
updateInstanceConfig();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

```` php
use Google\Cloud\Spanner\Admin\Instance\V1\Client\InstanceAdminClient;
use Google\Cloud\Spanner\Admin\Instance\V1\InstanceConfig;
use Google\Cloud\Spanner\Admin\Instance\V1\UpdateInstanceConfigRequest;
use Google\Protobuf\FieldMask;

/**
 * Updates a customer managed instance configuration.
 * Example:
 * ```
 * update_instance_config($instanceConfigId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceConfigId The customer managed instance configuration id. The id must start with 'custom-'.
 */
function update_instance_config(string $projectId, string $instanceConfigId): void
{
    $instanceAdminClient = new InstanceAdminClient();

    $instanceConfigPath = $instanceAdminClient->instanceConfigName($projectId, $instanceConfigId);
    $displayName = 'New display name';

    $instanceConfig = new InstanceConfig();
    $instanceConfig->setName($instanceConfigPath);
    $instanceConfig->setDisplayName($displayName);
    $instanceConfig->setLabels(['cloud_spanner_samples' => true, 'updated' => true]);

    $fieldMask = new FieldMask();
    $fieldMask->setPaths(['display_name', 'labels']);

    $updateInstanceConfigRequest = (new UpdateInstanceConfigRequest())
        ->setInstanceConfig($instanceConfig)
        ->setUpdateMask($fieldMask);

    $operation = $instanceAdminClient->updateInstanceConfig($updateInstanceConfigRequest);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf('Updated instance configuration %s' . PHP_EOL, $instanceConfigId);
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` python
def update_instance_config(user_config_name):
    """Updates the user-managed instance configuration."""

    # user_config_name = `custom-nam11`
    spanner_client = spanner.Client()
    config = spanner_client.instance_admin_api.get_instance_config(
        name="{}/instanceConfigs/{}".format(
            spanner_client.project_name, user_config_name
        )
    )
    config.display_name = "updated custom instance config"
    config.labels["updated"] = "true"
    operation = spanner_client.instance_admin_api.update_instance_config(
        instance_config=config,
        update_mask=field_mask_pb2.FieldMask(paths=["display_name", "labels"]),
    )
    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)
    print("Updated instance configuration {}".format(user_config_name))
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` ruby
require "google/cloud/spanner"
require "google/cloud/spanner/admin/instance"

def spanner_update_instance_config user_config_id:
  # user_config_id = "The customer managed instance configuration ID, e.g projects/<project>/instanceConfigs/custom-nam11"

  instance_admin_client = Google::Cloud::Spanner::Admin::Instance.instance_admin
  config = instance_admin_client.get_instance_config name: user_config_id
  config.display_name = "updated custom instance config"
  config.labels["updated"] = "true"
  request = {
    instance_config: config,
    update_mask: { paths: ["display_name", "labels"] },
    validate_only: false
  }
  job = instance_admin_client.update_instance_config request

  puts "Waiting for update instance config operation to complete"

  job.wait_until_done!

  if job.error?
    puts job.error
  else
    puts "Updated instance configuration #{config.name}"
  end
end
```

## Delete a custom instance configuration

**Note:** Deleting an instance configuration permanently removes the instance configuration. You can't delete a custom instance configuration which is used by an instance. To delete an instance, see [Delete an instance](/spanner/docs/create-manage-instances#delete-instance) .

To delete a custom instance configuration, first delete any instance in the instance configuration.

### gcloud

Use the `  gcloud spanner instance-configs delete  ` command, replacing `  CUSTOM-INSTANCE-CONFIG-ID  ` with the custom instance configuration ID:

``` text
gcloud spanner instance-configs delete CUSTOM-INSTANCE-CONFIG-ID
```

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` cpp
void DeleteInstanceConfig(
    google::cloud::spanner_admin::InstanceAdminClient client,
    std::string const& project_id, std::string const& config_id) {
  auto project = google::cloud::Project(project_id);
  auto config_name = project.FullName() + "/instanceConfigs/" + config_id;
  auto status = client.DeleteInstanceConfig(config_name);
  if (!status.ok()) throw std::move(status);
  std::cout << "Instance config " << config_name << " successfully deleted\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` csharp
using System;
using System.Threading.Tasks;
using Google.Cloud.Spanner.Admin.Instance.V1;
using Grpc.Core;

public class DeleteInstanceConfigAsyncSample
{

    public async Task DeleteInstanceConfigAsync(string projectId, string instanceConfigId)
    {
        InstanceAdminClient instanceAdminClient = await InstanceAdminClient.CreateAsync();
        var instanceConfigName = new InstanceConfigName(projectId, instanceConfigId);

        try
        {
            await instanceAdminClient.DeleteInstanceConfigAsync(new DeleteInstanceConfigRequest
            {
                InstanceConfigName = instanceConfigName
            });
        }
        catch (RpcException ex) when (ex.Status.StatusCode == StatusCode.NotFound)
        {
            Console.WriteLine("The specified instance config does not exist. It cannot be deleted.");
            return;
        }

        Console.WriteLine("Delete Instance Config operation is completed");
    }
}
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` go
import (
 "context"
 "fmt"
 "io"

 instance "cloud.google.com/go/spanner/admin/instance/apiv1"
 "cloud.google.com/go/spanner/admin/instance/apiv1/instancepb"
)

// deleteInstanceConfig deletes the custom spanner instance config
func deleteInstanceConfig(w io.Writer, projectID, userConfigID string) error {
 // projectID := "my-project-id"
 // userConfigID := "custom-config", custom config names must start with the prefix “custom-”.

 ctx := context.Background()
 adminClient, err := instance.NewInstanceAdminClient(ctx)
 if err != nil {
     return err
 }
 defer admiadminClient.Closeerr = adminClient.DeleteInstanceConfig(ctx, &instancepb.DeleteInstanceConfigRequest{
     Name: fmt.Sprintf("projects/%s/instanceConfigs/%s", projectID, userConfigID),
 })
 if err != nil {
     return err
 }
 fmt.Fprintf(w, "Deleted instance configuration [%s]\n", userConfigID)
 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerException;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.instance.v1.InstanceAdminClient;
import com.google.spanner.admin.instance.v1.DeleteInstanceConfigRequest;
import com.google.spanner.admin.instance.v1.InstanceConfigName;

class DeleteInstanceConfigSample {

  static void deleteInstanceConfig() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceConfigId = "custom-user-config";
    deleteInstanceConfig(projectId, instanceConfigId);
  }

  static void deleteInstanceConfig(String projectId, String instanceConfigId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder()
            .setProjectId(projectId)
            .build()
            .getService();
        InstanceAdminClient instanceAdminClient = spanner.createInstanceAdminClient()) {
      final InstanceConfigName instanceConfigName = InstanceConfigName.of(projectId,
          instanceConfigId);
      final DeleteInstanceConfigRequest request =
          DeleteInstanceConfigRequest.newBuilder().setName(instanceConfigName.toString()).build();

      try {
        System.out.printf("Deleting %s...\n", instanceConfigName);
        instanceAdminClient.deleteInstanceConfig(request);
        System.out.printf("Deleted instance configuration %s\n", instanceConfigName);
      } catch (SpannerException e) {
        System.out.printf(
            "Error: Deleting instance configuration %s failed with error message: %s\n",
            instanceConfigName, e.getMessage());
      }
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` javascript
/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const instanceConfigId = 'custom-my-instance-config-id';
// const projectId = 'my-project-id';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const instanceAdminClient = spanner.getInstanceAdminClient();

async function deleteInstanceConfig() {
  // Deletes an instance config.

  try {
    // Delete the instance config.
    console.log(`Deleting ${instanceConfigId}...\n`);
    await instanceAdminClient.deleteInstanceConfig({
      name: instanceAdminClient.instanceConfigPath(
        projectId,
        instanceConfigId,
      ),
    });
    console.log(`Deleted instance config ${instanceConfigId}.\n`);
  } catch (err) {
    console.error(
      'ERROR: Deleting instance config ',
      instanceConfigId,
      ' failed with error message ',
      err,
    );
  }
}
deleteInstanceConfig();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

```` php
use Google\Cloud\Spanner\Admin\Instance\V1\Client\InstanceAdminClient;
use Google\Cloud\Spanner\Admin\Instance\V1\DeleteInstanceConfigRequest;

/**
 * Deletes a customer managed instance configuration.
 * Example:
 * ```
 * delete_instance_config($instanceConfigId);
 * ```
 *
 * @param string $projectId The Google Cloud Project ID.
 * @param string $instanceConfigId The customer managed instance configuration id. The id must start with 'custom-'.
 */
function delete_instance_config(string $projectId, string $instanceConfigId)
{
    $instanceAdminClient = new InstanceAdminClient();
    $instanceConfigName = $instanceAdminClient->instanceConfigName(
        $projectId,
        $instanceConfigId
    );

    $request = new DeleteInstanceConfigRequest();
    $request->setName($instanceConfigName);

    $instanceAdminClient->deleteInstanceConfig($request);
    printf('Deleted instance configuration %s' . PHP_EOL, $instanceConfigId);
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` python
def delete_instance_config(user_config_id):
    """Deleted the user-managed instance configuration."""
    spanner_client = spanner.Client()
    spanner_client.instance_admin_api.delete_instance_config(name=user_config_id)
    print("Instance config {} successfully deleted".format(user_config_id))
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

``` ruby
require "google/cloud/spanner"
require "google/cloud/spanner/admin/instance"

def spanner_delete_instance_config user_config_id:
  # user_config_id = "The customer managed instance configuration ID, e.g projects/<project>/instanceConfigs/custom-nam11"

  instance_admin_client = Google::Cloud::Spanner::Admin::Instance.instance_admin
  instance_admin_client.delete_instance_config name: user_config_id
  puts "Deleted instance configuration #{user_config_id}"
end
```

## What's next

  - Learn how to insert, update, and delete data with [Data Manipulation Language (DML)](/spanner/docs/dml-tasks) or the [gcloud CLI](/spanner/docs/modify-gcloud) .
  - Grant [Identity and Access Management (IAM) roles](/spanner/docs/grant-permissions) for the instance and its databases.
  - Understand how to [design a Spanner schema](/spanner/docs/schema-design) .
  - Spanner [Quotas and limits](/spanner/quotas) .
