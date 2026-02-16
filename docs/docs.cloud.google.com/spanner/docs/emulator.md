The [gcloud CLI](/sdk) provides a local, in-memory emulator, which you can use to develop and test your applications. Because the emulator stores data only in memory, all state, including data, schema, and configs, is lost on restart. The emulator offers the same APIs as the Spanner production service and is intended for local development and testing, not for production deployments.

The emulator supports both the GoogleSQL and PostgreSQL dialects. It supports all languages of the [client libraries](#client-libraries) . You can also use the emulator with the [Google Cloud CLI](/sdk/gcloud) and [REST APIs](/spanner/docs/reference/rest) .

The emulator is also available as an open source project in [GitHub](https://github.com/GoogleCloudPlatform/cloud-spanner-emulator) .

**Note:** To explore Spanner using a 90-day free trial instance, see [Create a Spanner free trial instance](/spanner/docs/free-trial-quickstart) .

## Limitations and differences

The emulator doesn't support the following:

  - TLS/HTTPS, authentication, Identity and Access Management, permissions, or roles.
  - In the `  PLAN  ` or `  PROFILE  ` [query modes](/spanner/docs/reference/rest/v1/QueryMode) , the query plan that is returned is empty.
  - The [`  ANALYZE  ` statement](/spanner/docs/query-optimizer/overview#construct-statistics-package) . The emulator accepts but ignores it.
  - Any of the [audit logging](/spanner/docs/audit-logging) and monitoring tools.

The emulator also differs from the Spanner production service in the following ways:

  - Error messages might not be consistent between the emulator and the production service.
  - Performance and scalability for the emulator is not comparable to the production service.
  - Read-write transactions and schema changes lock the entire database for exclusive access until they are completed.
  - [Partitioned DML](/spanner/docs/dml-partitioned) and [`  partitionQuery  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/partitionQuery) are supported, but the emulator doesn't check to ensure that statements are [partitionable](/spanner/docs/dml-partitioned#partitionable-idempotent) . This means that a partitioned DML or `  partitionQuery  ` statement might run in the emulator, but it might fail in the production service with the non-partitionable statement error.

For a complete list of APIs and features that are supported, unsupported, and partially supported, see the [README](https://github.com/GoogleCloudPlatform/cloud-spanner-emulator/blob/master/README.md) file in GitHub.

## Options for running the emulator

There are two common ways to run the emulator:

  - [gcloud CLI](#emulator-for-gcloud)
  - [Docker](#install-docker)

Choose the way that is appropriate for your application development and test workflow.

**Note:** For more ways to run the emulator, see the [README](https://github.com/GoogleCloudPlatform/cloud-spanner-emulator/blob/master/README.md#quickstart) .

## Set up the emulator for gcloud CLI

For Windows and macOS users, before installing the emulator, do the following:

  - Install the [gcloud CLI](/sdk/install) components on your workstation:
    
    ``` text
    gcloud components install cloud-spanner-emulator
    ```
    
    If gcloud CLI is already installed, run the following command to ensure all of its components are updated:
    
    ``` text
    gcloud components update
    ```

### Create and configure the emulator using gcloud CLI

To use the emulator with gcloud CLI, you must disable authentication and override the endpoint. We recommend creating a separate [gcloud CLI configuration](/sdk/docs/configurations) so that you can quickly switch back and forth between the emulator and the production service.

1.  Create and activate an emulator configuration:
    
    ``` text
      gcloud config configurations create emulator
      gcloud config set auth/disable_credentials true
      gcloud config set project your-project-id
      gcloud config set api_endpoint_overrides/spanner http://localhost:9020/
    ```
    
    Once configured, your gcloud CLI commands are sent to the emulator instead of the production service. You can verify this by creating an instance with the emulator's instance config:
    
    ``` text
    gcloud spanner instances create test-instance \
      --config=emulator-config --description="Test Instance" --nodes=1
    ```
    
    To switch between the emulator and default configuration, run:
    
    ``` text
    gcloud config configurations activate [emulator | default]
    ```

2.  Start the emulator using [gcloud CLI](#start-emulator-gcloud) .

## Install the emulator in Docker

1.  Install [Docker](https://www.docker.com/products/docker-desktop) on your system and make it available on the system path.

2.  Get the latest emulator image:
    
    ``` text
    docker pull gcr.io/cloud-spanner-emulator/emulator
    ```

3.  Run the emulator in Docker:
    
    ``` text
    docker run -p 9010:9010 -p 9020:9020 gcr.io/cloud-spanner-emulator/emulator
    ```
    
    This command runs the emulator and maps the ports in the container to the same ports on your local host. The emulator uses two local endpoints: `  localhost:9010  ` for gRPC requests and `  localhost:9020  ` for REST requests.

4.  Start the emulator using [gcloud CLI](#start-emulator-gcloud) .

## Start the emulator using gcloud CLI

Start the emulator using the [gcloud emulators spanner](/sdk/gcloud/reference/emulators/spanner/start) command:

``` text
gcloud emulators spanner start
```

The emulator uses two local endpoints:

  - `  localhost:9010  ` for gRPC requests
  - `  localhost:9020  ` for REST requests

## Use the client libraries with the emulator

You can use [supported versions](#supported-versions) of the client libraries with the emulator by setting the `  SPANNER_EMULATOR_HOST  ` environment variable. There are many ways to do this. For example:

**Important:** If you are using C\#, see the [additional instructions for C\#](#cs) .

### Linux/macOS

``` text
export SPANNER_EMULATOR_HOST=localhost:9010
```

### Windows

``` text
set SPANNER_EMULATOR_HOST=localhost:9010
```

Or with [gcloud env-init](/sdk/gcloud/reference/emulators/spanner/env-init) :

### Linux/macOS

``` text
$(gcloud emulators spanner env-init)
```

### Windows

``` text
gcloud emulators spanner env-init > set_vars.cmd && set_vars.cmd
```

When your application starts, the client library automatically checks for `  SPANNER_EMULATOR_HOST  ` and connects to the emulator if it's running.

Once `  SPANNER_EMULATOR_HOST  ` is set, you can test the emulator by following the Getting Started guides. Ignore the instructions related to project creation, authentication, and credentials since these aren't needed to use the emulator.

  - [Getting Started in C++](/spanner/docs/getting-started/cpp)

  - [Getting Started in C\#](/spanner/docs/getting-started/csharp) . You must set connection string options. See [additional instructions for C\#](#cs) .

  - [Getting Started in Go](/spanner/docs/getting-started/go)

  - [Getting Started in Java](/spanner/docs/getting-started/java)

  - [Getting Started in Node.js](/spanner/docs/getting-started/nodejs)

  - [Getting Started in PHP](/spanner/docs/getting-started/php)

  - [Getting Started in Python](/spanner/docs/getting-started/python)

  - [Getting Started in Ruby](/spanner/docs/getting-started/ruby)

### Supported versions

The following table lists the versions of the [client libraries](/spanner/docs/reference/libraries) that support the emulator.

<table>
<thead>
<tr class="header">
<th>Client library</th>
<th>Minimum version</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>C++</td>
<td><a href="https://github.com/googleapis/google-cloud-cpp/releases">v0.9.x+</a></td>
</tr>
<tr class="even">
<td>C#</td>
<td><a href="https://github.com/googleapis/google-cloud-dotnet/releases">v3.1.0+</a></td>
</tr>
<tr class="odd">
<td>Go</td>
<td><a href="https://github.com/googleapis/google-cloud-go/releases">v1.5.0+</a></td>
</tr>
<tr class="even">
<td>Java</td>
<td><a href="https://github.com/googleapis/java-spanner/releases">v1.51.0+</a></td>
</tr>
<tr class="odd">
<td>Node.js</td>
<td><a href="https://github.com/googleapis/nodejs-spanner/releases">v4.5.0+</a></td>
</tr>
<tr class="even">
<td>PHP</td>
<td><a href="https://github.com/googleapis/google-cloud-php/releases">v1.25.0+</a></td>
</tr>
<tr class="odd">
<td>Python</td>
<td><a href="https://github.com/googleapis/python-spanner/releases">v1.15.0+</a></td>
</tr>
<tr class="even">
<td>Ruby</td>
<td><a href="https://github.com/googleapis/google-cloud-ruby/releases">v1.13.0+</a></td>
</tr>
</tbody>
</table>

### Additional instructions for C\#

For the C\# client library, you must also specify the [`  emulatordetection  `](/dotnet/docs/reference/Google.Api.Gax/latest/Google.Api.Gax.EmulatorDetection) option in the [connection string](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/connection_string) . Unlike the other client libraries, C\# ignores the `  SPANNER_EMULATOR_HOST  ` environment variable by default. The following is an example for the connection string:

``` text
var builder = new SpannerConnectionStringBuilder
{
    DataSource = $"projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
    EmulatorDetection = "EmulatorOnly"
};
```
