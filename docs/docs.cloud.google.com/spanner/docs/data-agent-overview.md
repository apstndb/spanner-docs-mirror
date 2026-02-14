**Preview — Data agents**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) , and the [Additional Terms for Generative AI Preview Products](https://cloud.google.com/trustedtester/aitos) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

For information about access to this release, see the [access request page](https://forms.gle/pJByTWfenZAWbaXo7) .

**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

Data agents let you interact with the data in your database using conversational language. You can create data agents by defining context for a set of tables in your database which allows data agents to translate natural language questions into accurate queries for your target use cases.

Context is database-specific information that enables the LLM to generate queries with high accuracy. Context includes templates and facets that help the agent understand your database schema and the business logic of your applications.

The following databases are supported:

  - AlloyDB for PostgreSQL
  - Cloud SQL for MySQL
  - Cloud SQL for PostgreSQL
  - Spanner

## When to use data agents

You can use data agents to build conversational data applications ideal for use cases such as:

  - **Customer service automation** : Handle high-volume inquiries like "Where is my order?" or "What is my current balance?".
  - **E-commerce shopping assistants** : Help users navigate large product catalogs with natural language queries like "Show me running shoes under $100."
  - **Booking and reservation systems** : Enable users to check availability and book appointments, flights, or dining services through chat interfaces.
  - **Field operations tools** : Allow mobile employees to query inventory levels, part availability, or service ticket details in real-time.

## How data agents work

To build effective agentic applications, the agent must understand your data organization and business logic. You provide this information in the form of agent context.

You define agent context in files that contain JSON objects for templates and facets. You author these context files with the help of the [Gemini CLI](/gemini/docs/codeassist/gemini-cli) . You then upload the context file to a data agent that you create in the Google Cloud console. This process enables the agent to learn the specific schema of the database and business logic of the application.

The agent context file looks similar to the following:

``` text
{
  "templates": [
    {
      "nl_query": "Count prague loan accounts",
      "sql": "SELECT COUNT(T1.account_id) FROM bird_dev_financial.account AS T1 INNER JOIN bird_dev_financial.loan AS T2 ON T1.account_id = T2.account_id INNER JOIN bird_dev_financial.district AS T3 ON T1.district_id = T3.district_id WHERE T3.\"A3\" = \"Prague\"",
      "intent": "How many accounts associated with loans are located in the Prague region?",
      "manifest": "How many accounts associated with loans are located in a given city?",
      "parameterized": {
        "parameterized_intent": "How many accounts associated with loans are located in $1",
        "parameterized_sql": "SELECT COUNT(T1.account_id) FROM bird_dev_financial.account AS T1 INNER JOIN bird_dev_financial.loan AS T2 ON T1.account_id = T2.account_id INNER JOIN bird_dev_financial.district AS T3 ON T1.district_id = T3.district_id WHERE T3.\"A3\" = $1"
      }
    }
  ],
  "facets": [
    {
      "sql_snippet": "T.\"A11\" BETWEEN 6000 AND 10000",
      "intent": "Average salary between 6000 and 10000",
      "manifest": "Average salary between a given number and a given number",
      "parameterized": {
         "parameterized_intent": "Average salary between $1 and $2",
         "parameterized_sql_snippet": "T.\"A11\" BETWEEN $1 AND $2"
      }
    }
  ]
}
```

When an end user asks a natural language question, the agent prioritizes matching the question to the templates and facets that have been audited by the developer curating the context. Once the agent identifies a match, it uses the selected query template and facets to synthesize a database query. The agent then executes that query against the database to return accurate results.

The `  QueryData  ` endpoint in the [Conversational Analytics API](/gemini/docs/conversational-analytics-api/overview#key_api_operations) is an agentic tool that allows programmatic integration with your applications to enable SQL query generation from natural language questions. In a conversational application, the `  QueryData  ` endpoint must be used within the framework that manages the conversation history and context.

## Agent context

Agent context consists of a curated set of templates and facets in JSON format that guide the agent in translating natural language questions into queries for a specific database. Defining context ensures high-accuracy SQL generation for common query patterns.

Ensure that the agent context is accurate and comprehensive in its coverage of expected application queries to maximize accuracy.

Agents and agent context can be created in the `  us-central1  ` , `  us-east1  ` , `  europe-west4  ` , and `  asia-southeast1  ` regions.

### Query templates

Query templates are a curated set of representative natural language questions with corresponding SQL queries. They also include explanations to provide a declarative rationale for the natural language-to-SQL generation.

A query template object looks similar to the following:

``` text
{
  "templates": [
    {
      "nl_query": "Count prague loan accounts",
      "sql": "SELECT COUNT(T1.account_id) FROM bird_dev_financial.account AS T1 INNER JOIN bird_dev_financial.loan AS T2 ON T1.account_id = T2.account_id INNER JOIN bird_dev_financial.district AS T3 ON T1.district_id = T3.district_id WHERE T3.\"A3\" = \"Prague\"",
      "intent": "How many accounts associated with loans are located in the Prague region?",
      "manifest": "How many accounts associated with loans are located in a given city?",
      "parameterized": {
        "parameterized_intent": "How many accounts associated with loans are located in $1",
        "parameterized_sql": "SELECT COUNT(T1.account_id) FROM bird_dev_financial.account AS T1 INNER JOIN bird_dev_financial.loan AS T2 ON T1.account_id = T2.account_id INNER JOIN bird_dev_financial.district AS T3 ON T1.district_id = T3.district_id WHERE T3.\"A3\" = $1"
      }
    }
  ]
},
...
```

The main components of the query template JSON object are as follows:

  - `  nl_query  ` : An example of a natural language query that the data agent handles.
  - `  sql  ` : The SQL query for the natural language query.
  - `  intent  ` : The goal or purpose of the natural language query. If not set, this value defaults to the natural language query.
  - `  manifest  ` : A generalized, auto-generated form of the intent.
  - `  parameterized_intent  ` : A templated, auto-generated form of the intent, with entity values replaced by parameters.
  - `  parameterized_sql  ` : A templated, auto-generated form of the SQL query that corresponds to the parameterized intent.

### Query facets

Query facets are a curated set of representative natural language conditions with corresponding SQL predicates. Facets manage filtering and conditions, which enables query templates to perform faceted searches.

A query facet object looks similar to the following:

``` text
{
...
"facets": [
    {
      "sql_snippet": "T.\"A11\" BETWEEN 6000 AND 10000",
      "intent": "Average salary between 6000 and 10000",
      "manifest": "Average salary between a given number and a given number",
      "parameterized": {
         "parameterized_intent": "Average salary between $1 and $2",
         "parameterized_sql_snippet": "T.\"A11\" BETWEEN $1 AND $2"
      }
    }
  ]
}
```

The main components of the facet JSON object are as follows:

  - `  sql_snippet  ` : A SQL snippet.
  - `  intent  ` : An explanation of the SQL predicate.
  - `  manifest  ` : A generalized, auto-generated form of the intent.
  - `  parameterized_intent  ` : A templated, auto-generated form of the intent, with entity values replaced by parameters.
  - `  parameterized_sql_snippet  ` : A templated, auto-generated form of the sql\_snippet that corresponds to the parameterized intent.

## Limitations

Data agents have the following limitations:

  - Agent context for databases only supports templates and facets.
  - Agent context for databases is only used by the `  QueryData  ` endpoint in the Conversational Analytics API.

**Warning:** All agent contexts associated with a database are viewable to all users with any read access to that database. In cases where users have narrower access to the database, such as Spanner fine-grained access control, users can potentially discover other fields and tables from the agent context.

## What's next

  - Learn how to [create or delete a data agent in Spanner Studio](/spanner/docs/manage-data-agents) .
