**Preview**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](https://docs.cloud.google.com/terms/service-terms#1) , and the [Additional Terms for Generative AI Preview Products](https://cloud.google.com/trustedtester/aitos) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

This document describes how to use the Gemini CLI and the MCP toolbox to build agent context files. These files contain templates, facets, and value searches that provide context for generating SQL queries from natural language. You will also use the DB Context Enrichment extension.

To learn about context sets, see [Context sets overview](https://docs.cloud.google.com/spanner/docs/context-sets-overview) .

To build an agent context file, perform the following high-level steps:

  - Prepare your environment
  - Generate targeted templates
  - Generate targeted facets
  - Generate targeted value searches
  - Optional. Generate bulk templates

## Before you begin

Complete the following prerequisites before creating an agent.

### Enable required services

Enable the following services for your project:

  - [Data Analytics API with Gemini](https://console.cloud.google.com/apis/library/geminidataanalytics.googleapis.com)
  - [Gemini for Google Cloud API](https://console.cloud.google.com/apis/library/cloudaicompanion.googleapis.com)
  - [Dataplex Universal Catalog API](https://console.cloud.google.com/apis/library/dataplex.googleapis.com)

### Prepare a Spanner instance

  - Make sure that a Spanner instance is available. For more information, see [Create an instance](https://docs.cloud.google.com/spanner/docs/create-manage-instances) .
  - Ensure that you create a database in your instance where you will create the tables. For more information, see [Create a database on the Spanner instance](https://docs.cloud.google.com/spanner/docs/create-manage-databases#create-database)

  
This tutorial requires you to have a database in your Spanner instance. For more information, see [Create a database](https://docs.cloud.google.com/spanner/docs/create-manage-databases#create-database) .

### Required roles and permissions

  - Add an IAM user or service account to the cluster. For more information, see [Apply IAM roles](https://docs.cloud.google.com/spanner/docs/grant-permissions) .
  - Grant the `spanner.databaseReader` and `geminidataanalytics.queryDataUser` roles to the IAM user at the project level. For more information, see [Add IAM policy binding for a project](https://docs.cloud.google.com/sdk/gcloud/reference/projects/add-iam-policy-binding) .
  - [Grant roles and permissions](https://docs.cloud.google.com/spanner/docs/grant-permissions#project-level_permissions) to the IAM user at the project-level for the required databases.

## Prepare your environment

You can build context set files from any local development environment or IDE. To prepare the environment, perform the following steps:

  - Install Gemini CLI
  - Install the DB Context Enrichment extension
  - Setup database connection

### Install Gemini CLI

To install Gemini CLI, see [Get Started with Gemini CLI](https://geminicli.com/docs/get-started) .

### Install the DB Context Enrichment MCP extension

The DB Context Enrichment extension provides a guided, interactive workflow to generate structured NL2SQL context from your database schemas.

For more information about installing the DB Context Enrichment extension, see [DB Context Enrichment extension](https://github.com/GoogleCloudPlatform/db-context-enrichment/tree/main/mcp) .

To install the DB Context Enrichment extension, follow these steps:

1.  Install the MCP Toolbox Gemini CLI extension:
    
        gemini extensions install https://github.com/gemini-cli-extensions/mcp-toolbox
    
    **Note:** The extension requires a Gemini API key at installation to authenticate with the Gemini API and enable context generation. For more information about how to find your API key, see [Using Gemini API keys](https://ai.google.dev/gemini-api/docs/api-key) .

2.  (Optional) Update the DB Context Enrichment extension.
    
    To verify the installed version of the extension, run the following command:
    
        gemini extensions list
    
    Make sure that the version is `0.4.2` or higher. To update the DB Context Enrichment extension, run the following command:
    
    ``` 
      gemini extensions update mcp-db-context-enrichment
    ```
    
    To update the DB Context Enrichment extension or to replace the `GEMINI_API_KEY` , run the following command:
    
        gemini extensions config mcp-db-context-enrichment GEMINI_API_KEY
    
    Replace GEMINI\_API\_KEY with your Gemini API key.

### Set up the DB Connection

The extension requires the database connection for context generation for fetching schemas, and executing SQL statements. To enable the extension to interact with your database, you must configure authentication credentials and define your database sources and tools.

#### Configure Application Default Credentials

You must configure [Application Default Credentials (ADC)](https://docs.cloud.google.com/authentication/set-up-adc-local-dev-environment) to provide user credentials for two main components:

  - Toolbox MCP server: uses credentials to connect to your database, fetch schemas, and run SQL for validation.
  - DB Context Enrichment extension: uses credentials to authenticate and call the Gemini API.

Run the following commands in your terminal to authenticate:

    gcloud auth application-default login

#### Configure the `tools.yaml` file

The extension requires a database connection for context generation, which is supported by the [MCP Toolbox](https://mcp-toolbox.dev/documentation/introduction/) and defined within the tools.yaml configuration file.

The `tools.yaml` file specifies your database source and tools required to either fetch schemas or execute SQL. The extension comes with pre-installed Agent Skills to help you generate the configuration.

**Note:** If this connection is not established, the extension will return error messages, such as "Error Discovering tools from mcp\_toolbox" and context generation won't work.

1.  Start Gemini CLI:
    
        gemini

2.  Verify the skills are active by typing in the Gemini CLI:
    
        /skills

3.  Type a prompt such as `help me setup the database connection` . The skill guides you through creating the `tools.yaml` file in your current working directory.

4.  Run the following command in Gemini CLI to apply the `tools.yaml` configuration to the Toolbox MCP server.
    
        /mcp reload

For more information about manually configuring the `tools.yaml` file, see [MCP Toolbox Configuration](https://mcp-toolbox.dev/documentation/configuration/) .

## Generate Context

The extensions installed earlier empower Gemini CLI to help you author context in the form of a JSON file.

**Note:** Gemini CLI can access your local files to reduce overheads such as specifying exact locations of files in your local directories. For example, if a step in the Gemini CLI workflow asks you for information that you can find in your `tools.yaml` file, you can ask Gemini CLI to `use tools.yaml` or respond with a prompt such as `look it up` .

### Generate targeted templates

If you want to add a specific query pair as a query template to the context set, then you can use the `/generate_targeted_templates` command. For more information about templates, see [Context sets overview](https://docs.cloud.google.com/spanner/docs/context-sets-overview)

To add a query template to the context set, perform the following steps:

1.  In the same directory, start Gemini CLI:
    
        gemini

2.  Complete the [Gemini CLI Authentication Setup](https://geminicli.com/docs/get-started/authentication/) .

3.  Verify that the MCP toolbox and the database enrichment extension are ready to use:
    
        /mcp reload

4.  Run the `/generate_targeted_templates` command:
    
        /generate_targeted_templates

5.  Enter the natural language query that you want to add to the query template.

6.  Enter the corresponding SQL query to the query template.

7.  Review the generated query template. You can either save the query template as a context set file or append it to an existing context set file.

The context set file similar to `my-cluster-psc-primary_postgres_templates_20251104111122.json` is saved in the directory where you ran the commands.

For more information about the context set file and the query template, see [Context sets overview](https://docs.cloud.google.com/spanner/docs/context-sets-overview#context-sets) .

### Generate targeted facets

If you want to add a specific query pair as a facet to the context set file, then you can use the `/generate_targeted_facets` command.

To add a facet to the context set file, perform the following steps:

1.  Run the `/generate_targeted_facets` command:
    
        /generate_targeted_facets

2.  Enter the natural language query that you want to add to the query template.

3.  Enter the corresponding SQL query to the query template.

4.  Review the generated facet. You can either save the facet to a context set file or append it to an existing context set file.

The context set file similar to `my-cluster-psc-primary_postgres_templates_20251104111122.json` is saved in the directory where you ran the commands.

For more information about the context set file and facets, see [Context sets overview](https://docs.cloud.google.com/spanner/docs/context-sets-overview#context-sets)

### Generate value search queries

If you want to generate value searches that specify how the system should search for and match specific values within a concept type, then you can use the `/generate_targeted_value_searches` command.

To generate a value index, perform the following steps:

1.  Run the `/generate_targeted_value_searches` command:
    
        /generate_targeted_value_searches

<!-- end list -->

1.  Enter `spanner` to select Spanner as the database engine.

<!-- end list -->

1.  Enter the value search configuration as follows:
    
        Table name: TABLE_NAME
        Column name: COLUMN_NAME
        Concept type: CONCEPT_TYPE
        Match function: MATCH_FUNCTION
        Description: DESCRIPTION
    
    Replace the following:
    
      - `  TABLE_NAME  ` : The table where the column associated with the concept type exists.
    
      - `  COLUMN_NAME  ` : The column name associated with the concept type.
    
      - `  CONCEPT_TYPE  ` : The concept type that you want to define—for example, `City name` .
    
      - `  MATCH_FUNCTION  ` : The match function to use for value search. You can use one of the following functions:
        
          - `EXACT_STRING_MATCH` : For exact match of two string values. Best for unique IDs, codes, and primary keys.
          - `TRIGRAM_STRING_MATCH` : For fuzzy-matching that calculates normalized trigram distance. Best for user searches and name correction.
    
      - `  DESCRIPTION  ` : (Optional) The description of the value search query.

2.  Add additional value searches as required. If you skip adding additional value indexes, the template-based SQL generation moves to the next step.

3.  Review the generated value searches. You can either save the context set as a context set file or append it to an existing context set file.

The context set file similar to `my-cluster-psc-primary_postgres_templates_20251104111122.json` is saved in the directory where you ran the commands.

For more information about the value index, see [Context sets overview](https://docs.cloud.google.com/spanner/docs/context-sets-overview#context-sets)

### Optional: Generate bulk templates

If you want to auto-generate the context set file based on your database schema and data, then you can use the `/generate_bulk_templates` command.

To auto-generate bulk templates, perform the following steps:

1.  Run the `/generate_bulk_templates` command:
    
        /generate_bulk_templates

2.  Based on your database schema, the template-based SQL generation takes you through a series of questions related to verifying the database information and granting permissions to access the database schema.

3.  Review the generated query template. You can either approve the template or update a query pair that you want to revise.

4.  Enter the natural language query that you want to add to the query template.

5.  Enter the corresponding SQL query to the query template.

6.  Review the generated query template. You can either save the query template as a context set file or append it to an existing context file.

7.  After you approve the query template, you can either create a new template file or append the query pairs to an existing template file. The query template is saved as a JSON file in your local directory.

The context set file similar to `my-cluster-psc-primary_postgres_templates_20251104111122.json` is saved in the directory where you ran the commands.

For more information about the context set file, see [Context sets overview](https://docs.cloud.google.com/spanner/docs/context-sets-overview#context-sets) .

## What's next

  - Learn more about [context sets](https://docs.cloud.google.com/spanner/docs/context-sets-overview) .
  - Learn how to [create or delete a context set in Spanner Studio](https://docs.cloud.google.com/spanner/docs/manage-data-agents)
  - Learn how to [test a context set](https://docs.cloud.google.com/spanner/docs/inspect-data-agent)
