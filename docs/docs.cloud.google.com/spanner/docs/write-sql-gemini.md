**Preview — [Gemini](/gemini/docs/overview) in Spanner**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

This document describes how you can use [Gemini Code Assist](/gemini/docs/databases/overview) to get AI-powered assistance with the following in Spanner:

  - [Generate SQL queries using natural language prompts.](#generate-sql-queries-using-natural-language-prompts)
  - [Explain SQL queries in the query editor.](#explain-sql-in-query-editor)

Learn [how and when Gemini for Google Cloud uses your data](/gemini/docs/discover/data-governance) .

This document is intended for database administrators and data engineers who are familiar with Spanner, SQL, and data analysis. If you're new to Spanner, see [Create and query a database by using the Google Cloud console](/spanner/docs/create-query-database-console) .

**Note** : Coding assistance is part of Gemini Code Assist and is available at no charge until it's included in Gemini Code Assist Standard edition. This change will be communicated at a later date. At that time, you will need to acquire a [Gemini Code Assist Standard edition license](/gemini/docs/codeassist/set-up-gemini#purchase-subscription) to continue to use coding assistance in Spanner Studio.

## Before you begin

1.  Optional: [Set up Gemini Code Assist](/gemini/docs/discover/set-up-gemini) .

2.  To complete the tasks in this document, ensure that you have the [necessary Identity and Access Management (IAM) permissions](#required-roles) .

3.  In the Google Cloud console, go to the **Spanner** page.

4.  Select an instance from the list.

5.  Select a database.

6.  In the navigation menu, click **Spanner Studio** .

7.  In the taskbar, click pen\_spark **Gemini** to view Gemini features in Spanner.

8.  Select the Gemini features that you want to enable—for example, [**Comment-to-query generation**](#generate-sql-queries-using-natural-language-prompts) . You can select and try features for yourself without affecting others working in your project.

9.  Optional: If you want to follow along with the examples in this document, first create the `  Singers  ` table as described in [Create a schema for your database](/spanner/docs/create-query-database-console#create-schema) .

To disable Gemini features in Spanner, repeat these steps, and then deselect the Gemini features that you want to disable.

### Required roles

To get the permissions that you need to complete the tasks in this document, ask your administrator to grant you the [Gemini for Google Cloud User](/iam/docs/roles-permissions/cloudaicompanion#cloudaicompanion.user) ( `  roles/cloudaicompanion.user  ` ) IAM role on the project. For more information about granting roles, see [Manage access to projects, folders, and organizations](/iam/docs/granting-changing-revoking-access) .

You might also be able to get the required permissions through [custom roles](/iam/docs/creating-custom-roles) or other [predefined roles](/iam/docs/roles-overview#predefined) .

## Generate SQL queries using natural language prompts

As an early-stage technology, Gemini for Google Cloud products can generate output that seems plausible but is factually incorrect. We recommend that you validate all output from Gemini for Google Cloud products before you use it. For more information, see [Gemini for Google Cloud and responsible AI](/gemini/docs/discover/responsible-ai) .

You can give Gemini natural language comments (or *prompts* ) to generate queries that are based on your schema. For example, you can prompt Gemini to generate SQL in response to the following prompts:

  - "Create a table that tracks customer satisfaction survey results."
  - "Add a date column called birthday to the Singers table."
  - "How many singers were born in the 90s?"

**Note:** When you enter a prompt for Gemini in Spanner, information about your database's schema is included with the prompt. This can include table and column names, data types, and column descriptions. Your database schema and data remain in Spanner and aren't sent to Gemini.

To generate SQL in Spanner with Gemini assistance, follow these steps:

1.  In the Google Cloud console, go to the **Spanner** page.

2.  Select an instance from the list.

3.  Select a database.

4.  In the navigation pane, click **Spanner Studio** . The **Explorer** pane displays a list of objects in your database.

5.  To query your database, click the add **New SQL editor tab** . Make sure that [SQL generation is enabled](#before-you-begin) .

6.  To generate SQL, type a comment in the query editor starting with `  --  ` followed by a [single-line comment](/bigquery/docs/reference/standard-sql/lexical#comments) , and then press `  Return  ` .
    
    For example, if you enter the prompt `  -- add a row to table singers  ` and press `  Return  ` , then Gemini generates SQL that's similar to the following:
    
    ``` text
    INSERT INTO Singers (SingerId, FirstName, LastName, BirthDate)
    VALUES (1, Alex, 'M.', '1977-10-16');
    ```
    
    To continue the example using the `  Singers  ` table, if you enter the prompt `  -- show all singers born in the 70s  ` , then Gemini generates SQL that's similar to the following:
    
    ``` text
    SELECT *
    FROM Singers
    WHERE Singers.BirthDate
    BETWEEN '1970-01-01' AND '1979-12-31'
    ```
    
    **Note:** Gemini might suggest different syntax each time that you enter the same prompt.

7.  Review the generated SQL and take any of the following actions:
    
      - To accept SQL generated by Gemini, press `  Tab  ` , and then click **Run** to execute the suggested SQL.
      - To edit the SQL generated by Gemini, press `  Tab  ` , edit the SQL, and then click **Run** .
      - To dismiss the suggestion, press `  Esc  ` or continue typing.

## Explain SQL statements in the query editor

You can use Gemini in Spanner to explain SQL queries in natural language. This explanation can help you understand the syntax, underlying schema, and business context for complex or long queries.

1.  In the Google Cloud console, go to the **Spanner** page.

2.  Select an instance from the list.

3.  Select a database.

4.  In the navigation pane, click **Spanner Studio** .

5.  To query your database, click the add **New tab** .

6.  In the query editor, paste the query.

7.  Highlight the query that you want Gemini to explain, and then click spark **Explain this query** .
    
    The SQL explanation appears in the **Gemini** pane.

## What's next

  - Read [Gemini for Google Cloud overview](/gemini/docs/overview) .
  - Learn [how Gemini uses your data](/gemini/docs/discover/data-governance) .
