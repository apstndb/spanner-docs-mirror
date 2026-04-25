This document shows you how to use the Spanner remote Model Context Protocol (MCP) server to connect with AI applications including Gemini CLI, ChatGPT, Claude, and custom applications you are developing. The Spanner MCP server lets you access and run Spanner tools to create, manage, and query Spanner resources from your AI-enabled development environments and AI agent platforms. .

The Spanner remote MCP server is enabled when you enable the Spanner API.

[Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro) (MCP) standardizes how large language models (LLMs) and AI applications or agents connect to external data sources. MCP servers let you use their tools, resources, and prompts to take actions and get updated data from their backend service.

## What's the difference between local and remote MCP servers?

  - Local MCP servers  
    Typically run on your local machine and use the standard input and output streams (stdio) for communication between services on the same device.
  - Remote MCP servers  
    Run on the service's infrastructure and offer an HTTP endpoint to AI applications for communication between the AI MCP client and the MCP server. For more information about MCP architecture, see [MCP architecture](https://modelcontextprotocol.io/docs/learn/architecture) .

For information on the Spanner local MCP server, see [Spanner MCP server on GitHub](https://googleapis.github.io/genai-toolbox/resources/sources/spanner/) .

## Google and Google Cloud remote MCP servers

Google and Google Cloud remote MCP servers have the following features and benefits:

  - Simplified, centralized discovery
  - Managed global or regional HTTP endpoints
  - Fine-grained authorization
  - Optional prompt and response security with Model Armor protection
  - Centralized audit logging

For information about other MCP servers and information about security and governance controls available for Google Cloud MCP servers, see [Google Cloud MCP servers overview](https://docs.cloud.google.com/mcp/overview) .

## Before you begin

1.  Enable the Spanner API.
    
    **Roles required to enable APIs**
    
    To enable APIs, you need the Service Usage Admin IAM role ( `roles/serviceusage.serviceUsageAdmin` ), which contains the `serviceusage.services.enable` permission. [Learn how to grant roles](https://docs.cloud.google.com/iam/docs/granting-changing-revoking-access) .
    
    For new projects, the Spanner API is automatically enabled.

### Required roles

To get the permissions that you need to use the Spanner MCP server, ask your administrator to grant you the following IAM roles on the project where you want to use the Spanner MCP server:

  - Make MCP tool calls: [MCP Tool User](https://docs.cloud.google.com/iam/docs/roles-permissions/mcp#mcp.toolUser) ( `roles/mcp.toolUser` )
  - Create an OAuth client ID: [OAuth Config Editor](https://docs.cloud.google.com/iam/docs/roles-permissions/oauthconfig#oauthconfig.editor) ( `roles/oauthconfig.editor` )
  - Use Spanner MCP tools: [Cloud Spanner Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/spanner#spanner.admin) ( `roles/spanner.admin` )

For more information about granting roles, see [Manage access to projects, folders, and organizations](https://docs.cloud.google.com/iam/docs/granting-changing-revoking-access) .

These predefined roles contain the permissions required to use the Spanner MCP server. To see the exact permissions that are required, expand the **Required permissions** section:

#### Required permissions

The following permissions are required to use the Spanner MCP server:

  - Make MCP tool calls: `mcp.tools.call`
  - Use Spanner MCP tools:
      - `spanner.instances.create`
      - `spanner.instances.get`
      - `spanner.databases.create`
      - `spanner.databases.update`
      - `spanner.sessions.create`
      - `spanner.instanceOperations.get`
      - `spanner.databases.getDdl`
      - `spanner.databases.select`
      - `spanner.databases.write`

You might also be able to get these permissions with [custom roles](https://docs.cloud.google.com/iam/docs/creating-custom-roles) or other [predefined roles](https://docs.cloud.google.com/iam/docs/roles-overview#predefined) .

## Authentication and authorization

Spanner MCP servers use the [OAuth 2.0](https://developers.google.com/identity/protocols/oauth2) protocol with [Identity and Access Management (IAM)](https://docs.cloud.google.com/iam/docs/overview) for authentication and authorization. All [Google Cloud identities](https://docs.cloud.google.com/docs/authentication/identity-products) are supported for authentication to MCP servers.

The Spanner remote MCP server doesn't accept API keys.

We recommend creating a separate identity for agents using MCP tools so that access to resources can be controlled and monitored. For more information on authentication, see [Authenticate to MCP servers](https://docs.cloud.google.com/mcp/authenticate-mcp) .

## Spanner MCP OAuth scopes

OAuth 2.0 uses scopes and credentials to determine if an authenticated principal is authorized to take a specific action on a resource. For more information about OAuth 2.0 scopes at Google, read [Using OAuth 2.0 to access Google APIs](https://developers.google.com/identity/protocols/oauth2) .

Spanner has the following MCP tool OAuth scopes:

| Scope URI for gcloud CLI                        | Description                                                       |
| ----------------------------------------------- | ----------------------------------------------------------------- |
| `https://www.googleapis.com/auth/spanner.admin` | Allows access to administer your Spanner instances and databases. |
| `https://www.googleapis.com/auth/spanner.data`  | Allows access to view and manage data in a Spanner database.      |

For more information about these scopes, see [Spanner API](https://developers.google.com/identity/protocols/oauth2/scopes#spanner) .

## Configure an MCP client to use the Spanner MCP server

Host programs, such as Claude or the Gemini CLI, can instantiate MCP clients that connect to a single MCP server. A host program can have multiple clients that connect to different MCP servers. To connect to a remote MCP server, the MCP client must know at a minimum the URL of the remote MCP server.

Use the following instructions to configure MCP clients to connect to your remote Spanner MCP server.

### Gemini CLI

To add a Spanner remote MCP server to your Gemini CLI, configure it as an extension.

1.  Create an extension file in the following location: `~/.gemini/extensions/ EXT_NAME /gemini-extension.json` where `~/` is your home directory and EXT\_NAME is the name you want to give the extension.

2.  Add the following content to your extension file:
    
    ``` 
            {
              "name": "EXT_NAME",
              "version": "1.0.0",
              "mcpServers": {
                "Spanner MCP Server": {
                  "httpUrl": "https://spanner.googleapis.com/mcp",
                  "authProviderType": "google_credentials",
                  "oauth": {
                    "scopes": ["https://www.googleapis.com/auth/cloud-platform"]
                  },
                  "timeout": 30000,
                  "headers": {
                    "x-goog-user-project": "PROJECT_ID"
                  }
                }
              }
            }
            
    ```

3.  Save the extensions file.

4.  Start Gemini CLI:
    
    ``` 
            gemini
            
    ```

5.  Run `/mcp` in the CLI to view your configured MCP server and its tools.
    
    The response is similar to the following:
    
    ``` 
            Configured MCP servers:
            🟢 Spanner MCP Server (from spanner )
              - get_database_ddl
              - get_instance
              - get_operation
              - create_database
              - create_instance
              - create_session
              - commit
              - execute_sql
              - list_databases
              - list_instances
            
    ```

The remote MCP server is ready to use in Gemini CLI.

### Claude.ai

You must have the Claude Enterprise, Pro, Max, or Team plan to configure Google and Google Cloud MCP servers in Claude.ai. For pricing information, see [Claude Pricing](https://claude.com/pricing) .

To add a Spanner remote MCP server to Claude.ai, configure a custom connector with an OAuth client ID and OAuth client secret:

#### Create an Oauth 2.0 client ID and secret

1.  In the Google Cloud console, go to **Google Auth Platform \> Clients \> Create client** .
    
    You are prompted to create a project if you don't have one selected.

2.  In the **Application type** list, select **Web application** .

3.  In the **Name** field, enter a name for your application.

4.  In the **Authorized redirect URIs** section, click **+ Add URI** , and then add `https://claude.ai/api/mcp/auth_callback` in the **URIs** field.

5.  Click **Create** . The client is created. To access the client ID, in the Google Cloud console, go to **Google Auth Platform \> Clients** .

6.  In the **OAuth 2.0 client IDs** list, select the client name.

7.  In the **Client secrets** section, copy the **Client secret** and save it in a secure place. You can only copy it once. If you lose it, delete the secret and create a new one.
    
    > **Caution:** Treat client secrets like passwords and store them in a secure place.

#### Create a custom connector in Claude.ai

1.  In Claude.ai, navigate to the Connectors settings for your plan:
    
      - For the Enterprise or Team plan, navigate to **Admin settings \> Connectors** .
      - For the Pro or Max plan, navigate to **Settings \> Connectors** .

2.  Click **Add custom connector** .

3.  In the **Add custom connector** dialog, enter the following:
    
      - **Server name** : a human readable name for the server.
      - **Remote MCP server URL** : `https://spanner.googleapis.com/mcp`

4.  Expand the **Advanced settings** menu and then enter the following:
    
      - **OAuth client ID** : the OAuth 2.0 client ID you created.
      - **OAuth client secret** (Pro and Max plans only): the secret for your OAuth 2.0 client. To retrieve the secret, go to **Google Auth Platform \> Clients** and then select the OAuth client ID you created. In the **Client secrets** section, click to copy the **Client secret** .

5.  Click **Add** .
    
    The custom connector is created.

6.  Open the **Tools** menu and enable the connector.
    
    Claude.ai can use the MCP server.

### ChatGPT

You must have a [ChatGPT Business subscription](https://chatgpt.com/business/) to use Google and Spanner MCP servers with ChatGPT.

To add a Spanner remote MCP server to ChatGPT, create a Google OAuth 2.0 client ID and secret, and then add the MCP server as an App in ChatGPT.

#### Create an Oauth 2.0 client ID and secret

1.  In the Google Cloud console, go to **Google Auth Platform \> Clients \> Create client** .
    
    You are prompted to create a project if you don't have one selected.

2.  In the **Application type** list, select **Web application** .

3.  In the **Name** field, enter a name for your application.

4.  In the **Authorized JavaScript origins** section, click **+ Add URI** , and then add `https://chatgpt.com` in the **URIs** field.

5.  In the **Authorized redirect URIs** section, click **+ Add URI** , and then add `https://chatgpt.com/connector_platform_oauth_redirect` in the **URIs** field.

6.  Click **Create** . The client is created. To access the client ID, in the Google Cloud console, go to **Google Auth Platform \> Clients** .

7.  In the **OAuth 2.0 client IDs** list, select the client name.

8.  In the **Client secrets** section, copy the **Client secret** and save it in a secure place. You can only copy it once. If you lose it, delete the secret and create a new one.
    
    > **Caution:** Treat client secrets like passwords and store them in a secure place.

#### Add the MCP server as an app in ChatGPT

1.  Sign in to ChatGPT.
2.  Turn on Developer mode:
    1.  In ChatGPT, click your username to open the **Profile menu** , and then select **Settings** .
    2.  In the Settings menu, select **Apps** , and then click **Advanced settings** .
    3.  In the **Advanced settings** , click the **Developer mode** toggle to the on position.
3.  In **Settings** \> **Apps** , click the **Create app** button.
4.  In the **New app** dialog, enter the following information:
      - **Name** : the name of the MCP server.
      - **Description** : an optional description of the MCP server.
      - **MCP server URL** : `https://spanner.googleapis.com/mcp`
      - **Authentication** :
          - In the **Authentication** menu, select **OAuth** .
          - In the **OAuth client ID** field, enter your Google OAuth client ID.
          - In the **OAuth secret** field, enter your Google OAuth client secret.
      - Confirm that you understand the risk associated with MCP server use, and then click **Create** .

The MCP server is displayed in the **Apps** menu, and is ready for use through chat prompts.

## General guidance for MCP clients

If specific instructions for your MCP client aren't included in [Configure an MCP client to use the Spanner MCP server](https://docs.cloud.google.com/spanner/docs/use-spanner-mcp#configure-client) , use the following information to connect to a remote MCP server in your host program or AI application. You are prompted to enter details about the server, such as its name and URL.

For the Spanner remote MCP server, enter the following information:

  - **Server name** : Spanner MCP server
  - **Server URL** or **Endpoint** : `https://spanner.googleapis.com/mcp`
  - **Transport** : HTTP
  - **Authentication details** : Depending on how you want to authenticate, you can enter your Google Cloud credentials, your OAuth Client ID and secret, or an agent identity and credentials.

For more general guidance, see the following resources:

  - [Authenticate to MCP servers](https://docs.cloud.google.com/mcp/authenticate-mcp) .
  - [Configure MCP in an AI application](https://docs.cloud.google.com/mcp/configure-mcp-ai-application) .

## Available tools

To view details of available MCP tools and their descriptions for the Spanner MCP server, see the [Spanner MCP reference](https://docs.cloud.google.com/spanner/docs/reference/mcp) .

### List tools

Use the [MCP inspector](https://modelcontextprotocol.io/docs/tools/inspector) to list tools, or send a `tools/list` HTTP request directly to the Spanner remote MCP server. The `tools/list` method doesn't require authentication.

    POST /mcp HTTP/1.1
    Host: spanner.googleapis.com
    Content-Type: application/json
    
    {
      "jsonrpc": "2.0",
      "method": "tools/list",
    }

## Observability

The Spanner MCP server supports Spanner introspection and observability tools.

### Request Tags

The queries executed or transactions committed using the Spanner MCP server are autotagged with specific request tags. You can use these tags to debug queries and transactions. For more information, see [Troubleshoot with request tags and transaction tags](https://docs.cloud.google.com/spanner/docs/introspection/troubleshooting-with-tags) .

| Tool name              | Request tag                |
| ---------------------- | -------------------------- |
| `execute_sql`          | `mcp_execute_sql`          |
| `execute_sql_readonly` | `mcp_execute_sql_readonly` |
| `commit`               | `mcp_commit`               |

## Sample use cases

The following are sample use cases for the Spanner MCP server.

### Application development with Spanner

An application developer can use the Spanner MCP server to provision resources, create databases, and populate sample data.

**Sample prompt** : "Create a regional Spanner instance in the PROJECT\_ID project in the `us-central1` regional instance configuration. Create a database for tracking inventory and populate 5 sample products."

Replace `PROJECT_ID` with your Google Cloud project ID.

**Workflow** :

The workflow for developing an application might look like the following:

  - The agent calls the `create_instance` tool to provision a new Spanner instance using the specified instance configuration. The agent might invoke the `get_operation` tool to verify if the instance is ready to be used.

  - The agent calls the `create_database` tool for creating a new database with the required schema. The agent might call the `get_operation` tool to check the status of the database creation operation.

  - The agent can use a combination of `create_session` , `execute_sql` , and the `commit` tools to insert sample data.

  - Optionally, the agent can call the `execute_sql` tool to query and validate the sample data creation.

### Operational insights and database configuration management

Spanner administrators can use the Spanner MCP server to gather information about Spanner instances and databases using tools like `list_instances` , `get_instance` , `list_databases` , and `get_database_ddl` .

**Sample prompts** :

  - "List all Spanner instances in the current project."
  - "List all databases in the current Spanner instance."
  - "Show the schema for the current Spanner database."

## Optional security and safety configurations

MCP introduces new security risks and considerations due to the wide variety of actions that you can do with the MCP tools. To minimize and manage these risks, Google Cloud offers default settings and customizable policies to control the use of MCP tools in your Google Cloud organization or project.

For more information about MCP security and governance, see [AI security and safety](https://docs.cloud.google.com/mcp/ai-security-safety) .

### Use Model Armor

[Model Armor](https://docs.cloud.google.com/model-armor/overview) is a Google Cloud service designed to enhance the security and safety of your AI applications. It works by proactively screening LLM prompts and responses, protecting against various risks and supporting responsible AI practices. Whether you are deploying AI in your cloud environment, or on external cloud providers, Model Armor can help you prevent malicious input, verify content safety, protect sensitive data, maintain compliance, and enforce your AI safety and security policies consistently across your diverse AI landscape.

When Model Armor is enabled with [logging enabled](https://docs.cloud.google.com/model-armor/configure-logging) , Model Armor logs the entire payload. This might expose sensitive information in your logs.

> **Caution:** Model Armor is available in [certain regions](https://docs.cloud.google.com/model-armor/locations) . When Model Armor is enabled and you use an MCP server in a jurisdiction that Model Armor doesn't support, the routing behavior of the call might be different for different MCP servers. For more information about the behavior of individual MCP servers, see [Model Armor supported products](https://docs.cloud.google.com/mcp/model-armor-supported-products) .

#### Enable Model Armor

You must enable Model Armor APIs before you can use Model Armor.

### Console

1.  Enable the Model Armor API.
    
    **Roles required to enable APIs**
    
    To enable APIs, you need the Service Usage Admin IAM role ( `roles/serviceusage.serviceUsageAdmin` ), which contains the `serviceusage.services.enable` permission. [Learn how to grant roles](https://docs.cloud.google.com/iam/docs/granting-changing-revoking-access) .

2.  Select the project where you want to activate Model Armor.

### gcloud

Before you begin, follow these steps using the Google Cloud CLI with the Model Armor API:

1.  In the Google Cloud console, activate Cloud Shell.
    
    At the bottom of the Google Cloud console, a [Cloud Shell](https://docs.cloud.google.com/shell/docs/how-cloud-shell-works) session starts and displays a command-line prompt. Cloud Shell is a shell environment with the Google Cloud CLI already installed and with values already set for your current project. It can take a few seconds for the session to initialize.

2.  Run the following command to set the API endpoint for the Model Armor service.
    
        gcloud config set api_endpoint_overrides/modelarmor "https://modelarmor.LOCATION.rep.googleapis.com/"
    
    Replace `  LOCATION  ` with the region where you want to use Model Armor.

#### Configure protection for Google and Google Cloud remote MCP servers

To help protect your MCP tool calls and responses you can use Model Armor floor settings. A floor setting defines the minimum security filters that apply across the project. This configuration applies a consistent set of filters to all MCP tool calls and responses within the project.

> **Tip:** Don't enable the prompt injection and jailbreak filter unless your MCP traffic carries natural language data.

Set up a Model Armor floor setting with MCP sanitization enabled. For more information, see [Configure Model Armor floor settings](https://docs.cloud.google.com/model-armor/configure-floor-settings) .

> **Note:** If the agent and the MCP server are in different projects, you can create floor settings in both projects (the client project and the resource project). In this case, Model Armor is invoked twice, once for each project.

See the following example command:

    gcloud model-armor floorsettings update \
    --full-uri='projects/PROJECT_ID/locations/global/floorSetting' \
    --enable-floor-setting-enforcement=TRUE \
    --add-integrated-services=GOOGLE_MCP_SERVER \
    --google-mcp-server-enforcement-type=INSPECT_AND_BLOCK \
    --enable-google-mcp-server-cloud-logging \
    --malicious-uri-filter-settings-enforcement=ENABLED \
    --add-rai-settings-filters='[{"confidenceLevel": "MEDIUM_AND_ABOVE", "filterType": "DANGEROUS"}]'

Replace `  PROJECT_ID  ` with your Google Cloud project ID.

Note the following settings:

  - `INSPECT_AND_BLOCK` : The enforcement type that inspects content for the Google MCP server and blocks prompts and responses that match the filters.
  - `ENABLED` : The setting that enables a filter or enforcement.
  - `MEDIUM_AND_ABOVE` : The confidence level for the Responsible AI - Dangerous filter settings. You can modify this setting, though lower values might result in more false positives. For more information, see [Model Armor confidence levels](https://docs.cloud.google.com/model-armor/overview#ma-confidence-levels) .

#### Disable scanning MCP traffic with Model Armor

To stop Model Armor from automatically scanning traffic to and from Google MCP servers based on the project's floor settings, run the following command:

    gcloud model-armor floorsettings update \
      --full-uri='projects/PROJECT_ID/locations/global/floorSetting' \
      --remove-integrated-services=GOOGLE_MCP_SERVER

Replace `  PROJECT_ID  ` with the Google Cloud project ID. Model Armor doesn't automatically apply the rules defined in this project's floor settings to any Google MCP server traffic.

Model Armor floor settings and general configuration can impact more than just MCP. Because Model Armor integrates with services like Vertex AI, any changes you make to floor settings can affect traffic scanning and safety behaviors across all integrated services, not just MCP.

### Control MCP use with IAM deny policies

[Identity and Access Management (IAM) deny policies](https://docs.cloud.google.com/iam/docs/deny-overview) help you secure Google Cloud remote MCP servers. Configure these policies to block unwanted MCP tool access.

For example, you can deny or allow access based on:

  - The principal
  - Tool properties like read-only
  - The application's OAuth client ID

For more information, see [Control MCP use with Identity and Access Management](https://docs.cloud.google.com/mcp/control-mcp-use-iam) .

## What's next

  - Read the [Spanner MCP reference documentation](https://docs.cloud.google.com/spanner/docs/reference/mcp) .
  - Learn more about [Google Cloud MCP servers](https://docs.cloud.google.com/mcp/overview) .
