This page walks you through the setup steps required to use the Cloud Spanner API with the [Google Cloud CLI](/sdk/gcloud) , [client libraries](/spanner/docs/reference/libraries) , and [Spanner drivers](/spanner/docs/drivers-overview) .

If you want to use Spanner with the Google Cloud console, see [Quickstart using the console](/spanner/docs/create-query-database-console) .

## Required roles

To get the permissions that you need to use and interact with Spanner databases, ask your administrator to grant you the following IAM roles:

  - Read and write data: [Cloud Spanner Database User](/iam/docs/roles-permissions/spanner#spanner.databaseUser) ( `  roles/spanner.databaseUser  ` ) on the instance
  - Read-only access to databases: [Cloud Spanner Database Reader](/iam/docs/roles-permissions/spanner#spanner.databaseReader) ( `  roles/spanner.databaseReader  ` ) on the instance

For more information about granting roles, see [Manage access to projects, folders, and organizations](/iam/docs/granting-changing-revoking-access) .

You might also be able to get the required permissions through [custom roles](/iam/docs/creating-custom-roles) or other [predefined roles](/iam/docs/roles-overview#predefined) .

## Set up a Google Cloud project

1.  Go to the **Projects** page in the Google Cloud console.

2.  Create a new Google Cloud project, or open an existing project by clicking on the project name.

3.  Open a terminal window, and set your project as the default project for the Google Cloud CLI, replacing `  MY_PROJECT_ID  ` with your project ID (not your project name):
    
    ``` text
    gcloud config set project MY_PROJECT_ID
    ```

4.  Enable the Cloud Spanner API for the project.
    
    Note: If you use a service account in a different project to access your Spanner instance, you need to enable the Spanner API in both projects.

## Set up authentication

Select the tabs for how you plan to access the API:

### Console

When you use the Google Cloud console to access Google Cloud services and APIs, you don't need to set up authentication.

### gcloud

[Install](/sdk/docs/install) the Google Cloud CLI. After installation, [initialize](/sdk/docs/initializing) the Google Cloud CLI by running the following command:

``` text
gcloud init
```

If you're using an external identity provider (IdP), you must first [sign in to the gcloud CLI with your federated identity](/iam/docs/workforce-log-in-gcloud) .

To set up the gcloud CLI to use service account impersonation to authenticate to Google APIs, rather than your user credentials, run the following command:

``` text
gcloud config set auth/impersonate_service_account SERVICE_ACCT_EMAIL
```

For more information, see [Service account impersonation](/spanner/docs/authentication#sa-impersonation) .

### Terraform

To use Terraform code in a local development environment, install and initialize the gcloud CLI, and then set up Application Default Credentials with your user credentials.

For more information, see [Set up authentication for a local development environment](/spanner/docs/authentication#local-development) .

### Client libraries

To use client libraries in a local development environment, install and initialize the gcloud CLI, and then set up Application Default Credentials with your user credentials.

For more information, see [Set up authentication for a local development environment](/spanner/docs/authentication#local-development) .

To set up your local ADC file to use service account impersonation to authenticate to Google APIs, rather than your user credentials, run the following command:

``` text
gcloud auth application-default login --impersonate-service-account=SERVICE_ACCT_EMAIL
```

For more information, see [Service account impersonation](/spanner/docs/authentication#sa-impersonation) .

### REST

To use the REST API in a local development environment, you use the credentials you provide to the gcloud CLI.

For more information, see [Authenticate for using REST](/docs/authentication/rest) in the Google Cloud authentication documentation.

You can use service account impersonation to generate an access token for REST API requests. For more information, see [Impersonated service account](/docs/authentication/rest#impersonated-sa) .

## Run the Google Cloud CLI

Now that you've set up your development environment and authentication, run the [`  gcloud  ` command-line](/spanner/docs/gcloud-spanner) tool to interact with Spanner:

``` text
gcloud spanner instance-configs list
```

You should see a list of the Spanner instance configurations that your project can access, including regional, dual-region, and multi-region configurations. For more information, see the [Instances overview](/spanner/docs/instances) .

You've completed the setup\!

## What's next

Learn how to use the Cloud Client Libraries and drivers to create a Spanner instance, database, tables, and indexes. Then store, query, and read data in Spanner.

  - [Getting started with Spanner in C++](/spanner/docs/getting-started/cpp)
  - [Getting started with Spanner in C\#](/spanner/docs/getting-started/csharp)
  - [Getting started with Spanner in Go](/spanner/docs/getting-started/go)
  - [Getting started with Spanner in Java](/spanner/docs/getting-started/java)
  - [Getting started with Spanner in JDBC](/spanner/docs/getting-started/jdbc)
  - [Getting started with Spanner in Node.js](/spanner/docs/getting-started/nodejs)
  - [Getting started with Spanner in PHP](/spanner/docs/getting-started/php)
  - [Getting started with Spanner in Python](/spanner/docs/getting-started/python)
  - [Getting started with Spanner in Ruby](/spanner/docs/getting-started/ruby)
  - [Getting started with Spanner in REST](/spanner/docs/getting-started/rest)
  - [Getting started with Spanner in gcloud](/spanner/docs/getting-started/gcloud)
