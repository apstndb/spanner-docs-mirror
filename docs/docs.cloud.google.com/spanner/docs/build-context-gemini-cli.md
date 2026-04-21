> **Preview**
> 
> This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](https://docs.cloud.google.com/terms/service-terms#1) , and the [Additional Terms for Generative AI Preview Products](https://cloud.google.com/trustedtester/aitos) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

> **PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

This document describes how to create and optimize the context that lets you improve the accuracy of [QueryData](https://docs.cloud.google.com/spanner/docs/data-agent-overview) for building your data agent applications. Using the DB context enrichment extension in the Gemini CLI, this provides access to a suite of developer tools that automate the creation and optimization of context sets.

To learn about context sets, see [Context sets overview](https://docs.cloud.google.com/spanner/docs/context-sets-overview) .

The extension automates the creation and optimization of the context sets in the following sequence:

1.  Understand applications: Ingest artifacts such as database schemas, application code, and business requirements to establish the foundational business logic for your data agent.
2.  Create datasets: Curate an evaluation dataset containing representative natural language questions and their expected SQL answers. Establishing this baseline dataset is crucial for measuring performance and tracking improvements over time.
3.  Generate initial context: Automatically generate a baseline context set derived directly from your database schema and optional application artifacts as a quick start.
4.  Optimize context iteratively: Evaluate your dataset to identify why specific queries fail. Gemini uses automated reasoning to suggest targeted context updates, iteratively achieving higher accuracy.

While the extension offers a robust automated workflow, it is adaptable to your needs. You can bypass automation to author and insert context at a more granular level. Using specialized generation commands, you control the creation of high-quality templates, facets, and value search queries.

## Before you begin

Complete the following prerequisites before creating an agent.

### Enable required services

Enable the following services for your project:

  - [Data Analytics API with Gemini](https://console.cloud.google.com/apis/library/geminidataanalytics.googleapis.com)
  - [Gemini for Google Cloud API](https://console.cloud.google.com/apis/library/cloudaicompanion.googleapis.com)
  - [Knowledge Catalog API](https://console.cloud.google.com/apis/library/dataplex.googleapis.com)

### Prepare a Spanner instance

  - Make sure that a Spanner instance is available. For more information, see [Create an instance](https://docs.cloud.google.com/spanner/docs/create-manage-instances) .
  - Ensure that you create a database in your instance where you will create the tables. For more information, see [Create a database on the Spanner instance](https://docs.cloud.google.com/spanner/docs/create-manage-databases#create-database)

  
This tutorial requires you to have a database in your Spanner instance. For more information, see [Create a database](https://docs.cloud.google.com/spanner/docs/create-manage-databases#create-database) .

### Required roles and permissions

  - Add an IAM user or service account to the cluster. For more information, see [Apply IAM roles](https://docs.cloud.google.com/spanner/docs/grant-permissions) .
  - Grant the `spanner.databaseReader` and `geminidataanalytics.queryDataUser` roles to the IAM user at the project level. For more information, see [Add IAM policy binding for a project](https://docs.cloud.google.com/sdk/gcloud/reference/projects/add-iam-policy-binding) .
  - [Grant roles and permissions](https://docs.cloud.google.com/spanner/docs/grant-permissions#project-level_permissions) to the IAM user at the project-level for the required databases.

## Prepare your environment

You can build context set files from any local development environment or IDE. To prepare your environment, complete the following steps:

  - Install Gemini CLI
  - Install the DB Context Enrichment extension
  - Set up the database connection

### Install Gemini CLI

To install Gemini CLI, see [Get Started with Gemini CLI](https://geminicli.com/docs/get-started) .

### Install the DB Context Enrichment extension

The DB context enrichment extension provides a guided, interactive workflow to generate structured context sets and iterate on them.

For more information about installing the DB Context Enrichment extension, see [DB Context Enrichment extension](https://github.com/GoogleCloudPlatform/db-context-enrichment/tree/main/mcp) .

To install the DB Context Enrichment extension, complete these steps:

1.  Install the DB Context Enrichment Gemini CLI extension:
    
        gemini extensions install https://github.com/GoogleCloudPlatform/db-context-enrichment
    
    > **Note:** The extension requires a Gemini API key at installation to authenticate with the Gemini API and enable context generation. For more information about how to find your API key, see [Using Gemini API keys](https://ai.google.dev/gemini-api/docs/api-key) .

2.  (Optional) Update the DB Context Enrichment extension.
    
    To verify the installed version of the extension, run the following command:
    
        gemini extensions list
    
    Ensure the version is `0.5.0` or higher. To update the DB Context Enrichment extension, run the following command:
    
    ``` 
      gemini extensions update mcp-db-context-enrichment
    ```
    
    To update the DB Context Enrichment extension or replace the `GEMINI_API_KEY` , run the following command:
    
        gemini extensions config mcp-db-context-enrichment GEMINI_API_KEY
    
    Replace GEMINI\_API\_KEY with your Gemini API key.

### Set up the database connection

The extension requires a database connection to fetch schemas and ability to validate the syntax of generated SQL context. To enable the extension to interact with your database, configure authentication credentials and define your database connection configuration.

#### Configure Application Default Credentials

Configure [Application Default Credentials (ADC)](https://docs.cloud.google.com/authentication/set-up-adc-local-dev-environment) to provide user credentials for two main components:

  - Toolbox MCP server: Uses credentials to connect to your database, fetch schemas, and run SQL for validation.
  - DB Context Enrichment extension: Uses credentials to authenticate and call the Gemini API.

Run the following commands in your terminal to authenticate:

    gcloud auth application-default login

#### Configure the database connection file

The extension requires a database connection for context generation, which the [MCP Toolbox](https://mcp-toolbox.dev/documentation/introduction/) supports and defines within a configuration file.

The configuration file specifies your database source and tools required to either fetch schemas or execute SQL. The DB context enrichment extension comes with pre-installed Agent Skills to help you generate the configuration.

> **Note:** If this connection is not established, the extension returns error messages, such as "Error Discovering tools from mcp\_toolbox", and context generation doesn't work.

1.  Start the Gemini CLI:
    
        gemini

2.  Verify the skills are active by typing the following in the Gemini CLI:
    
        /skills

3.  Type a prompt, for example, `help me set up the database connection` . The skill guides you through creating the configuration file in your current working directory as `autoctx/tools.yaml` .

4.  Run the following command in the Gemini CLI to apply the `tools.yaml` configuration to the Toolbox MCP server.
    
        /mcp reload

For more information about manually configuring the database configuration file, see [MCP Toolbox Configuration](https://mcp-toolbox.dev/documentation/configuration/) .

## Generate the context with automated workflow

Improving accuracy through context engineering is usually a manual process of trial and error. Developers often guess why a query failed, write a fix, and test it manually. The DB Context Enrichment extension in the Gemini CLI automates this improvement process. It uses evaluation datasets—sets of questions with their correct SQL answers—to measure performance and identify why certain queries fail. Gemini then automatically suggests specific context updates to achieve higher accuracy. Complete these steps to systematically improve the accuracy of your data agent.

### Initialize a workspace

The initialization command sets up your local workspace, including the database connection configuration and experiment directory. This dedicated workspace ensures that all configurations, experiments, and generated files are organized in one place, making it easier to manage and track your context optimization efforts.

1.  Create a new directory to serve as your workspace for the iterative optimization flow, and navigate to it.

2.  Start the Gemini CLI in the new directory:
    
        gemini

3.  Run the initialization command:
    
        /autoctx:init
    
    The agent guides you through creating the `tools.yaml` file if no database connection has been set up, and also initializes the local `state.md` file and an `experiments` directory.
    
    After the initialization, your workspace should look like the following:
    
        my-workspace/
        └── autoctx/
            ├── tools.yaml          # Database connection and tools configuration
            ├── state.md            # Local file to track the experiment progress
            └── experiments/        # Dedicated directory for future experiment-specific files

### Prepare and expand datasets

To enable Gemini to systematically perform optimizations on your context set, prepare an evaluation dataset of representative natural language questions and their expected SQL answers ("goldens") to evaluate your context set. A high-quality evaluation dataset is crucial for measuring performance, identifying query failures, and tracking improvements over time. The dataset should be a JSON file containing the Natural Language Question (NLQ) and the golden SQL which covers the targeted use cases in your data application.

Here is an example of the expected format:

    [
      {
        "id": "example_001",
        "nlq": "What is the total revenue for the top 5 products?",
        "golden_sql": "SELECT product_id, sum(net_revenue) FROM sales GROUP BY product_id ORDER BY sum(net_revenue) DESC LIMIT 5;"
      }
    ]

The Gemini CLI extension includes a provided command that creates and scales a small baseline of questions for evaluation purposes.

1.  Navigate to your workspace folder.

2.  Start the Gemini CLI in the new directory:
    
        gemini

3.  Run the `/autoctx:generate-dataset` command in the Gemini CLI:
    
        /autoctx:generate-dataset

4.  When you're prompted by the agent, provide a seed, which is an initial example or small set of examples that guides the generation of a larger dataset. A seed can be one of the following:
    
      - A small golden dataset file
      - Specific natural language-to-SQL (NL2SQL) golden pairs
    
    For example, you could provide the following NL2SQL golden pair as a seed:
    
        Question: "What are the names of all airports in California?"
        SQL: "SELECT name FROM airports WHERE state = 'CA';"

5.  The agent prompts for permission to verify syntax and execution validity using the `execute_sql` tool. This step is optional.

6.  The agent asks whether to expand the dataset with variations from seed data (applying different filters, synonyms, and so on). This step is optional.
    
    The agent uses the `execute_sql` tool to run the newly generated SQL queries against the database to verify syntax and execution validity before presenting them to you.

7.  Selectively accept, edit, or reject the suggestions. Approved pairs are automatically saved locally and ready for evaluation.
    
        my-workspace/
        └── autoctx/
            ├── tools.yaml
            ├── state.md
            ├── golden.json  # Generated dataset
            └── experiments/

### Create initial context set

> **Note:** Skip this section if you already have a context set ready for optimization.

Generating an initial context set provides a baseline for evaluation and iterative improvement. This step uses your database schema and application artifacts to create a foundational context that reflects your business logic.

The Gemini CLI extension includes a prebuilt command to generate an initial set of templates and facets based on the database schema and information about your data agent application, for example, your application code or files with information about your business requirements. To generate a baseline context set from scratch:

1.  Navigate to your workspace folder.

2.  Start the Gemini CLI in the new directory:
    
        gemini

3.  Run the `/autoctx:bootstrap` command in the Gemini CLI:
    
        /autoctx:bootstrap
    
    You can generally expect the following from the agent.
    
      - The agent prompts you to specify an experiment name. An experiment is a dedicated workspace folder that encapsulates the complete lifecycle of a database context configuration, tracking its baseline state, evaluation test results, and subsequent iterative hill-climbing improvements. This name is used to organize all generated files under the experiment folder in your workspace; choose a name that is descriptive and memorable.
    
      - The agent fetches and lists schemas from your target database, and prompts you to optionally provide additional resources or files. If the schema is complex, the agent also prompts you to select specific schemas or tables for the initial context set. If you don't specify any, it assumes all tables available in the current database schemas.

4.  Review and optionally refine the generated context set. Once refined, the agent produces a JSON context file directly on your local disk under your workspace folder:
    
        my-workspace/
        └── autoctx/
            ├── tools.yaml
            ├── state.md
            └── experiments/
                └── my-experiment/
                    └── bootstrap_context.json  # The generated initial context set file

5.  Follow the instructions to [upload the context from Spanner Studio](https://docs.cloud.google.com/spanner/docs/manage-data-agents#edit-context-set) .

### Evaluate context effectiveness

The Gemini CLI extension includes a built-in command to evaluate your data agent using a golden dataset. The extension integrates with [Evalbench](https://github.com/GoogleCloudPlatform/evalbench) to perform evaluations by querying the agent's QueryData API with the questions specified in the golden set, and then comparing the generated SQL and its execution results with the golden SQL. Evaluation is key to understanding the effectiveness of your current context set. By comparing the generated SQL against the golden dataset, you can pinpoint specific queries that are failing and identify areas where context improvement is needed.

To measure your current context effectiveness against your golden dataset:

1.  Upload the context from Spanner Studio to the target context sets for evaluation. This step is optional if the context to be evaluated is not uploaded.

2.  Navigate to your workspace folder.

3.  Start the Gemini CLI in the folder:
    
        gemini

4.  Run the `/autoctx:evaluate` command in the Gemini CLI:
    
        /autoctx:evaluate

5.  Provide the paths for your golden dataset, your context set ID for evaluation configuration generation and evaluation run, and a designated output directory.
    
    > **Note:** For more information about how to find the context set ID, see [Find the agent context ID](https://docs.cloud.google.com/spanner/docs/inspect-data-agent#find-context-set-id) .
    
    Once complete, the agent generates the evaluation results as files in your experiment folder and summarizes the evaluation result.
    
    Optionally, you can manually inspect the evaluation from the detailed evaluation report, which is stored as CSV files in your experiment folder.
    
        my-workspace/
        └── autoctx/
            ├── tools.yaml
            ├── state.md
            ├── golden.json
            └── experiments/
                └── my-experiment/
                    └── bootstrap_context.json
                    └── eval_configs/
                        └── <configs_for_eval_run>/
                    └── eval_reports/
                        └── <eval_id>/
                            └── eval_report/
                                ├── configs.csv
                                ├── evals.csv
                                ├── scores.csv
                                └── summary.csv

### Perform gap analysis and context optimization

As a critical step in optimizing the context set, the Gemini CLI extension includes a built-in command to perform gap analysis on your existing context set and propose changes to improve its quality. Gap analysis is critical to understand why specific queries are failing and where context can be improved. Based on this analysis, Gemini uses automated reasoning to suggest targeted context updates—such as new templates or facets—to address these failures and iteratively improve query accuracy.

1.  Navigate to your workspace folder.

2.  Start the Gemini CLI in the folder:
    
        gemini

3.  Run the `/autoctx:hillclimb` command in the Gemini CLI:
    
        /autoctx:hillclimb
    
    The agent automatically identifies the most suitable evaluation results and base context for hill-climbing and asks for confirmation if there are multiple options.
    
    If no evaluation result is available, the agent prompts you for an evaluation run with the dataset and context set.
    
    Once ready, the agent reads the evaluation results and the existing context set, then generates a gap analysis report.
    
        my-workspace/
        └── autoctx/
            ├── tools.yaml
            ├── state.md
            ├── golden.json
            └── experiments/
                └── my-experiment/
                    └── bootstrap_context.json
                    └── eval_configs/
                    └── eval_reports/
                    └── hillclimb/
                        └── gap_analysis_v1.md
    
    The agent formulates fixes by proposing new prescriptive templates and facets, optionally testing SQL against the DB through `execute_sql` .
    
    Once ready, a new, improved context JSON file is generated locally, leaving the baseline context JSON file intact.
    
        my-workspace/
        └── autoctx/
            ├── tools.yaml
            ├── state.md
            ├── golden.json
            └── experiments/
                └── my-experiment/
                    └── bootstrap_context.json
                    └── eval_configs/
                    └── eval_reports/
                    └── hillclimb/
                        ├── gap_analysis_v1.md
                        └── improved_context_v1.md

4.  Follow the instructions to [upload the context to the target context set from Spanner Studio](https://docs.cloud.google.com/spanner/docs/manage-data-agents#edit-context-set) , ready for the next round of iteration starting with evaluation.

### Limitations

The automated workflow supports generating and optimizing templates and facets only. If you want to configure value search for your data agent, see [Generate value search queries](https://docs.cloud.google.com/spanner/docs/build-context-gemini-cli#generate-value-search-queries) .

## Generate targeted context

If you prefer a more customized approach to context creation, you can use the DB Context Enrichment extension to manually generate specific context elements. The following commands guide you through authoring context as a JSON file, giving you fine-grained control over template, facet, and value search query generation.

> **Note:** the Gemini CLI can access your local files to reduce overhead, for example, by specifying exact locations of files in your local directories. For example, if a step in the Gemini CLI workflow asks for information from your `tools.yaml` file, you can ask the Gemini CLI to `use tools.yaml` or respond with a prompt, for example, `look it up` .

### Generate targeted templates

To add a specific query-SQL pair as a query template to the context set, use the `/generate_targeted_templates` command.

For more information about the context set file and the query template, see [Context sets overview](https://docs.cloud.google.com/spanner/docs/context-sets-overview#context-sets) .

To add a query template to the context set, complete the following steps:

1.  Run the `/generate_targeted_templates` command in the Gemini CLI:
    
        /generate_targeted_templates

2.  Enter the natural language query to add to the query template.

3.  Enter the corresponding SQL query to the query template.

4.  Review the generated query template. You can save the query template as a context set file or append it to an existing context set file.

The context set file, for example, `my-cluster-psc-primary_postgres_context_set_20251104111122.json` , is saved in the directory where you ran the commands.

### Generate targeted facets

To add a specific query to SQL condition as a facet to the context set file, use the `/generate_targeted_facets` command.

For more information about the context set file and facets, see [Context sets overview](https://docs.cloud.google.com/spanner/docs/context-sets-overview#context-sets)

To add a facet to the context set file, complete the following steps:

1.  Run the `/generate_targeted_facets` command in the Gemini CLI:
    
        /generate_targeted_facets

2.  Enter the natural language intent to add to the facet.

3.  Enter the corresponding SQL snippet to the facet.

4.  Review the generated facet. You can save the facet to a context set file or append it to an existing context set file.

The context set file, for example, `my-cluster-psc-primary_postgres_context_set_20251104111122.json` , is saved in the directory where you ran the commands.

### Generate value search queries

To generate value searches that specify how the system searches for and matches specific values within a concept type, use the `/generate_targeted_value_searches` command.

For more information about the value index, see [Context sets overview](https://docs.cloud.google.com/spanner/docs/context-sets-overview#context-sets)

To generate a value index, complete the following steps:

1.  Run the `/generate_targeted_value_searches` command:
    
        /generate_targeted_value_searches

2.  Enter `spanner` to select Spanner as the database engine.

3.  Enter the value search configuration as follows:
    
        Table name: TABLE_NAME
        Column name: COLUMN_NAME
        Concept type: CONCEPT_TYPE
        Match function: MATCH_FUNCTION
        Description: DESCRIPTION
    
    Replace the following:
    
      - `  TABLE_NAME  ` : The table where the column associated with the concept type exists.
    
      - `  COLUMN_NAME  ` : The column name associated with the concept type.
    
      - `  CONCEPT_TYPE  ` : The concept type to define—for example, `City name` .
    
      - `  MATCH_FUNCTION  ` : The match function to use for value search. You can use one of the following functions:
        
          - `EXACT_STRING_MATCH` : For exact match of two string values. Best for unique IDs, codes, and primary keys.
          - `TRIGRAM_STRING_MATCH` : For fuzzy-matching that calculates normalized trigram distance. Best for user searches and name correction.
    
      - `  DESCRIPTION  ` : (Optional) The description of the value search query.

4.  Add additional value searches as needed. If you skip adding additional value indexes, the template-based SQL generation moves to the next step.

5.  Review the generated value searches. You can save the context set as a context set file or append it to an existing context set file.

The context set file, for example, `my-cluster-psc-primary_postgres_context_set_20251104111122.json` , is saved in the directory where you ran the commands.

## What's next

  - Learn more about [context sets](https://docs.cloud.google.com/spanner/docs/context-sets-overview) .
  - Learn how to [create or delete a context set in Spanner Studio](https://docs.cloud.google.com/spanner/docs/manage-data-agents)
  - Learn how to [test a context set](https://docs.cloud.google.com/spanner/docs/inspect-data-agent)
