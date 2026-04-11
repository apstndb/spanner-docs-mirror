This page walks you through the setup steps required to use the Cloud Spanner API with the [Google Cloud CLI](https://docs.cloud.google.com/sdk/gcloud) , [client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) , and [Spanner drivers](https://docs.cloud.google.com/spanner/docs/drivers-overview) .

If you want to use Spanner with the Google Cloud console, see [Quickstart using the console](https://docs.cloud.google.com/spanner/docs/create-query-database-console) .

## Required roles

To get the permissions that you need to use and interact with Spanner databases, ask your administrator to grant you the following IAM roles:

  - Read and write data: [Cloud Spanner Database User](https://docs.cloud.google.com/iam/docs/roles-permissions/spanner#spanner.databaseUser) ( `roles/spanner.databaseUser` ) on the instance
  - Read-only access to databases: [Cloud Spanner Database Reader](https://docs.cloud.google.com/iam/docs/roles-permissions/spanner#spanner.databaseReader) ( `roles/spanner.databaseReader` ) on the instance

For more information about granting roles, see [Manage access to projects, folders, and organizations](https://docs.cloud.google.com/iam/docs/granting-changing-revoking-access) .

You might also be able to get the required permissions through [custom roles](https://docs.cloud.google.com/iam/docs/creating-custom-roles) or other [predefined roles](https://docs.cloud.google.com/iam/docs/roles-overview#predefined) .

## Set up a Google Cloud project

1.  Go to the **Projects** page in the Google Cloud console.
    
    [Go to the Projects page](https://console.cloud.google.com/project)

2.  Create a new Google Cloud project, or open an existing project by clicking on the project name.

3.  Open a terminal window, and set your project as the default project for the Google Cloud CLI, replacing `MY_PROJECT_ID` with your project ID (not your project name):
    
        gcloud config set project MY_PROJECT_ID

4.  [Verify that billing is enabled for your Google Cloud project](https://docs.cloud.google.com/billing/docs/how-to/verify-billing-enabled#confirm_billing_is_enabled_on_a_project) .

5.  Enable the Cloud Spanner API for the project.
    
    [Enable the Cloud Spanner API](https://console.cloud.google.com/flows/enableapi?apiid=spanner.googleapis.com)
    
    Note: If you use a service account in a different project to access your Spanner instance, you need to enable the Spanner API in both projects.

## Set up authentication

Select the tabs for how you plan to access the API:

### Console

When you use the Google Cloud console to access Google Cloud services and APIs, you don't need to set up authentication.

### gcloud

[Install](https://docs.cloud.google.com/sdk/docs/install) the Google Cloud CLI. After installation, [initialize](https://docs.cloud.google.com/sdk/docs/initializing) the Google Cloud CLI by running the following command:

    gcloud init

If you're using an external identity provider (IdP), you must first [sign in to the gcloud CLI with your federated identity](https://docs.cloud.google.com/iam/docs/workforce-log-in-gcloud) .

To set up the gcloud CLI to use service account impersonation to authenticate to Google APIs, rather than your user credentials, run the following command:

    gcloud config set auth/impersonate_service_account SERVICE_ACCT_EMAIL

For more information, see [Service account impersonation](https://docs.cloud.google.com/spanner/docs/authentication#sa-impersonation) .

### Terraform

To use Terraform code in a local development environment, install and initialize the gcloud CLI, and then set up Application Default Credentials with your user credentials.

1.  [Install](https://docs.cloud.google.com/sdk/docs/install) the Google Cloud CLI.

2.  If you're using an external identity provider (IdP), you must first [sign in to the gcloud CLI with your federated identity](https://docs.cloud.google.com/iam/docs/workforce-log-in-gcloud) .

3.  If you're using a local shell, then create local authentication credentials for your user account:
    
        gcloud auth application-default login
    
    You don't need to do this if you're using Cloud Shell.
    
    If an authentication error is returned, and you are using an external identity provider (IdP), confirm that you have [signed in to the gcloud CLI with your federated identity](https://docs.cloud.google.com/iam/docs/workforce-log-in-gcloud) .

For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/spanner/docs/authentication#local-development) .

### Client libraries

To use client libraries in a local development environment, install and initialize the gcloud CLI, and then set up Application Default Credentials with your user credentials.

1.  [Install](https://docs.cloud.google.com/sdk/docs/install) the Google Cloud CLI.

2.  If you're using an external identity provider (IdP), you must first [sign in to the gcloud CLI with your federated identity](https://docs.cloud.google.com/iam/docs/workforce-log-in-gcloud) .

3.  If you're using a local shell, then create local authentication credentials for your user account:
    
        gcloud auth application-default login
    
    You don't need to do this if you're using Cloud Shell.
    
    If an authentication error is returned, and you are using an external identity provider (IdP), confirm that you have [signed in to the gcloud CLI with your federated identity](https://docs.cloud.google.com/iam/docs/workforce-log-in-gcloud) .

For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/spanner/docs/authentication#local-development) .

To set up your local ADC file to use service account impersonation to authenticate to Google APIs, rather than your user credentials, run the following command:

    gcloud auth application-default login --impersonate-service-account=SERVICE_ACCT_EMAIL

For more information, see [Service account impersonation](https://docs.cloud.google.com/spanner/docs/authentication#sa-impersonation) .

### REST

To use the REST API in a local development environment, you use the credentials you provide to the gcloud CLI.

For more information, see [Authenticate for using REST](https://docs.cloud.google.com/docs/authentication/rest) in the Google Cloud authentication documentation.

You can use service account impersonation to generate an access token for REST API requests. For more information, see [Impersonated service account](https://docs.cloud.google.com/docs/authentication/rest#impersonated-sa) .

## Run the Google Cloud CLI

Now that you've set up your development environment and authentication, run the [`gcloud` command-line](https://docs.cloud.google.com/spanner/docs/gcloud-spanner) tool to interact with Spanner:

    gcloud spanner instance-configs list

You should see a list of the Spanner instance configurations that your project can access, including regional, dual-region, and multi-region configurations. For more information, see the [Instances overview](https://docs.cloud.google.com/spanner/docs/instances) .

You've completed the setup\!

## What's next

Learn how to use the Cloud Client Libraries and drivers to create a Spanner instance, database, tables, and indexes. Then store, query, and read data in Spanner.

  - [Getting started with Spanner in C++](https://docs.cloud.google.com/spanner/docs/getting-started/cpp)
  - [Getting started with Spanner in C\#](https://docs.cloud.google.com/spanner/docs/getting-started/csharp)
  - [Getting started with Spanner in Go](https://docs.cloud.google.com/spanner/docs/getting-started/go)
  - [Getting started with Spanner in Java](https://docs.cloud.google.com/spanner/docs/getting-started/java)
  - [Getting started with Spanner in JDBC](https://docs.cloud.google.com/spanner/docs/getting-started/jdbc)
  - [Getting started with Spanner in Node.js](https://docs.cloud.google.com/spanner/docs/getting-started/nodejs)
  - [Getting started with Spanner in PHP](https://docs.cloud.google.com/spanner/docs/getting-started/php)
  - [Getting started with Spanner in Python](https://docs.cloud.google.com/spanner/docs/getting-started/python)
  - [Getting started with Spanner in Ruby](https://docs.cloud.google.com/spanner/docs/getting-started/ruby)
  - [Getting started with Spanner in REST](https://docs.cloud.google.com/spanner/docs/getting-started/rest)
  - [Getting started with Spanner in gcloud](https://docs.cloud.google.com/spanner/docs/getting-started/gcloud)
