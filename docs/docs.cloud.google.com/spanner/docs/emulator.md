The [gcloud CLI](https://docs.cloud.google.com/sdk) provides a local, in-memory emulator, which you can use to develop and test your applications. Because the emulator stores data only in memory, all state, including data, schema, and configs, is lost on restart. The emulator offers the same APIs as the Spanner production service and is intended for local development and testing, not for production deployments.

The emulator supports both the GoogleSQL and PostgreSQL dialects. It supports all languages of the [client libraries](https://docs.cloud.google.com/spanner/docs/emulator#client-libraries) . You can also use the emulator with the [Google Cloud CLI](https://docs.cloud.google.com/sdk/gcloud) and [REST APIs](https://docs.cloud.google.com/spanner/docs/reference/rest) .

The emulator is also available as an open source project in [GitHub](https://github.com/GoogleCloudPlatform/cloud-spanner-emulator) .

**Note:** To explore Spanner using a 90-day free trial instance, see [Create a Spanner free trial instance](https://docs.cloud.google.com/spanner/docs/free-trial-quickstart) .

## Limitations and differences

The emulator doesn't support the following:

  - TLS/HTTPS, authentication, Identity and Access Management, permissions, or roles.
  - In the `PLAN` or `PROFILE` [query modes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/QueryMode) , the query plan that is returned is empty.
  - The [`ANALYZE` statement](https://docs.cloud.google.com/spanner/docs/query-optimizer/overview#construct-statistics-package) . The emulator accepts but ignores it.
  - Any of the [audit logging](https://docs.cloud.google.com/spanner/docs/audit-logging) and monitoring tools.

The emulator also differs from the Spanner production service in the following ways:

  - Error messages might not be consistent between the emulator and the production service.
  - Performance and scalability for the emulator is not comparable to the production service.
  - Read-write transactions and schema changes lock the entire database for exclusive access until they are completed.
  - [Partitioned DML](https://docs.cloud.google.com/spanner/docs/dml-partitioned) and [`partitionQuery`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/partitionQuery) are supported, but the emulator doesn't check to ensure that statements are [partitionable](https://docs.cloud.google.com/spanner/docs/dml-partitioned#partitionable-idempotent) . This means that a partitioned DML or `partitionQuery` statement might run in the emulator, but it might fail in the production service with the non-partitionable statement error.

For a complete list of APIs and features that are supported, unsupported, and partially supported, see the [README](https://github.com/GoogleCloudPlatform/cloud-spanner-emulator/blob/master/README.md) file in GitHub.

## Options for running the emulator

There are two common ways to run the emulator:

  - [gcloud CLI](https://docs.cloud.google.com/spanner/docs/emulator#emulator-for-gcloud)
  - [Docker](https://docs.cloud.google.com/spanner/docs/emulator#install-docker)

Choose the way that is appropriate for your application development and test workflow.

**Note:** For more ways to run the emulator, see the [README](https://github.com/GoogleCloudPlatform/cloud-spanner-emulator/blob/master/README.md#quickstart) .

## Set up the emulator for gcloud CLI

For Windows and macOS users, before installing the emulator, do the following:

  - Install the [gcloud CLI](https://docs.cloud.google.com/sdk/install) components on your workstation:
    
        gcloud components install cloud-spanner-emulator
    
    If gcloud CLI is already installed, run the following command to ensure all of its components are updated:
    
        gcloud components update

### Create and configure the emulator using gcloud CLI

To use the emulator with gcloud CLI, you must disable authentication and override the endpoint. We recommend creating a separate [gcloud CLI configuration](https://docs.cloud.google.com/sdk/docs/configurations) so that you can quickly switch back and forth between the emulator and the production service.

1.  Create and activate an emulator configuration:
    
    ``` 
      gcloud config configurations create emulator
      gcloud config set auth/disable_credentials true
      gcloud config set project your-project-id
      gcloud config set api_endpoint_overrides/spanner http://localhost:9020/
    ```
    
    Once configured, your gcloud CLI commands are sent to the emulator instead of the production service. You can verify this by creating an instance with the emulator's instance config:
    
        gcloud spanner instances create test-instance \
          --config=emulator-config --description="Test Instance" --nodes=1
    
    To switch between the emulator and default configuration, run:
    
        gcloud config configurations activate [emulator | default]

2.  Start the emulator using [gcloud CLI](https://docs.cloud.google.com/spanner/docs/emulator#start-emulator-gcloud) .

## Install the emulator in Docker

1.  Install [Docker](https://www.docker.com/products/docker-desktop) on your system and make it available on the system path.

2.  Get the latest emulator image:
    
        docker pull gcr.io/cloud-spanner-emulator/emulator

3.  Run the emulator in Docker:
    
        docker run -p 9010:9010 -p 9020:9020 gcr.io/cloud-spanner-emulator/emulator
    
    This command runs the emulator and maps the ports in the container to the same ports on your local host. The emulator uses two local endpoints: `localhost:9010` for gRPC requests and `localhost:9020` for REST requests.

4.  Start the emulator using [gcloud CLI](https://docs.cloud.google.com/spanner/docs/emulator#start-emulator-gcloud) .

## Start the emulator using gcloud CLI

Start the emulator using the [gcloud emulators spanner](https://docs.cloud.google.com/sdk/gcloud/reference/emulators/spanner/start) command:

    gcloud emulators spanner start

The emulator uses two local endpoints:

  - `localhost:9010` for gRPC requests
  - `localhost:9020` for REST requests

## Use the client libraries with the emulator

You can use [supported versions](https://docs.cloud.google.com/spanner/docs/emulator#supported-versions) of the client libraries with the emulator by setting the `SPANNER_EMULATOR_HOST` environment variable. There are many ways to do this. For example:

**Important:** If you are using C\#, see the [additional instructions for C\#](https://docs.cloud.google.com/spanner/docs/emulator#cs) .

### Linux/macOS

    export SPANNER_EMULATOR_HOST=localhost:9010

### Windows

    set SPANNER_EMULATOR_HOST=localhost:9010

Or with [gcloud env-init](https://docs.cloud.google.com/sdk/gcloud/reference/emulators/spanner/env-init) :

### Linux/macOS

    $(gcloud emulators spanner env-init)

### Windows

    gcloud emulators spanner env-init > set_vars.cmd && set_vars.cmd

When your application starts, the client library automatically checks for `SPANNER_EMULATOR_HOST` and connects to the emulator if it's running.

Once `SPANNER_EMULATOR_HOST` is set, you can test the emulator by following the Getting Started guides. Ignore the instructions related to project creation, authentication, and credentials since these aren't needed to use the emulator.

  - [Getting Started in C++](https://docs.cloud.google.com/spanner/docs/getting-started/cpp)

  - [Getting Started in C\#](https://docs.cloud.google.com/spanner/docs/getting-started/csharp) . You must set connection string options. See [additional instructions for C\#](https://docs.cloud.google.com/spanner/docs/emulator#cs) .

  - [Getting Started in Go](https://docs.cloud.google.com/spanner/docs/getting-started/go)

  - [Getting Started in Java](https://docs.cloud.google.com/spanner/docs/getting-started/java)

  - [Getting Started in Node.js](https://docs.cloud.google.com/spanner/docs/getting-started/nodejs)

  - [Getting Started in PHP](https://docs.cloud.google.com/spanner/docs/getting-started/php)

  - [Getting Started in Python](https://docs.cloud.google.com/spanner/docs/getting-started/python)

  - [Getting Started in Ruby](https://docs.cloud.google.com/spanner/docs/getting-started/ruby)

### Supported versions

The following table lists the versions of the [client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) that support the emulator.

| Client library | Minimum version                                                       |
| -------------- | --------------------------------------------------------------------- |
| C++            | [v0.9.x+](https://github.com/googleapis/google-cloud-cpp/releases)    |
| C\#            | [v3.1.0+](https://github.com/googleapis/google-cloud-dotnet/releases) |
| Go             | [v1.5.0+](https://github.com/googleapis/google-cloud-go/releases)     |
| Java           | [v1.51.0+](https://github.com/googleapis/java-spanner/releases)       |
| Node.js        | [v4.5.0+](https://github.com/googleapis/nodejs-spanner/releases)      |
| PHP            | [v1.25.0+](https://github.com/googleapis/google-cloud-php/releases)   |
| Python         | [v1.15.0+](https://github.com/googleapis/python-spanner/releases)     |
| Ruby           | [v1.13.0+](https://github.com/googleapis/google-cloud-ruby/releases)  |

### Additional instructions for C\#

For the C\# client library, you must also specify the [`emulatordetection`](https://docs.cloud.google.com/dotnet/docs/reference/Google.Api.Gax/latest/Google.Api.Gax.EmulatorDetection) option in the [connection string](https://docs.cloud.google.com/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest/connection_string) . Unlike the other client libraries, C\# ignores the `SPANNER_EMULATOR_HOST` environment variable by default. The following is an example for the connection string:

    var builder = new SpannerConnectionStringBuilder
    {
        DataSource = $"projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        EmulatorDetection = "EmulatorOnly"
    };
