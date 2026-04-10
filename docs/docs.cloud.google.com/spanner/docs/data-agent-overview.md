**Preview**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](https://docs.cloud.google.com/terms/service-terms#1) , and the [Additional Terms for Generative AI Preview Products](https://cloud.google.com/trustedtester/aitos) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

Context sets enable you to interact with the data in your database using conversational language and are used by tools like [QueryData](https://docs.cloud.google.com/gemini/data-agents/reference/rest/v1beta/projects.locations/queryData) to build conversation agents. You can create context sets by defining context for a set of tables in your database. This context allows QueryData to translate natural language questions into accurate queries for your target use cases.

Context set is a collection of database-specific information that enables QueryData to generate queries with high accuracy. Context set includes templates, facets, and value searches that help QueryData understand your database schema and the business logic of your applications.

The following databases are supported:

  - AlloyDB for PostgreSQL
  - Cloud SQL for MySQL
  - Cloud SQL for PostgreSQL
  - Spanner

## When to use context sets

You can use context sets to build conversational data applications ideal for use cases such as:

  - **Customer service automation** : Handle high-volume inquiries like "Where is my order?" or "What is my current balance?".
  - **E-commerce shopping assistants** : Help users navigate large product catalogs with natural language queries like "Show me running shoes under $100."
  - **Field operations tools** : Allow mobile employees to query inventory levels, part availability, or service ticket details in real-time.

## How context sets work

To build effective agentic applications, QueryData must understand your data organization and business logic. You provide this information in the form of context set.

You define context in files that contain JSON objects for each context type. You author these context files with the help of the [Gemini CLI](https://docs.cloud.google.com/gemini/docs/codeassist/gemini-cli) . You then upload the context file to a context set that you create in the Google Cloud console. This process enables QueryData to learn the specific schema of the database and business logic of the application.

The context file looks similar to the following:

    {
      "templates": [
        {
          "nl_query": "Count Prague loan accounts",
          "sql": "SELECT COUNT(T1.account_id) FROM bird_dev_financial.account AS T1 INNER JOIN bird_dev_financial.loan AS T2 ON T1.account_id = T2.account_id INNER JOIN bird_dev_financial.district AS T3 ON T1.district_id = T3.district_id WHERE T3.\"A3\" ='Prague'",
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
      ],
      "value_searches": [
        {
          "query": "SELECT $value as value, 'accounts.account_type' as columns, 'Account Type' as concept_type, 0 as distance, '{}'::text as context FROM \"accounts\" T WHERE T.\"account_type\" = $value",
          "concept_type": "Account Type",
          "description": "Exact match for account types"
        }
       ]
    }

When an end user asks a natural language question, QueryData prioritizes matching the question to the templates and facets that have been audited by the developer curating the context. Once QueryData identifies a match, it uses the selected query template and facets to synthesize a database query. The context set logic then executes that query against the database to return accurate results.

A recommended step is to define value searches within your context. Value searches allow the agent to map value phrases to specific values stored in your database columns. This grounds the LLM in your actual data, helping it resolve ambiguities, such as whether a term refers to a `  District  ` name or a `  City  ` .

If the agent fails to find a matching template, then it uses value searches to map natural language phrases to specific values stored in your database columns. This grounds the LLM in your actual data, helping it resolve ambiguities, such as whether a term refers to a `  District  ` , `  Name  ` , or a `  City  ` .

The `  QueryData  ` endpoint in the [Conversational Analytics API](https://docs.cloud.google.com/gemini/docs/conversational-analytics-api/overview#key_api_operations) is an agentic tool that allows programmatic integration with your applications to enable SQL query generation from natural language questions. In a conversational application, the `  QueryData  ` endpoint must be used within the framework that manages the conversation history and context.

## Context set file

Context set file consists of a curated set of templates and facets in JSON format that guide QueryData in translating natural language questions into queries for a specific database. Defining context ensures high-accuracy SQL generation for common query patterns.

Ensure that the context set is accurate and comprehensive in its coverage of expected application queries to maximize accuracy.

Context sets can be created in the `  us-central1  ` , `  us-east1  ` , `  europe-west4  ` , and `  asia-southeast1  ` regions.

### Query templates

Query templates are a curated set of representative natural language questions with corresponding SQL queries. They also include explanations to provide a declarative rationale for the natural language-to-SQL generation.

A query template object looks similar to the following:

    {
      "templates": [
        {
          "nl_query": "Count prague loan accounts",
          "sql": "SELECT COUNT(T1.account_id) FROM bird_dev_financial.account AS T1 INNER JOIN bird_dev_financial.loan AS T2 ON T1.account_id = T2.account_id INNER JOIN bird_dev_financial.district AS T3 ON T1.district_id = T3.district_id WHERE T3.\"A3\" = 'Prague'",
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

The main components of the query template JSON object are as follows:

  - `  nl_query  ` : An example of a natural language query that QueryData handles.
  - `  sql  ` : The SQL query for the natural language query.
  - `  intent  ` : The goal or purpose of the natural language query. If not set, this value defaults to the natural language query.
  - `  manifest  ` : A generalized, auto-generated form of the intent.
  - `  parameterized_intent  ` : A templated, auto-generated form of the intent, with entity values replaced by parameters.
  - `  parameterized_sql  ` : A templated, auto-generated form of the SQL query that corresponds to the parameterized intent.

### Query facets

Query facets are a curated set of representative natural language conditions with corresponding SQL predicates. Facets manage filtering and conditions, which enables query templates to perform faceted searches.

A query facet object looks similar to the following:

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

The main components of the facet JSON object are as follows:

  - `  sql_snippet  ` : A SQL snippet.
  - `  intent  ` : An explanation of the SQL predicate.
  - `  manifest  ` : A generalized, auto-generated form of the intent.
  - `  parameterized_intent  ` : A templated, auto-generated form of the intent, with entity values replaced by parameters.
  - `  parameterized_sql_snippet  ` : A templated, auto-generated form of the sql\_snippet that corresponds to the parameterized intent.

### Value search queries

[Value search queries](https://docs.cloud.google.com/alloydb/docs/ai/build-context-gemini-cli#generate-value-searches) are developer-defined queries that use match functions to find values and their context within a database. Value linking uses the results of these queries to identify which tables and columns contain a matching value, understand the value's concept type, and correct misspellings.

The QueryData API uses value linking to more accurately convert natural language into SQL. By using value search queries, the API can correct misspellings and resolve value types based on database values, thus improving conversion accuracy.

Value linking improves natural language to SQL conversion accuracy. For example, if a user asks, "Are there any flights out of Heathrow?", the database might store the airport name as "London Heathrow". Without value linking, the generated SQL might filter by `  WHERE name = 'Heathrow'  ` and return no results. Value search queries guide the agent to map "Heathrow" to the correct database value "London Heathrow" and its schema location ( `  airports.name  ` ), ensuring that the generated SQL is accurate.

An example value search query is:

    {
      ...
      "value_searches": [
        {
          "query": "SELECT $value as value, 'airports.iata' as columns, 'Airport IATA Code' as concept_type, 0 as distance, '{}'::text as context FROM \"airports\" T WHERE T.\"iata\" = $value",
          "concept_type": "Airport IATA Code",
          "description": "Exact match (Standard SQL) for 3-letter airport codes"
        },
        {
          "query": "WITH TrigramMetrics AS ( SELECT T.\"name\" AS original_value, (T.\"name\" <-> $value::text) AS normalized_dist FROM \"airports\" T WHERE T.\"name\" % $value::text ) SELECT original_value AS value, 'airports.name' AS columns, 'Airport Name' AS concept_type, normalized_dist AS distance, '{}'::text AS context FROM TrigramMetrics",
          "concept_type": "Airport Name",
          "description": "Fuzzy match using standard trigram for partial airport names"
        }
      ]
    }

The main components of the value search JSON object are as follows:

  - `  query  ` : A parameterized SQL statement that defines the logic for matching a value phrase against the values stored in a column of a table in a database. The result set typically projects the matched value, the schema location, the concept type, and a normalized distance metric (between 0 and 1).
  - `  concept_type  ` : A semantic label assigned to the value—for example, the `  district  ` or the `  loan_status  ` . This label helps value linking and eventually the QueryData API to understand the role of the value phrase in the schema. It also helps produce a SQL statement that targets the concept for the value, as well as the table and column where the value phrase appears in.
  - `  description  ` : A description of the search logic.

## Limitations

Context sets have the following limitations:

  - Context sets for databases only supports templates, facets, and value searches.
  - Context sets for databases is only used by the `  QueryData  ` endpoint in the Conversational Analytics API.

**Warning:** All agent contexts associated with a database are viewable to all users with any read access to that database. In cases where users have narrower access to the database, such as Spanner fine-grained access control, users can potentially discover other fields and tables from the agent context.

## What's next

  - Learn how to [create or delete a context set in Spanner Studio](https://docs.cloud.google.com/spanner/docs/manage-data-agents) .
