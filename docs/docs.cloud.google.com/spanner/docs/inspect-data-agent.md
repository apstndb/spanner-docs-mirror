**Preview — Data agents**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) , and the [Additional Terms for Generative AI Preview Products](https://cloud.google.com/trustedtester/aitos) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

For information about access to this release, see the [access request page](https://forms.gle/pJByTWfenZAWbaXo7) .

**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

This document describes how to inspect an agent and update the agent's context file. You can inspect an agent to test its ability to generate SQL queries from natural language questions. If a generated query is not accurate, you can update the agent's context.

To learn about data agents, see [Data agents overview](/spanner/docs/data-agent-overview) .

## Before you begin

Make sure that a data agent is already created and agent context is uploaded to the data agent. For more information, see [Manage data agents in Spanner Studio](/spanner/docs/manage-data-agents) .

## Inspect a data agent

To inspect a data agent, perform the following steps:

1.  In the Google Cloud console, go to the Spanner page.

2.  Select an instance from the list, and then select a database.

3.  In the navigation menu, click **Spanner Studio** .

4.  In the **Explorer pane** , click **View actions** next to the data agent you're using.

5.  Click **Inspect agent** .

6.  In the query editor, click **Generate SQL** to open the **Help me code** panel.

7.  Enter a natural language question in the editor to generate a SQL query, and click **Generate** .

8.  Review the SQL query for accuracy.

## Download and update context for a data agent

If you are not satisfied with the generated SQL query for a natural language question, download the existing agent context file. You can then review and update the query template, and reupload the updated context file to the agent.

To download and update context for a data agent, perform the following steps:

1.  In the **Explorer pane** , click **View actions** .
2.  Click **Download agent context file** .
3.  Follow steps in [Build contexts using Gemini CLI](/spanner/docs/build-context-gemini-cli) to update context with additional query pairs.
4.  In the **Explorer pane** , click **View actions** next to the data agent you're using.
5.  Click **Edit agent** .
6.  Click **Browse** in the **Upload agent context file** section, and select the updated agent context file.
7.  Click **Save** to update the agent context.

After you are satisfied with the accuracy of your responses, you can use the `  QueryData  ` endpoint to connect your application to the data agent.

**Note:** After you upload the updated context file, it overwrites the existing context.

## Find the agent context ID

To connect a data application to the data agent, you need the agent's context ID.

1.  In the Google Cloud console, go to the Spanner page.

2.  Select an instance from the list, and then select a database.

3.  In the navigation menu, click **Spanner Studio** .

4.  In the **Explorer pane** , click **View actions** next to the data agent you're using.

5.  Click **Edit agent** .

6.  Note the context ID in **Agent context ID** . The agent context ID format is similar to `  projects/cloud-db-nl2sql/locations/us-east1/contextSets/bdf_pg_all_templates  ` .

## Connect the data agent to application

Set the agent context ID in the `  QueryData  ` method call to provide authored context for database data sources such as AlloyDB, Spanner, Cloud SQL, and Cloud SQL for PostgreSQL. For more information, see [Define data agent context for database data sources](/gemini/docs/conversational-analytics-api/data-agent-authored-context-databases)

After inspecting the data agent, you can reference the database data source in your `  QueryData  ` call.

### Example `     QueryData    ` request with authored context

The following example shows a `  QueryData  ` request using `  spanner_reference  ` database data source. The `  agent_context_reference.context_set_id  ` field is used to link to pre-authored context stored in the database.

``` text
{
  "parent": "projects/cloud-db-nl2sql/locations/us-central1",
  "prompt": "How many accounts in the Prague region are eligible for loans? A3 contains the data of region.",
  "context": {
    "datasource_references": [
      {
        "spanner_reference" {
          "database_reference" {
            "engine": "GOOGLE_SQL"
            "project_id": "cloud-db-nl2sql"
            "region": "us-central1"
            "instance_id": "evalbench"
            "database_id": "financial"
          },
          "agent_context_reference": {
            "context_set_id": "projects/cloud-db-nl2sql/locations/us-east1/contextSets/bdf_pg_all_templates"
          }
        }
      }
    ]
  },
  "generation_options": {
    "generate_query_result": true,
    "generate_natural_language_answer": true,
    "generate_disambiguation_question": true,
    "generate_explanation": true
  }
}
```

The request body contains the following fields:

  - `  prompt  ` : The natural language question from the end user.
  - `  context  ` : Contains information about the data sources.
      - `  datasource_references  ` : Specifies the data source type.
          - `  spanner_reference  ` : Required when querying the database. This field changes based on the database you are querying.
              - `  database_reference  ` : Specifies information related to your database instance.
                  - `  engine  ` : The SQL dialect of the database. Set to `  GOOGLE_SQL  ` for Spanner databases.
                  - `  project_id  ` : The project ID of the database instance.
                  - `  region  ` : The region of the Spanner instance.
                  - `  instance_id  ` : The instance ID of the Spanner instance.
                  - `  database_id  ` : The ID of the database.
              - `  agent_context_reference  ` : Links to authored context in the database.
                  - `  context_set_id  ` : The complete agent context ID of the context stored in the database. For example, `  projects/cloud-db-nl2sql/locations/us-east1/contextSets/bdf_gsql_gemini_all_templates  ` .
  - `  generationOptions  ` : Configures the type of output to generate.
      - `  generate_query_result  ` : Set to true to generate and return the query results.
      - `  generate_natural_language_answer  ` : Optional. If set to true, generates a natural language answer.
      - `  generate_explanation  ` : Optional. If set to true, generates an explanation of the SQL query.
      - `  generate_disambiguation_question  ` : Optional. If set to true, generates disambiguation questions if the query is ambiguous.

### Example `     QueryData    ` response

Here is an example of a successful response from a `  QueryData  ` call:

``` text
{
  "generated_query": "-- Count the number of accounts in Prague that are eligible for loans\nSELECT\n  COUNT(DISTINCT \"loans\".\"account_id\")\nFROM \"loans\"\nJOIN \"district\" -- Join based on district ID\n  ON \"loans\".\"district_id\" = \"district\".\"district_id\"\nWHERE\n  \"district\".\"A3\" = 'Prague'; -- Filter for the Prague region",
  "intent_explanation": "The question asks for the number of accounts eligible for loans in the Prague region. I need to join the `district` table with the `loans` table to filter by region and count the distinct accounts. The `A3` column in the `district` table contains the region information, and I'll filter for 'Prague'. The `loans` table contains information about loans, including the `account_id` and `district_id`. I will join these two tables on their respective district IDs.",
  "query_result": {
    "columns": [
      {
        "name": "count"
      }
    ],
    "rows": [
      {
        "values": [
          {
            "value": "2"
          }
        ]
      }
    ],
    "total_row_count": 1
  },
  "natural_language_answer": "There are 2 accounts in Prague that are eligible for loans."
}
```

## What's next

  - Learn more about [data agents](/spanner/docs/data-agent-overview) .
  - Learn how to [build contexts using Gemini CLI](/spanner/docs/build-context-gemini-cli)
  - Learn how to [Manage data agents in Spanner Studio](/spanner/docs/manage-data-agents) .
