This document describes how to authenticate to Spanner programmatically. How you authenticate to Spanner depends on the interface you use to access the API and the environment where your code is running.

For more information about Google Cloud authentication, see the [Authentication methods](/docs/authentication) .

## API access

Spanner supports programmatic access. You can access the API in the following ways:

  - [Client libraries](#client-libraries)
  - [Google Cloud CLI](#gcloud)
  - [REST](#rest)

### Client libraries

The [Spanner client libraries](/spanner/docs/reference/libraries) provide high-level language support for authenticating to Spanner programmatically. To authenticate calls to Google Cloud APIs, client libraries support [Application Default Credentials (ADC)](/docs/authentication/application-default-credentials) ; the libraries look for credentials in a set of defined locations and use those credentials to authenticate requests to the API. With ADC, you can make credentials available to your application in a variety of environments, such as local development or production, without needing to modify your application code.

### Google Cloud CLI

When you use the [gcloud CLI](/sdk/gcloud/reference/spanner) to access Spanner, you [log in to the gcloud CLI](/sdk/docs/authorizing) with a user account, which provides the credentials used by the gcloud CLI commands.

If your organization's security policies prevent user accounts from having the required permissions, you can use [service account impersonation](#sa-impersonation) .

For more information, see [Authenticate for using the gcloud CLI](/docs/authentication/gcloud) . For more information about using the gcloud CLI with Spanner, see [gcloud spanner](/sdk/gcloud/reference/spanner) .

### REST

You can authenticate to [Spanner API](/spanner/docs/reference/rest) by using your gcloud CLI credentials or by using [Application Default Credentials](/docs/authentication/application-default-credentials) . For more information about authentication for REST requests, see [Authenticate for using REST](/docs/authentication/rest) . For information about the types of credentials, see [gcloud CLI credentials and ADC credentials](/docs/authentication/gcloud#gcloud-credentials) .

## User credentials and ADC for Spanner

One way to provide credentials to ADC is to use the gcloud CLI to insert your user credentials into a credential file. This file is placed on your local file system where ADC can find it; ADC then uses the provided user credentials to authenticate requests. This method is often used for local development.

If you use this method, you might encounter an authentication error when you try to authenticate to Spanner. For more information about this error and how to address it, see [User credentials not working](/docs/authentication/troubleshoot-adc#user-creds-client-based) .

## Set up authentication for Spanner

How you set up authentication depends on the environment where your code is running.

The following options for setting up authentication are the most commonly used. For more options and information about authentication, see [Authentication methods](/docs/authentication) .

Before you complete these instructions, you must complete the basic setup for Spanner, as described in [Install the gcloud CLI and set up the Cloud Spanner API](/spanner/docs/getting-started/set-up) .

### For a local development environment

You can set up credentials for a local development environment in the following ways:

  - [User credentials for client libraries or third-party tools](#client-libs)
  - [User credentials for REST requests from the command line](#rest-requests)
  - [Service account impersonation](#sa-impersonation)

#### Client libraries or third-party tools

Set up [Application Default Credentials (ADC)](/docs/authentication/application-default-credentials) in your local environment:

1.  [Install](/sdk/docs/install) the Google Cloud CLI. After installation, [initialize](/sdk/docs/initializing) the Google Cloud CLI by running the following command:
    
    ``` text
    gcloud init
    ```
    
    If you're using an external identity provider (IdP), you must first [sign in to the gcloud CLI with your federated identity](/iam/docs/workforce-log-in-gcloud) .

2.  If you're using a local shell, then create local authentication credentials for your user account:
    
    ``` text
    gcloud auth application-default login
    ```
    
    You don't need to do this if you're using Cloud Shell.
    
    **Note:** If the gcloud CLI prints a warning that your account doesn't have the `  serviceusage.services.use  ` permission, then some gcloud CLI commands and client libraries might not work. Ask an administrator to grant you the Service Usage Consumer IAM role ( `  roles/serviceusage.serviceUsageConsumer  ` ), then run the following command:
    
    ``` text
    gcloud auth application-default set-quota-project PROJECT_ID
    ```
    
    If an authentication error is returned, and you are using an external identity provider (IdP), confirm that you have [signed in to the gcloud CLI with your federated identity](/iam/docs/workforce-log-in-gcloud) .
    
    A sign-in screen appears. After you sign in, your credentials are stored in the [local credential file used by ADC](/docs/authentication/application-default-credentials#personal) .

For more information about working with ADC in a local environment, see [Set up ADC for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

#### REST requests from the command line

When you make a REST request from the command line, you can use your gcloud CLI credentials by including [`  gcloud auth print-access-token  `](/sdk/gcloud/reference/auth/print-access-token) as part of the command that sends the request.

The following example lists service accounts for the specified project. You can use the same pattern for any REST request.

Before using any of the request data, make the following replacements:

  - PROJECT\_ID : Your Google Cloud project ID.

To send your request, expand one of these options:

#### curl (Linux, macOS, or Cloud Shell)

Execute the following command:

``` text
curl -X GET \
     -H "Authorization: Bearer $(gcloud auth print-access-token)" \
     -H "x-goog-user-project: PROJECT_ID" \
     "https://iam.googleapis.com/v1/projects/PROJECT_ID/serviceAccounts"
```

#### PowerShell (Windows)

Execute the following command:

``` text
$cred = gcloud auth print-access-token
$headers = @{ "Authorization" = "Bearer $cred"; "x-goog-user-project" = "PROJECT_ID" }

Invoke-WebRequest `
    -Method GET `
    -Headers $headers `
    -Uri "https://iam.googleapis.com/v1/projects/PROJECT_ID/serviceAccounts" | Select-Object -Expand Content
```

For more information about authenticating using REST and gRPC, see [Authenticate for using REST](/docs/authentication/rest) . For information about the difference between your local ADC credentials and your gcloud CLI credentials, see [gcloud CLI authentication configuration and ADC configuration](/docs/authentication/gcloud#gcloud-credentials) .

#### Service account impersonation

In most cases, you can use your user credentials to authenticate from a local development environment. If that is not feasible, or if you need to test the permissions assigned to a service account, you can use service account impersonation. You must have the `  iam.serviceAccounts.getAccessToken  ` permission, which is included in the [Service Account Token Creator](/iam/docs/roles-permissions/iam#iam.serviceAccountTokenCreator) ( `  roles/iam.serviceAccountTokenCreator  ` ) IAM role.

You can set up the gcloud CLI to use service account impersonation by using the [`  gcloud config set  ` command](/sdk/gcloud/reference/config) :

``` text
gcloud config set auth/impersonate_service_account SERVICE_ACCT_EMAIL
```

For select languages, you can use service account impersonation to create a local ADC file for use by client libraries. This approach is supported only for the Go, Java, Node.js, and Python client libraries—it is not supported for the other languages. To set up a local ADC file with service account impersonation, use the [`  --impersonate-service-account  ` flag](/sdk/gcloud/reference#--impersonate-service-account) with the [`  gcloud auth application-default login  ` command](/sdk/gcloud/reference/auth/application-default/login) :

``` text
gcloud auth application-default login --impersonate-service-account=SERVICE_ACCT_EMAIL
```

For more information about service account impersonation, see [Use service account impersonation](/docs/authentication/use-service-account-impersonation) .

### On Google Cloud

To authenticate a workload running on Google Cloud, you use the credentials of the service account attached to the compute resource where your code is running, such as a [Compute Engine virtual machine (VM) instance](/compute/docs/access/create-enable-service-accounts-for-instances#using) . This approach is the preferred authentication method for code running on a Google Cloud compute resource.

For most services, you must attach the service account when you create the resource that will run your code; you cannot add or replace the service account later. Compute Engine is an exception—it lets you attach a service account to a VM instance at any time.

Use the gcloud CLI to create a service account and attach it to your resource:

1.  [Install](/sdk/docs/install) the Google Cloud CLI. After installation, [initialize](/sdk/docs/initializing) the Google Cloud CLI by running the following command:
    
    ``` text
    gcloud init
    ```
    
    If you're using an external identity provider (IdP), you must first [sign in to the gcloud CLI with your federated identity](/iam/docs/workforce-log-in-gcloud) .

2.  Set up authentication:
    
    1.  Ensure that you have the Create Service Accounts IAM role ( `  roles/iam.serviceAccountCreator  ` ) and the Project IAM Admin role ( `  roles/resourcemanager.projectIamAdmin  ` ). [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .
    
    2.  Create the service account:
        
        ``` text
        gcloud iam service-accounts create SERVICE_ACCOUNT_NAME
        ```
        
        Replace `  SERVICE_ACCOUNT_NAME  ` with a name for the service account.
    
    3.  To provide access to your project and your resources, grant a role to the service account:
        
        ``` text
        gcloud projects add-iam-policy-binding PROJECT_ID --member="serviceAccount:SERVICE_ACCOUNT_NAME@PROJECT_ID.iam.gserviceaccount.com" --role=ROLE
        ```
        
        Replace the following:
        
          - `  SERVICE_ACCOUNT_NAME  ` : the name of the service account
          - `  PROJECT_ID  ` : the project ID where you created the service account
          - `  ROLE  ` : the role to grant
        
        **Note** : The `  --role  ` flag affects which resources the service account can access in your project. You can revoke these roles or grant additional roles later. In production environments, do not grant the Owner, Editor, or Viewer roles. Instead, grant a [predefined role](/iam/docs/understanding-roles#predefined_roles) or [custom role](/iam/docs/understanding-custom-roles) that meets your needs.
    
    4.  To grant another role to the service account, run the command as you did in the previous step.
    
    5.  Grant the required role to the principal that will attach the service account to other resources.
        
        ``` text
        gcloud iam service-accounts add-iam-policy-binding SERVICE_ACCOUNT_NAME@PROJECT_ID.iam.gserviceaccount.com --member="user:USER_EMAIL" --role=roles/iam.serviceAccountUser
        ```
        
        Replace the following:
        
          - `  SERVICE_ACCOUNT_NAME  ` : the name of the service account
          - `  PROJECT_ID  ` : the project ID where you created the service account
          - `  USER_EMAIL  ` : the email address for a Google Account

3.  Create the resource that will run your code, and attach the service account to that resource. For example, if you use Compute Engine:
    
    Create a Compute Engine instance. Configure the instance as follows:
    
      - Replace `  INSTANCE_NAME  ` with your preferred instance name.
      - Set the `  --zone  ` flag to the [zone](/compute/docs/zones#available) in which you want to create your instance.
      - Set the `  --service-account  ` flag to the email address for the service account that you created.
    
    <!-- end list -->
    
    ``` text
    gcloud compute instances create INSTANCE_NAME --zone=ZONE --service-account=SERVICE_ACCOUNT_EMAIL
    ```

For more information about authenticating to Google APIs, see [Authentication methods](/docs/authentication) .

### On-premises or on a different cloud provider

The preferred method to set up authentication from outside of Google Cloud is to use workload identity federation. For more information, see [Set up ADC for on-premises or another cloud provider](/docs/authentication/set-up-adc-on-premises) in the authentication documentation.

## Access control for Spanner

After you authenticate to Spanner, you must be authorized to access Google Cloud resources. Spanner uses Identity and Access Management (IAM) for authorization.

For more information about the roles for Spanner, see [Identity and Access Management overview](/spanner/docs/iam) . For more information about IAM and authorization, see [IAM overview](/iam/docs/overview) .

## What's next

  - Learn about [Google Cloud authentication methods](/docs/authentication#auth-decision-tree) .
  - See a list of [authentication use cases](/docs/authentication/use-cases) .
