**Preview**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) , and the [Additional Terms for Generative AI Preview Products](https://cloud.google.com/trustedtester/aitos) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

[Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro) (MCP) standardizes the way large language models (LLMs) and AI applications or agents connect to outside data sources. MCP servers let you use their tools, resources, and prompts to take actions and get updated data from their backend service.

Local MCP servers typically run on your local machine and use the standard input and output streams (stdio) for communication between services on the same device. Remote MCP servers run on the service's infrastructure and offer an HTTP endpoint to AI applications for communication between the AI MCP client and the MCP server. For more information on MCP architecture, see [MCP architecture](https://modelcontextprotocol.io/docs/learn/architecture) .

This document describes how to use the Spanner remote Model Context Protocol (MCP) server to connect to Spanner from AI applications such as Gemini CLI, agent mode in Gemini Code Assist, Claude Code, or from AI applications you're developing.

For information on the Spanner local MCP server, see [Spanner MCP server on GitHub](https://googleapis.github.io/genai-toolbox/resources/sources/spanner/) .

Google and Google Cloud remote MCP servers have the following features and benefits:

  - Simplified, centralized discovery.
  - Managed global or regional HTTP endpoints.
  - Fine-grained authorization.
  - Optional prompt and response security with Model Armor protection.
  - Centralized audit logging.

For information about other MCP servers and information about security and governance controls available for Google Cloud MCP servers, see [Google Cloud MCP servers overview](/mcp/overview) .

## Before you begin

Sign in to your Google Cloud account. If you're new to Google Cloud, [create an account](https://console.cloud.google.com/freetrial) to evaluate how our products perform in real-world scenarios. New customers also get $300 in free credits to run, test, and deploy workloads.

In the Google Cloud console, on the project selector page, select or create a Google Cloud project.

**Roles required to select or create a project**

  - **Select a project** : Selecting a project doesn't require a specific IAM role—you can select any project that you've been granted a role on.
  - **Create a project** : To create a project, you need the Project Creator role ( `  roles/resourcemanager.projectCreator  ` ), which contains the `  resourcemanager.projects.create  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .

**Note** : If you don't plan to keep the resources that you create in this procedure, create a project instead of selecting an existing project. After you finish these steps, you can delete the project, removing all resources associated with the project.

If you're using an existing project for this guide, [verify that you have the permissions required to complete this guide](#required_roles) . If you created a new project, then you already have the required permissions.

[Verify that billing is enabled for your Google Cloud project](/billing/docs/how-to/verify-billing-enabled#confirm_billing_is_enabled_on_a_project) .

Enable the Spanner API.

**Roles required to enable APIs**

To enable APIs, you need the Service Usage Admin IAM role ( `  roles/serviceusage.serviceUsageAdmin  ` ), which contains the `  serviceusage.services.enable  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .

[Install](/sdk/docs/install) the Google Cloud CLI. After installation, [initialize](/sdk/docs/initializing) the Google Cloud CLI by running the following command:

``` text
gcloud init
```

If you're using an external identity provider (IdP), you must first [sign in to the gcloud CLI with your federated identity](/iam/docs/workforce-log-in-gcloud) .

**Note:** If you installed the gcloud CLI previously, make sure you have the latest version by running `  gcloud components update  ` .

In the Google Cloud console, on the project selector page, select or create a Google Cloud project.

**Roles required to select or create a project**

  - **Select a project** : Selecting a project doesn't require a specific IAM role—you can select any project that you've been granted a role on.
  - **Create a project** : To create a project, you need the Project Creator role ( `  roles/resourcemanager.projectCreator  ` ), which contains the `  resourcemanager.projects.create  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .

**Note** : If you don't plan to keep the resources that you create in this procedure, create a project instead of selecting an existing project. After you finish these steps, you can delete the project, removing all resources associated with the project.

If you're using an existing project for this guide, [verify that you have the permissions required to complete this guide](#required_roles) . If you created a new project, then you already have the required permissions.

[Verify that billing is enabled for your Google Cloud project](/billing/docs/how-to/verify-billing-enabled#confirm_billing_is_enabled_on_a_project) .

Enable the Spanner API.

**Roles required to enable APIs**

To enable APIs, you need the Service Usage Admin IAM role ( `  roles/serviceusage.serviceUsageAdmin  ` ), which contains the `  serviceusage.services.enable  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .

[Install](/sdk/docs/install) the Google Cloud CLI. After installation, [initialize](/sdk/docs/initializing) the Google Cloud CLI by running the following command:

``` text
gcloud init
```

If you're using an external identity provider (IdP), you must first [sign in to the gcloud CLI with your federated identity](/iam/docs/workforce-log-in-gcloud) .

**Note:** If you installed the gcloud CLI previously, make sure you have the latest version by running `  gcloud components update  ` .

Enable the Spanner API.

**Roles required to enable APIs**

To enable APIs, you need the Service Usage Admin IAM role ( `  roles/serviceusage.serviceUsageAdmin  ` ), which contains the `  serviceusage.services.enable  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .

For new projects, the Spanner API is automatically enabled.

### Required roles

To get the permissions that you need to enable the Spanner MCP server, ask your administrator to grant you the following IAM roles on the project where you want to enable the Spanner MCP server:

  - [Service Usage Admin](/iam/docs/roles-permissions/serviceusage#serviceusage.serviceUsageAdmin) ( `  roles/serviceusage.serviceUsageAdmin  ` )
  - Make MCP tool calls: [MCP Tool User](/iam/docs/roles-permissions/mcp#mcp.toolUser) ( `  roles/mcp.toolUser  ` )
  - Use Spanner MCP tools: [Cloud Spanner Admin](/iam/docs/roles-permissions/spanner#spanner.admin) ( `  roles/spanner.admin  ` )

For more information about granting roles, see [Manage access to projects, folders, and organizations](/iam/docs/granting-changing-revoking-access) .

These predefined roles contain the permissions required to enable the Spanner MCP server. To see the exact permissions that are required, expand the **Required permissions** section:

#### Required permissions

The following permissions are required to enable the Spanner MCP server:

  - `  serviceusage.mcppolicy.get  `
  - `  serviceusage.mcppolicy.update  `
  - Make MCP tool calls: `  mcp.tools.call  `
  - Use Spanner MCP tools:
      - `  spanner.instances.create  `
      - `  spanner.instances.get  `
      - `  spanner.databases.create  `
      - `  spanner.databases.update  `
      - `  spanner.sessions.create  `
      - `  spanner.instanceOperations.get  `
      - `  spanner.databases.getDdl  `
      - `  spanner.databases.select  `
      - `  spanner.databases.write  `

You might also be able to get these permissions with [custom roles](/iam/docs/creating-custom-roles) or other [predefined roles](/iam/docs/roles-overview#predefined) .

## Enable or disable the Spanner MCP server

You can enable or disable the Spanner MCP server in a project with the `  gcloud beta services mcp enable  ` command. For more information, see the following sections.

### Enable the Spanner MCP server in a project

If you are using different projects for your client credentials, such as service account keys, OAuth client ID or API keys, and for hosting your resources, then you must enable the Spanner service and the Spanner remote MCP server on both projects.

To enable the Spanner MCP server in your Google Cloud project, run the following command:

``` text
gcloud beta services mcp enable spanner.googleapis.com \
    --project=PROJECT_ID
```

Replace the following:

  - `  PROJECT_ID  ` : the Google Cloud project ID.

The Spanner remote MCP server is enabled for use in your Google Cloud Project. If the Spanner service isn't enabled for your Google Cloud project, you are prompted to enable the service before enabling the Spanner remote MCP server.

As a security best practice, we recommend that you enable MCP servers only for the services required for your AI application to function.

### Disable the Spanner MCP server in a project

To disable the Spanner MCP server in your Google Cloud project, run the following command:

``` text
gcloud beta services mcp disable spanner.googleapis.com \
    --project=PROJECT_ID
```

The Spanner MCP server is disabled for use in your Google Cloud Project.

## Authentication and authorization

Spanner MCP servers use the [OAuth 2.0](https://developers.google.com/identity/protocols/oauth2) protocol with [Identity and Access Management (IAM)](/iam/docs/overview) for authentication and authorization. All [Google Cloud identities](/docs/authentication/identity-products) are supported for authentication to MCP servers.

The Spanner remote MCP server doesn't accept API keys.

We recommend creating a separate identity for agents using MCP tools so that access to resources can be controlled and monitored. For more information on authentication, see [Authenticate to MCP servers](/mcp/authenticate-mcp) .

## Spanner MCP OAuth scopes

OAuth 2.0 uses scopes and credentials to determine if an authenticated principal is authorized to take a specific action on a resource. For more information about OAuth 2.0 scopes at Google, read [Using OAuth 2.0 to access Google APIs](https://developers.google.com/identity/protocols/oauth2) .

Spanner has the following MCP tool OAuth scopes:

<table>
<thead>
<tr class="header">
<th>Scope URI for gcloud CLI</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       https://www.googleapis.com/auth/spanner.admin      </code></td>
<td>Allows access to administer your Spanner instances and databases.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       https://www.googleapis.com/auth/spanner.data      </code></td>
<td>Allows access to view and manage data in a Spanner database.</td>
</tr>
</tbody>
</table>

For more information about these scopes, see [Spanner API](https://developers.google.com/identity/protocols/oauth2/scopes#spanner) .

## Configure an MCP client to use the Spanner MCP server

AI applications and agents, such as Gemini CLI or Claude, can instantiate an MCP client that connects to a single MCP server. An AI application can have multiple clients that connect to different MCP servers. To connect to a remote MCP server, the MCP client must know at a minimum the URL of the remote MCP server.

In your AI application, look for a way to connect to a remote MCP server. You are prompted to enter details about the server, such as its name and URL.

For the Spanner MCP server, enter the following as required:

  - **Server name** : Spanner MCP server
  - **Server URL** or **Endpoint** : spanner.googleapis.com/mcp
  - **Transport** : HTTP
  - **Authentication details** : Depending on how you want to authenticate, you can enter your Google Cloud credentials, your OAuth Client ID and secret, or an agent identity and credentials. For more information on authentication, see [Authenticate to MCP servers](/mcp/authenticate-mcp) .
  - **OAuth scope** : the [OAuth 2.0 scope](https://developers.google.com/identity/protocols/oauth2/scopes) that you want to use when connecting to the Spanner MCP server.

For host specific guidance, see the following:

  - [Gemini CLI](/mcp/configure-mcp-ai-application#gemini-cli)
  - [Claude.ai](/mcp/configure-mcp-ai-application#claude-ai)

For more general guidance, see the following resources:

  - [Connect to remote MCP servers](https://modelcontextprotocol.io/docs/develop/connect-remote-servers) .
  - [Configure MCP in an AI application](/mcp/configure-mcp-ai-application) .

## Available tools

MCP Tools that are read-only have the MCP attribute `  mcp.tool.isReadOnly  ` set to `  true  ` . You might want to only allow read-only tools in certain environments through your [organization policy](#organization-level-mcp-control) .

To view details of available MCP tools and their descriptions for the Spanner MCP server, see the [Spanner MCP reference](/spanner/docs/reference/mcp) .

### List tools

Use the [MCP inspector](https://modelcontextprotocol.io/docs/tools/inspector) to list tools, or send a `  tools/list  ` HTTP request directly to the Spanner remote MCP server. The `  tools/list  ` method doesn't require authentication.

``` text
POST /mcp HTTP/1.1
Host: spanner.googleapis.com
Content-Type: application/json

{
  "jsonrpc": "2.0",
  "method": "tools/list",
}
```

## Sample use cases

The following are sample use cases for the Spanner MCP server.

### Application development with Spanner

An application developer can use the Spanner MCP server to provision resources, create databases, and populate sample data.

**Sample prompt** : Create a regional Spanner instance in the PROJECT\_ID project in the `  us-central1  ` regional instance configuration. Create a database for tracking inventory and populate 5 sample products.

Replace `  PROJECT_ID  ` with your Google Cloud project ID.

**Workflow** :

The workflow for developing an application might look like the following:

  - The agent calls the `  create_instance  ` tool to provision a new Spanner instance using the specified instance configuration. The agent might invoke the `  get_operation  ` tool to verify if the instance is ready to be used.

  - The agent calls the `  create_database  ` tool for creating a new database with the required schema. The agent might call the `  get_operation  ` tool to check the status of the database creation operation.

  - The agent can use a combination of `  create_session  ` , `  execute_sql  ` , and the `  commit  ` tools to insert sample data.

  - Optionally, the agent can call the `  execute_sql  ` tool to query and validate the sample data creation.

### Operational insights and database configuration management

Spanner administrators can use the Spanner MCP server to gather information about Spanner instances and databases using tools like `  list_instances  ` , `  get_instance  ` , `  list_databases  ` , and `  get_database_ddl  ` .

**Sample prompts** :

  - List all Spanner instances in the current project.
  - List all databases in the current Spanner instance.
  - Show the schema for the current Spanner database.

## Optional security and safety configurations

MCP introduces new security risks and considerations due to the wide variety of actions that can be taken with MCP tools. To minimize and manage these risks, Google Cloud offers default and customizable policies to control the use of MCP tools in your Google Cloud organization or project.

For more information about MCP security and governance, see [AI security and safety](/mcp/ai-security-safety) .

### Model Armor

[Model Armor](/model-armor/overview) is a Google Cloud service designed to enhance the security and safety of your AI applications. It works by proactively screening LLM prompts and responses, protecting against various risks and supporting responsible AI practices. Whether you are deploying AI in your cloud environment, or on external cloud providers, Model Armor can help you prevent malicious input, verify content safety, protect sensitive data, maintain compliance, and enforce your AI safety and security policies consistently across your diverse AI landscape.

Model Armor is only available in specific regional locations. If Model Armor is enabled for a project, and a call to that project comes from an unsupported region, Model Armor makes a cross-regional call. For more information, see [Model Armor locations](/security-command-center/docs/regional-endpoints#locations-model-armor) .

**Caution:** Model Armor logs the entire payload if a request fails. This might expose sensitive information in the logs.

#### Enable Model Armor

To enable Model Armor, complete the following steps:

1.  To enable Model Armor on your Google Cloud project, run the following gcloud CLI command:
    
    ``` text
    gcloud services enable modelarmor.googleapis.com \
        --project=PROJECT_ID
    ```
    
    Replace `  PROJECT_ID  ` with your Google Cloud project ID.

2.  To configure the recommended [floor settings for Model Armor](/security-command-center/docs/configure-model-armor-floor-settings) , run the following gcloud CLI command:
    
    ``` text
    gcloud model-armor floorsettings update \
        --full-uri='projects/PROJECT_ID/locations/global/floorSetting' \
        --mcp-sanitization=ENABLED \
        --malicious-uri-filter-settings-enforcement=ENABLED \
        --pi-and-jailbreak-filter-settings-enforcement=ENABLED \
        --pi-and-jailbreak-filter-settings-confidence-level=MEDIUM_AND_ABOVE
    ```
    
    Replace `  PROJECT_ID  ` with your Google Cloud project ID.
    
    Model Armor is configured to scan for [malicious URLs](/security-command-center/docs/model-armor-overview#ma-malicious-url-detection) and [prompt injection and jailbreak](/security-command-center/docs/model-armor-overview#ma-prompt-injection) attempts.
    
    For more information about configurable Model Armor filters, see [Model Armor filters](/security-command-center/docs/model-armor-overview#ma-filters) .

3.  To add Model Armor as a content security provider for MCP services, run the following gcloud CLI command:
    
    ``` text
    gcloud beta services mcp content-security add modelarmor.googleapis.com \
        --project=PROJECT_ID
    ```
    
    Replace `  PROJECT_ID  ` with the Google Cloud project ID.

4.  To confirm that MCP traffic is sent to Model Armor, run the following command:
    
    ``` text
    gcloud beta services mcp content-security get \
        --project=PROJECT_ID
    ```
    
    Replace `  PROJECT_ID  ` with the Google Cloud project ID.

#### Model Armor logging

For information about Model Armor audit and platform logs, see [Model Armor audit logging](/security-command-center/docs/audit-logging-model-armor) .

#### Disable Model Armor in a project

To disable Model Armor on a Google Cloud project, run the following command:

``` text
gcloud beta services mcp content-security remove modelarmor.googleapis.com \
    --project=PROJECT_ID
```

Replace `  PROJECT_ID  ` with the Google Cloud project ID.

MCP traffic on Google Cloud won't be scanned by Model Armor for the specified project.

#### Disable scanning MCP traffic with Model Armor

If you still want to use Model Armor in a project, but you want to stop scanning MCP traffic with Model Armor, then run the following command:

``` text
gcloud model-armor floorsettings update \
  --full-uri='projects/PROJECT_ID/locations/global/floorSetting' \
  --mcp-sanitization=DISABLED
```

Replace `  PROJECT_ID  ` with the Google Cloud project ID.

Model Armor won't scan MCP traffic on Google Cloud.

### Organization level MCP control

You can create custom organization policies to control the use of MCP servers in your Google Cloud organization using the `  gcp.managed.allowedMCPService  ` constraint. For more information and usage examples, see [Google Cloud MCP servers Access control with IAM](/mcp/access-control) .

## What's next

  - Read the [Spanner MCP reference documentation](/spanner/docs/reference/mcp) .
  - Learn more about [Google Cloud MCP servers](/mcp/overview) .
