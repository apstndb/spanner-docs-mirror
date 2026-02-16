**Preview — Data agents**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) , and the [Additional Terms for Generative AI Preview Products](https://cloud.google.com/trustedtester/aitos) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

For information about access to this release, see the [access request page](https://forms.gle/pJByTWfenZAWbaXo7) .

**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

This document describes how to use the Gemini CLI and the MCP toolbox to build agent context files. These files contain templates and facets that provide context for generating SQL queries from natural language. You will also use the DB Context Enrichment MCP Server.

To learn about data agents, see [Data agents overview](/spanner/docs/data-agent-overview) .

To a build an agent context file, perform the following high-level steps:

  - Prepare your environment
  - Generate targeted templates
  - Generate targeted facets
  - Optional. Generate bulk templates

## Before you begin

Complete the following prerequisites before creating an agent.

### Enable required services

Enable the following services for your project:

  - [Gemini Data Analytics API](https://console.cloud.google.com/apis/library/geminidataanalytics.googleapis.com)
  - [Gemini for Google Cloud API](https://console.cloud.google.com/apis/library/cloudaicompanion.googleapis.com)

### Prepare a Spanner instance

  - Make sure that a Spanner instance is available. For more information, see [Create an instance](/spanner/docs/create-manage-instances) .
  - Ensure that you create a database in your instance where you will create the tables. For more information, see [Create a database on the Spanner instance](/spanner/docs/create-manage-databases#create-database)

### Required roles and permissions

  - Add an IAM user or service account to the cluster. For more information, see [Apply IAM roles](/spanner/docs/grant-permissions) .
  - Grant the `  spanner.databaseReader  ` roles to the IAM user at the project level. For more information, see [Add IAM policy binding for a project](/sdk/gcloud/reference/projects/add-iam-policy-binding) .
  - [Grant roles and permissions](/spanner/docs/grant-permissions#project-level_permissions) to the IAM user at the project-level for the required databases.

## Prepare your environment

You can build agent context files from any any local development environment or IDE. To prepare the environment, perform the following steps:

  - Install Gemini CLI
  - Install and setup MCP toolbox
  - Install and setup the DB Context Enrichment MCP Server

### Install Gemini CLI

To install Gemini CLI, see [Get Started with Gemini CLI](https://geminicli.com/docs/get-started) . Make sure that you install Gemini CLI in a separate directory, which is also used to install the [MCP toolbox](https://github.com/gemini-cli-extensions/mcp-toolbox) and the [DB Context Enrichment MCP Server](https://github.com/GoogleCloudPlatform/db-context-enrichment/tree/main/mcp) .

### Install and set up MCP toolbox

1.  In the same directory where you installed Gemini CLI, install the MCP Toolbox Gemini CLI extension:
    
    ``` text
    gemini extensions install https://github.com/gemini-cli-extensions/mcp-toolbox
    ```

2.  Create a `  tools.yaml  ` configuration file in the same directory where you installed the MCP toolbox for configuring the database connection:
    
    ``` text
      sources:
        my-spanner-source:
          kind: spanner
          project: PROJECT_ID
          instance: INSTANCE_ID
          database: DATABASE_ID
    ```
    
    Replace the following:
    
      - `  PROJECT_ID  ` : Your Google Cloud project ID.
      - `  INSTANCE_ID  ` : The ID of your Spanner instance.
      - `  DATABASE_ID  ` : The name of the database to connect to.

3.  Verify that the `  tools.yaml  ` file is configured correctly:
    
    ``` text
    ./toolbox --tools-file "tools.yaml"
    ```

### Install the DB Context Enrichment MCP Server

The DB Context Enrichment MCP Server provides a guided, interactive workflow to generate structured NL2SQL templates from your database schemas. It relies on the MCP Toolbox extension for database connectivity. For more information about installing the DB Context Enrichment MCP Server, see [DB Context Enrichment MCP Server](https://github.com/GoogleCloudPlatform/db-context-enrichment/tree/main/mcp) .

To install the DB Context Enrichment MCP Server, do the following:

1.  In the same directory where you installed Gemini CLI, install `  uv  ` Python package installer using `  pip  ` .
    
    ``` text
    pip install uv
    ```
    
    If `  pip  ` is not installed, install it first.

2.  Install the DB Context Enrichment MCP Server.
    
    ``` text
    gemini extensions install https://github.com/GoogleCloudPlatform/db-context-enrichment
    ```

The server uses Gemini API for generation. Make sure that you export your API key as an environment variable. For more information about how to find your API key, see [Using Gemini API keys](https://ai.google.dev/gemini-api/docs/api-key) .

Export the Gemini API key:

``` text
export GEMINI_API_KEY="YOUR_API_KEY"
```

Replace YOUR\_API\_KEY with your Gemini API key.

## Generate targeted templates

If you want to add a specific query pair as a query template to the agent context, then you can use the `  /generate_targeted_templates  ` command. For more information about templates, see [Data agents overview](/spanner/docs/data-agent-overview) .

To add a query template to the agent context, perform the following steps:

1.  In the same directory where you installed the Gemini CLI, start Gemini:
    
    ``` text
    gemini
    ```

2.  Complete the [Gemini CLI Authentication Setup](https://geminicli.com/docs/get-started/authentication/) .

3.  Verify that the MCP toolbox and the database enrichment extension are ready to use:
    
    ``` text
    /mcp list
    ```

4.  Run the `  /generate_targeted_templates  ` command:
    
    ``` text
    /generate_targeted_templates
    ```

5.  Enter the natural language query that you want to add to the query template.

6.  Enter the corresponding SQL query to the query template.

7.  Review the generated query template. You can either save the query template as an agent context file or append it to an existing context file.

The agent context file similar to `  my-cluster-psc-primary_postgres_templates_20251104111122.json  ` is saved in the directory where you ran the commands.

For more information about the context file and the query template, see [Agent context](/spanner/docs/data-agent-overview#agent-context) .

## Generate targeted facets

If you want to add a specific query pair as a facet to the agent context file, then you can use the `  /generate_targeted_facets  ` command. For more information about facets, see [Data agents overview](/spanner/docs/data-agent-overview) .

To add a facet to the agent context, perform the following steps:

1.  Run the `  /generate_targeted_facets  ` command:
    
    ``` text
    /generate_targeted_facets
    ```

2.  Enter the natural language query that you want to add to the query template.

3.  Enter the corresponding SQL query to the query template.

4.  Review the generated facet. You can either save the facet to an agent context file or append it to an existing context file.

The agent context file similar to `  my-cluster-psc-primary_postgres_templates_20251104111122.json  ` is saved in the directory where you ran the commands.

For more information about the context file and facets, see [Agent context](/spanner/docs/data-agent-overview#agent-context) .

## Optional: Generate bulk templates

If you want to auto-generate the agent context file based on your database schema and data, then you can use the `  /generate_bulk_templates  ` command.

To auto-generate bulk templates, perform the following steps:

1.  Run the `  /generate_bulk_templates  ` command:
    
    ``` text
    /generate_bulk_templates
    ```

2.  Based on your database schema, the template-based SQL generation takes you through a series of questions related to verifying the database information and granting permissions to access the database schema.

3.  Review the generated query template. You can either approve the template or update a query pair that you want to revise.

4.  Enter the natural language query that you want to add to the query template.

5.  Enter the corresponding SQL query to the query template.

6.  Review the generated query template. You can either save the query template as an agent context file or append it to an existing context file.

7.  After you approve the query template, you can either create a new template file or append the query pairs to an existing template file. The query template is saved as a JSON file in your local directory.

The agent context file similar to `  my-cluster-psc-primary_postgres_templates_20251104111122.json  ` is saved in the directory where you ran the commands.

For more information about the agent context file, see [Agent context](/spanner/docs/data-agent-overview#agent-context) .

## What's next

  - Learn more about [data agents](/spanner/docs/data-agent-overview) .
  - Learn how to [create or delete a data agent in Spanner Studio](/spanner/docs/manage-data-agents) .
  - Learn how to [inspect and call a data agent](/spanner/docs/inspect-data-agent) .
