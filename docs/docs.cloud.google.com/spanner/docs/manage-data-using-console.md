This page explains how to explore and manage your Spanner data using Spanner Studio in the Google Cloud console.

Spanner Studio includes an **Explorer** pane that integrates with a query editor and a SQL query results table. You can run DDL, DML, and SQL statements from this one interface. For example, instead of configuring a third-party database query tool, you can create a table and immediately query your data using the query editor.

If you're new to Spanner, learn how to [create and query a database by using the Google Cloud console](/spanner/docs/create-query-database-console) .

## Explore your data

You can use the **Explorer** to view, search, and interact with your database objects. You can create, alter, and delete the following database objects:

  - Tables
  - Columns
  - [Indexes and keys](/spanner/docs/secondary-indexes)
  - [Change streams](/spanner/docs/change-streams)
  - ML models hosted on [Vertex AI](/spanner/docs/ml)
  - [Roles](/spanner/docs/fgac-about)
  - [Views](/spanner/docs/views)
  - [Placements (in preview)](/spanner/docs/geo-partitioning)
  - [Graphs](/spanner/docs/graph/overview)
      - [Nodes](/spanner/docs/graph/schema-overview#property-graph-data-model)
      - [Edges](/spanner/docs/graph/schema-overview#property-graph-data-model)
  - [Saved queries (in preview)](/spanner/docs/saved-queries)

**Note:** System-level views in the `  INFORMATION_SCHEMA  ` and `  SPANNER_SYS  ` schemas are read-only.

You can use the search field in the **Explorer** pane to search for database objects by name in your projects. The search finds resources that directly match, or contain matches, to your search query. If your search term matches too many resources in a database object level, the result might not show all matches. Narrow your search term, and search again.

To access the **Explorer** , follow these steps:

1.  In the Google Cloud console, open the **Spanner** page.

2.  Select an instance from the list.

3.  Select a database.

4.  In the navigation menu, click **Spanner Studio** . The **Explorer** pane displays a list of the objects in your database.

## Create, modify, and query your data

Using the query editor, you can run any combination of DDL, DML, and SQL statements. Statements must be separated by a semicolon. You can compose a query yourself, or you can populate the query editor with a template.

Statements are executed based on the order in which you enter them in the query editor. Consecutive DDL statements are batched and sent for execution all at once. For more information, see [Order of execution of statements in batches](/spanner/docs/schema-updates#order_of_execution_of_statements_in_batches) .

To structure, modify, or query your data, follow these steps:

1.  In the Google Cloud console, open the **Spanner** page.

2.  Select an instance from the list.

3.  Select a database.

4.  In the navigation menu, click **Spanner Studio** .

5.  Compose a query using one of the following methods:
    
      - To compose your own query, follow these steps:
        
        1.  Open a new tab by clicking add **New SQL editor tab** or add **New tab** .
        2.  When the query editor appears, write your query.
    
      - To compose a query starting with a template, follow these steps:
        
        1.  In the **Explorer** pane, next to a database object, click more\_vert **View actions** . One or more available actions appear.
        2.  Click an action. The query editor populates with a template.
        3.  Replace any placeholders in the template.

6.  Click **Run** . The results of your query appear in the **Results** table.

7.  Optionally, click download **Export** to export your query results. It provides the following options:
    
      - Download CSV
      - Download JSON
      - Export to Google Sheets
      - Copy to clipboard (CSV)
      - Copy to clipboard (TSV)
      - Copy to clipboard (JSON)

## View schema design best practice recommendations

Spanner automatically detects opportunities to apply [schema design best practices](/spanner/docs/schema-design) to your databases. If recommendations are available for your database, you can see them in **Spanner Studio** with the following steps:

1.  In the Google Cloud console, open the **Spanner** page.

2.  Select an instance from the list.

3.  Select a database.

4.  In the navigation menu, click **Spanner Studio** .

Schema design recommendations are shown under the home Home tab.
