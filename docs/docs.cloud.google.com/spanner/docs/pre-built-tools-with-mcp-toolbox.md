This document describes how to connect your Spanner instance to various developer tools that support the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction) .

We recommend using the dedicated Spanner extension for [Gemini CLI](/gemini/docs/codeassist/gemini-cli) . This extension abstracts away the need to set up a separate server connection. You can configure Gemini Code Assist to use the Gemini CLI, offering similar setup benefits in your IDE. For more information, see [Gemini CLI Extension - Spanner](https://github.com/gemini-cli-extensions/spanner) .

Alternatively, other IDEs and developer tools supporting the MCP can connect through [MCP Toolbox for Databases](https://github.com/googleapis/genai-toolbox) . MCP Toolbox is an open-source MCP server designed to connect AI agents to your data. It handles tasks like authentication and connection pooling, letting you interact with your data with natural language directly from your IDE.

## Use the Gemini CLI extension in Spanner

**Note:** MCP Toolbox for Databases is in beta (pre-v1.0), and may see breaking changes until the first stable release. (v1.0)

The Spanner integration with Gemini CLI is through an open-source extension that offers additional capabilities compared to the standard MCP Toolbox connection. The extension offers an installation process and a set of tools, in addition to providing detailed information on installation, configuration, and usage examples. If you use the Gemini CLI extension, you don't need to install MCP Toolbox. For more information, see [Gemini CLI Extension - Spanner](https://github.com/gemini-cli-extensions/spanner) .

The `  spanner  ` extension includes tools for listing tables, and executing SQL and SQL DQL statements.

<table>
<thead>
<tr class="header">
<th>Tools</th>
<th>Example natural language prompt</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       list_tables      </code></td>
<td>What tables do I have in my Spanner instance?</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       execute_sql      </code></td>
<td>Insert test data into the products table.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       execute_sql_dql      </code></td>
<td>What products in the electronics category are sold in America?</td>
</tr>
</tbody>
</table>

## Before you begin

1.  In the Google Cloud console, on the [project selector page](https://console.cloud.google.com/projectselector2/home/dashboard) , select or create a Google Cloud project.

2.  [Make sure that billing is enabled for your Google Cloud project](/billing/docs/how-to/verify-billing-enabled#confirm_billing_is_enabled_on_a_project) .

## Set up the Spanner instance

1.  [Enable the Spanner API in the Google Cloud project](https://console.cloud.google.com/flows/enableapi?apiid=spanner.googleapis.com&redirect=https://console.cloud.google.com/) .

2.  [Create or select a Spanner instance and database](/spanner/docs/create-query-database-console) .

3.  Configure the required roles and permissions to complete this task. The user invoking the LLM agents needs the following roles at the database level:
    
      - Cloud Spanner Database Reader ( `  roles/spanner.databaseReader  ` ) to execute DQL queries and list tables.
    
      - Cloud Spanner Database User ( `  roles/spanner.databaseUser  ` ) to execute DML queries.

4.  Configure [Application Default Credentials (ADC)](/docs/authentication/set-up-adc-local-dev-environment) for your environment.

## Install MCP Toolbox

**Note:** MCP Toolbox is only required for MCP clients other than Gemini CLI, Gemini Code Assist, or Antigravity. Skip this section if you are using Gemini CLI, Gemini Code Assist, or Antigravity.

1.  Download the latest version of MCP Toolbox as a binary. Select the [binary](https://github.com/googleapis/genai-toolbox/releases/latest) corresponding to your operating system (OS) and CPU architecture. You must use MCP Toolbox version 0.15.0 or later:
    
    ### linux/amd64
    
    ``` text
    curl -O https://storage.googleapis.com/genai-toolbox/version/linux/amd64/toolbox
    ```
    
    ### darwin/arm64
    
    ``` text
    curl -O https://storage.googleapis.com/genai-toolbox/version/darwin/arm64/toolbox
    ```
    
    ### darwin/amd64
    
    ``` text
    curl -O https://storage.googleapis.com/genai-toolbox/version/darwin/amd64/toolbox
    ```
    
    ### windows/amd64
    
    ``` text
    curl -O https://storage.googleapis.com/genai-toolbox/version/windows/amd64/toolbox
    ```

2.  Make the binary executable:
    
    ``` text
    chmod +x toolbox
    ```

3.  Verify the installation:
    
    ``` text
    ./toolbox --version
    ```

## Set up the agent tool

This section describes how to configure various developer tools to connect to your Spanner instance. Select your agent tool from the following options:

### Gemini CLI

1.  Install the [Gemini CLI.](https://github.com/google-gemini/gemini-cli?tab=readme-ov-file#-installation)  

2.  Install the Spanner extension for Gemini CLI from the GitHub repository using the following command:  

3.  Set the following environment variables to connect to your Spanner instance:  
    
    ``` text
        export SPANNER_PROJECT="PROJECT_ID"
        export SPANNER_INSTANCE="INSTANCE_NAME"
        export SPANNER_DATABASE="DATABASE_NAME"
        export SPANNER_DIALECT="DIALECT_NAME"
        
    ```
    
    Replace the following:
    
      - `  PROJECT_ID  ` : your Google Cloud project ID.
      - `  INSTANCE_NAME  ` : your Spanner instance name.
      - `  DATABASE_NAME  ` : your Spanner database name.
      - `  DIALECT_NAME  ` : your Spanner SQL dialect. Accepts `  googlesql  ` or `  postgresql  ` . Defaults to `  googlesql  ` if undefined.

4.  Start the Gemini CLI in interactive mode:
    
    ``` text
        gemini
        
    ```
    
    The CLI automatically loads the Spanner extension for Gemini CLI and its tools, which you can use to interact with your database.

### Gemini Code Assist

We strongly recommend configuring Gemini Code Assist to use the [Gemini CLI](#mcp-configure-your-mcp-client-geminicli) , as this approach removes the need to manually configure an MCP server. However, the directions to manually configure an MCP server are still available in the following section:

  
1\. Install the [Gemini Code Assist](https://marketplace.visualstudio.com/items?itemName=Google.geminicodeassist) extension in VS Code.  
2\. [Enable agent mode](https://code.visualstudio.com/docs/copilot/chat/chat-agent-mode) and switch the agent model to Gemini.  
3\. In your project root directory, create a folder named `  .gemini  ` and, within it, a `  settings.json  ` file.  
4\. Add one of the following configurations based on your Spanner dialect in the `  settings.json  ` file.  
5\. Replace the following variables with your values:  

  - `  PROJECT_ID  ` : your Google Cloud project ID.
  - `  INSTANCE_NAME  ` : your Spanner instance name.
  - `  DATABASE_NAME  ` : your Spanner database name.

6\. Save the file.  
  
Spanner with **GoogleSQL** dialect:  
  

``` text
{
  "mcpServers": {
    "spanner": {
      "command": "./PATH/TO/toolbox",
      "args": ["--prebuilt","spanner","--stdio"],
      "env": {
          "SPANNER_PROJECT": "PROJECT_ID",
          "SPANNER_INSTANCE": "INSTANCE_NAME",
          "SPANNER_DATABASE": "DATABASE_NAME"
      }
    }
  }
}
```

  
Spanner with **PostgreSQL** dialect:  
  

``` text
{
  "mcpServers": {
    "spanner": {
      "command": "./PATH/TO/toolbox",
      "args": ["--prebuilt","spanner-postgres","--stdio"],
      "env": {
          "SPANNER_PROJECT": "PROJECT_ID",
          "SPANNER_INSTANCE": "INSTANCE_NAME",
          "SPANNER_DATABASE": "DATABASE_NAME"
      }
    }
  }
}
```

### Claude Code

  
1\. Install [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview) .  
2\. Create `  .mcp.json  ` file in your project root, if it doesn't exist.  
3\. Add one of the following configurations based on your Spanner dialect, replace the environment variables with your values, and save the file:  
  
Spanner with **GoogleSQL** dialect:  
  

``` text
{
  "mcpServers": {
    "spanner": {
      "command": "./PATH/TO/toolbox",
      "args": ["--prebuilt","spanner","--stdio"],
      "env": {
          "SPANNER_PROJECT": "PROJECT_ID",
          "SPANNER_INSTANCE": "INSTANCE_NAME",
          "SPANNER_DATABASE": "DATABASE_NAME"
      }
    }
  }
}
```

  
Spanner with **PostgreSQL** dialect:  
  

``` text
{
  "mcpServers": {
    "spanner": {
      "command": "./PATH/TO/toolbox",
      "args": ["--prebuilt","spanner-postgres","--stdio"],
      "env": {
          "SPANNER_PROJECT": "PROJECT_ID",
          "SPANNER_INSTANCE": "INSTANCE_NAME",
          "SPANNER_DATABASE": "DATABASE_NAME"
      }
    }
  }
}
```

### Claude for Desktop

  
1\. Open [Claude for Desktop](https://claude.ai/download) and navigate to **Settings** .  
2\. In the **Developer** tab, click **Edit Config** to open the configuration file.  
3\. Add one of the following configurations based on your Spanner dialect, replace the environment variables with your values, and save the file:  
  
Spanner with **GoogleSQL** dialect:  
  

``` text
{
  "mcpServers": {
    "spanner": {
      "command": "./PATH/TO/toolbox",
      "args": ["--prebuilt","spanner","--stdio"],
      "env": {
          "SPANNER_PROJECT": "PROJECT_ID",
          "SPANNER_INSTANCE": "INSTANCE_NAME",
          "SPANNER_DATABASE": "DATABASE_NAME"
      }
    }
  }
}
```

  
Spanner with **PostgreSQL** dialect:  
  

``` text
{
  "mcpServers": {
    "spanner": {
      "command": "./PATH/TO/toolbox",
      "args": ["--prebuilt","spanner-postgres","--stdio"],
      "env": {
          "SPANNER_PROJECT": "PROJECT_ID",
          "SPANNER_INSTANCE": "INSTANCE_NAME",
          "SPANNER_DATABASE": "DATABASE_NAME"
      }
    }
  }
}
```

  
4\. Restart Claude for Desktop.  
5\. The new chat screen displays a hammer (MCP) icon with the new MCP server.  
  

### Cline

  
1\. Open [Cline](https://github.com/cline/cline) extension in VS Code and click **MCP Servers** icon.  
2\. Tap Configure MCP Servers to open the configuration file.  
3\. Add one of the following configurations based on your Spanner dialect, replace the environment variables with your values, and save the file:  
  
Spanner with **GoogleSQL** dialect:  
  

``` text
{
  "mcpServers": {
    "spanner": {
      "command": "./PATH/TO/toolbox",
      "args": ["--prebuilt","spanner","--stdio"],
      "env": {
          "SPANNER_PROJECT": "PROJECT_ID",
          "SPANNER_INSTANCE": "INSTANCE_NAME",
          "SPANNER_DATABASE": "DATABASE_NAME"
      }
    }
  }
}
```

  
Spanner with **PostgreSQL** dialect:  
  

``` text
{
  "mcpServers": {
    "spanner": {
      "command": "./PATH/TO/toolbox",
      "args": ["--prebuilt","spanner-postgres","--stdio"],
      "env": {
          "SPANNER_PROJECT": "PROJECT_ID",
          "SPANNER_INSTANCE": "INSTANCE_NAME",
          "SPANNER_DATABASE": "DATABASE_NAME"
      }
    }
  }
}
```

  
A green active status appears after the server connects successfully.  
  

### Cursor

  
1\. Create the `  .cursor  ` directory in your project root if it doesn't exist.  
2\. Create the `  .cursor/mcp.json  ` file if it doesn't exist and open it.  
3\. Add one of the following configurations based on your Spanner dialect, replace the environment variables with your values, and save the file:  
  
Spanner with **GoogleSQL** dialect:  
  

``` text
{
  "mcpServers": {
    "spanner": {
      "command": "./PATH/TO/toolbox",
      "args": ["--prebuilt","spanner","--stdio"],
      "env": {
          "SPANNER_PROJECT": "PROJECT_ID",
          "SPANNER_INSTANCE": "INSTANCE_NAME",
          "SPANNER_DATABASE": "DATABASE_NAME"
      }
    }
  }
}
```

  
Spanner with **PostgreSQL** dialect:  
  

``` text
{
  "mcpServers": {
    "spanner": {
      "command": "./PATH/TO/toolbox",
      "args": ["--prebuilt","spanner-postgres","--stdio"],
      "env": {
          "SPANNER_PROJECT": "PROJECT_ID",
          "SPANNER_INSTANCE": "INSTANCE_NAME",
          "SPANNER_DATABASE": "DATABASE_NAME"
      }
    }
  }
}
```

  
4\. Open [Cursor](https://www.cursor.com/) and navigate to **Settings \> Cursor Settings \> MCP** . A green active status appears when the server connects.  
  

### Visual Studio Code (Copilot)

  
1\. Open [VS Code](https://code.visualstudio.com/docs/copilot/overview) and create `  .vscode  ` directory in your project root if it does not exist.  
2\. Create the `  .vscode/mcp.json  ` file if it doesn't exist, and open it.  
3\. Add one of the following configurations based on your Spanner dialect, replace the environment variables with your values, and save the file:  
  
Spanner with **GoogleSQL** dialect:  
  

``` text
{
  "servers": {
    "spanner": {
      "command": "./PATH/TO/toolbox",
      "args": ["--prebuilt","spanner","--stdio"],
      "env": {
        "SPANNER_PROJECT": "PROJECT_ID",
        "SPANNER_INSTANCE": "INSTANCE_NAME",
        "SPANNER_DATABASE": "DATABASE_NAME"
      }
    }
  }
}
```

  
Spanner with **PostgreSQL** dialect:  
  

``` text
{
  "servers": {
    "spanner": {
      "command": "./PATH/TO/toolbox",
      "args": ["--prebuilt","spanner-postgres","--stdio"],
      "env": {
        "SPANNER_PROJECT": "PROJECT_ID",
        "SPANNER_INSTANCE": "INSTANCE_NAME",
        "SPANNER_DATABASE": "DATABASE_NAME"
      }
    }
  }
}
```

### Windsurf

  
1\. Open [Windsurf](https://docs.codeium.com/windsurf) and navigate to Cascade assistant.  
2\. Click the MCP icon, then click **Configure** to open the configuration file.  
3\. Add one of the following configurations based on your Spanner dialect, replace the environment variables with your values, and save the file:  
  
Spanner with **GoogleSQL** dialect:  
  

``` text
{
  "mcpServers": {
    "spanner": {
      "command": "./PATH/TO/toolbox",
      "args": ["--prebuilt","spanner","--stdio"],
      "env": {
          "SPANNER_PROJECT": "PROJECT_ID",
          "SPANNER_INSTANCE": "INSTANCE_NAME",
          "SPANNER_DATABASE": "DATABASE_NAME"
      }
    }
  }
}
```

  
Spanner with **PostgreSQL** dialect:  
  

``` text
{
  "mcpServers": {
    "spanner": {
      "command": "./PATH/TO/toolbox",
      "args": ["--prebuilt","spanner-postgres","--stdio"],
      "env": {
          "SPANNER_PROJECT": "PROJECT_ID",
          "SPANNER_INSTANCE": "INSTANCE_NAME",
          "SPANNER_DATABASE": "DATABASE_NAME"
      }
    }
  }
}
```

## Connect with Antigravity

You can connect Spanner to Antigravity in the following ways:

  - Using the MCP Store
  - Using a custom configuration

### MCP Store

The most recommended way to connect to [Antigravity](https://antigravity.google/docs/mcp) is by using the built-in MCP Store.

1.  Open [Antigravity](https://antigravity.google/docs/mcp) and open the editor's **agent panel** .
2.  Click the **Menu** icon at the top of the panel and select **MCP Servers** .
3.  Locate **Spanner** in the list of available servers and click **Install** .
4.  Follow the on-screen steps to authorize Antigravity to access your Google Cloud Project. This lets Antigravity access the Spanner instance on your project.

Once you install the Spanner server in the MCP Store, the resources and tools from the server are available to the editor.

### Custom configuration

To connect to a custom MCP server, do the following steps:

1.  Open [Antigravity](https://antigravity.google/docs/mcp) and open the editor's **agent panel** .
2.  Click the **Menu** icon at the top of the panel and select **MCP Servers** .
3.  Click **Manage MCP Servers \> View raw config** to open the `  mcp_config.json  ` file.
4.  Add the following configuration, replace the environment variables with your values, and save.

<!-- end list -->

``` text
{
  "mcpServers": {
    "spanner": {
      "command": "npx",
      "args": ["-y","@toolbox-sdk/server","--prebuilt","spanner","--stdio"],
      "env": {
          "SPANNER_PROJECT": "PROJECT_ID",
          "SPANNER_INSTANCE": "INSTANCE_NAME",
          "SPANNER_DATABASE": "DATABASE_NAME",
          "SPANNER_DIALECT": "DIALECT_NAME"
      }
    }
  }
}
```

Once you configure the custom MCP server, the resources and tools from the Spanner server are available to the editor.

Replace the following:

  - `  PROJECT_ID  ` : your Google Cloud project ID.
  - `  INSTANCE_NAME  ` : your Spanner instance name.
  - `  DATABASE_NAME  ` : your Spanner database name.
  - `  DIALECT_NAME  ` : your Spanner SQL dialect. Accepts `  googlesql  ` or `  postgresql  ` . If you don't specify a dialect, the default is `  googlesql  ` .
