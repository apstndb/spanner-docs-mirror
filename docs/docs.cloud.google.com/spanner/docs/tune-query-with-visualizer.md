The query plan visualizer lets you quickly understand the structure of the [query plan](/spanner/docs/query-execution-plans) chosen by Spanner to evaluate a query. This guide describes how you can use a query plan to help you understand the execution of your queries.

## Before you begin

**Note:** Spanner Studio (formerly labeled **Query** in the Google Cloud console) supports SQL, DML, and DDL operations in a single editor. For more information, see [Manage your data using the Google Cloud console](/spanner/docs/manage-data-using-console) .

To familiarize yourself with the parts of the Google Cloud console user interface mentioned in this guide, read the following:

### Run a query in Google Cloud console

1.  Go to the Spanner **Instances** page in Google Cloud console.

2.  Select the name of the instance containing the database you want to query.
    
    Google Cloud console displays the instance's **Overview** page.

3.  Select the name of the database you want to query.
    
    Google Cloud console displays the database's **Overview** page.

4.  In the side menu, click **Spanner Studio** .
    
    Google Cloud console displays the database's **Spanner Studio** page.

5.  Enter the SQL query in the editor pane.

6.  Click **Run** .
    
    Spanner runs the query.

7.  Click the **Explanation** tab to see the query plan visualization.

### A tour of the query editor

The Spanner Studio page provides query tabs that let you type or paste SQL query and DML statements, run them against your database, and view their results and query execution plans. The key components of the Spanner Studio page are numbered in the following screenshot.

**Figure 7.** Annotated query editor page.

1.  The *tab bar* shows the query tabs you have open. To create a new tab, click add **New tab** .
    
    You can also use Gemini Code Assist to get AI-powered assistance. For more information, see [Write SQL with Gemini assistance](/spanner/docs/write-sql-gemini) .

2.  The *editor commands bar* provides these options:
    
      - The **Run** command executes the statements entered in the editing pane, producing query results in the **Results** tab and query execution plans in the **Explanation** tab. Change the default behavior using the drop-down to produce **Results only** or **Explanation only** .
        
        Highlighting something in the editor changes the **Run** command to **Run selected** , allowing you to execute what you have selected.
    
      - The **Save** command lets you create, save, and manage SQL scripts as saved queries. For more information, see [Saved queries overview](/spanner/docs/saved-queries) .
    
      - The **Format** command formats statements in the editor so that they are easier to read.
    
      - The **Clear** command deletes all text in the editor and clears the **Results** and **Explanation** subtabs.
    
      - The **Documentation** link opens a browser tab to Spanner documentation about SQL query syntax.
    
    Queries are validated automatically any time they are updated in the editor. If the statements are valid, the editor commands bar displays a confirmation check mark and the message **Valid** . If there are any issues, it displays an error message with details.

3.  The *editor* is where you enter SQL query and DML statements. Inputs are color-coded and line numbers are automatically added for multi-line statements.
    
    If you enter more than one statement in the editor, you must use a [terminating semicolon](/spanner/docs/reference/standard-sql/lexical#terminating_semicolons) after each statement except the last one.

4.  The *bottom pane* of a query tab provides the following subtabs:
    
      - The **Results** subtab shows the results when you run the statements in the editor. For queries it shows a results table, and for DML statements like `  INSERT  ` and `  UPDATE  ` it shows a message about how many rows were affected.
        
        Optionally, click **Export** to export your query results. It provides the following options:
        
          - Download CSV
          - Download JSON
          - Export to Google Sheets
          - Copy to clipboard (CSV)
          - Copy to clipboard (TSV)
          - Copy to clipboard (JSON)
    
      - The **Explanation** subtab shows visual graphs of the query plans created when you run the statements in the editor.

### View sampled query plans

1.  Go to the Spanner **Instances** page in Google Cloud console.

2.  Click the name of the instance with the queries that you want to investigate.
    
    Google Cloud console displays the instance's **Overview** page.

3.  In the **Navigation** menu and under the Observability heading, click **Query insights** .
    
    Google Cloud console displays the Instance's **Query insights** page.

4.  In the **Database drop-down** menu, select the database with the queries you want to investigate.
    
    Google Cloud console displays the query load information for the database. The TopN queries and tags table displays the list of top queries and request tags sorted by CPU utilization.

5.  Find the query with high CPU utilization for which you want to view sampled query plans. Click the **FPRINT** value of that query.
    
    The **Query details** page shows a **Query plans samples** graph for your query over time. You can zoom out to a maximum of seven days prior to the current time. Note: Query plans are not supported for queries with partitionTokens obtained from the PartitionQuery API and [Partitioned DML](/spanner/docs/dml-partitioned) queries.

6.  Click one of the dots in the graph to see an older query plan and visualize the steps taken during the query execution. You can also click any operator to see expanded information about the operator.
    
    **Figure 8.** Query plan samples graph.

### Take a tour of the query plan visualizer

The key components of the visualizer are annotated in the following screenshot and described in more detail. After running a query in a query tab, select the **EXPLANATION** tab below the query editor to open the query execution plan visualizer.

The data flow in the following diagram is bottom-up, that is, all the tables and indexes are at the bottom of the diagram and the final output is at the top.

**Figure 9.** Annotated query plan visualizer.

1.  The visualization of your plan can be large, depending on the query you executed. To hide and show details toggle the **EXPANDED/COMPACT** view selector. You can customize how much of the plan you see at any one time using the zoom control.

2.  The algebra that explains how Spanner runs the query is drawn as an acyclic graph, where each node corresponds to an iterator that consumes rows from its inputs and produces rows to its parent. A sample plan is shown in **Figure 9** . Click the diagram to see an expanded view of some of the details of the plan.
    
    **Figure 9.** Sample visual plan ( **Click to zoom in** ).
    
    Each node, or *card* , on the graph represents an iterator and contains the following information:
    
      - The iterator name. An iterator consumes rows from its input and produces rows.
      - Runtime statistics telling you how many rows were returned, what the latency was, and how much CPU was consumed.
      - We provide the following **visual cues** to help you identify potential issues within the query execution plan.
      - Red bars in a node are visual indicators of the percentage of latency or CPU time for this iterator compared to the total for the query.
      - The thickness of lines connecting each node represents the row count. The thicker the line, the larger the number of rows passed to the next node. The actual number of rows is displayed in each card and when you hold the pointer over a connector.
      - A warning triangle is displayed on a node where a full table scan was performed. More details in the information panel include recommendations such as adding an index, or revising the query or schema in other ways if possible in order to avoid a full scan.
      - Select a card in the plan to see details in the information panel on the right ( **5** ).

3.  The execution plan mini-map shows a zoomed-out view of the full plan and is useful for determining the overall shape of the execution plan and for navigating to different parts of the plan quickly. Drag directly on the mini-map or click where you'd like to focus, in order to go to another part of the visual plan.

4.  Select **DOWNLOAD JSON** to download a JSON version of the execution plan, which is useful for troubleshooting. You can also share it when contacting the Spanner team for [support](/spanner/docs/getting-support) . Saving the JSON doesn't save the result of the query.
    
    To download and save a JSON version of the execution plan to visualize later:
    
    1.  In Spanner Studio, run a query.
    
    2.  Select the **Explanation** tab.
    
    3.  Click **DOWNLOAD JSON** to download the JSON version of the execution plan.
    
    4.  Save and copy the content of the JSON file.
    
    5.  Open a new query editor tab.
    
    6.  In the editor tab, enter:
        
        ``` text
          PROTO:
          CONTENT_OF_JSON
        ```
    
    7.  Click **Run** .
    
    8.  Select the **Explanation** tab below the query editor to view a visual representation of the downloaded execution plan.

5.  The information panel shows detailed contextual information about the selected node in the query plan diagram. The information is organized into the following categories.
    
      - **Iterator information** provides details, as well as runtime statistics, for the iterator card you selected in the graph.
      - **Query summary** provides details about the number of rows returned and the time it took to run the query. Prominent operators are those that exhibit significant latency, consume significant CPU relative to other operators, and return significant numbers of data rows.
      - **Query execution timeline** is a time-based graph that shows how long each machine group was running its portion of the query. A machine group might not necessarily be running for the entire duration of the query's running time. It's also possible that a machine group ran multiple times during the course of running the query, but the timeline here only represents the start of the first time it ran and the end of the last time it ran.

## Tune a query that exhibits poor performance

Imagine your company runs an online movie database that contains information about movies such as cast, production companies, movie details, and more. The service runs on Spanner, but has been experiencing some performance issues lately.

As lead developer for the service, you are asked to investigate these performance issues because they are causing poor ratings for the service. You open the Google Cloud console, go to your database instance and then open the [query editor](#query-editor-tour) . You enter the following query into the editor and run it.

``` text
SELECT
  t.title,
  MIN(t.production_year) AS year,
  ANY_VALUE(mc.note HAVING MIN t.production_year) AS note
FROM
  title AS t
JOIN
  movie_companies AS mc
ON
  t.id = mc.movie_id
WHERE
  t.title LIKE '% the %'
GROUP BY
  title;
```

The result of running this query is shown in the following screenshot. We formatted the query in the editor by selecting **FORMAT QUERY** . There is also a note in the top right of the screen telling us that the query is valid.

**Figure 1.** Query editor displaying the original query.

The **RESULTS** tab below the query editor shows that the query completed in just over two minutes. You decide to look closer at the query to see whether the query is efficient.

### Analyze slow query with the query plan visualizer

At this point, we know that the query in the preceding step takes over two minutes, but we don't know whether the query is as efficient as possible and, therefore, whether this duration is expected.

You select the **EXPLANATION** tab just below the query editor to view a visual representation of the execution plan that Spanner created to run the query and return results.

The plan shown in the following screenshot is relatively large but, even at this zoom level, you can make the following observations.

  - Based on the **Query summary** in the information panel on the right, we learn that nearly 3 million rows were scanned and under 64K were ultimately returned.

  - We can also see from the **Query execution timeline** panel that 4 machine groups were involved in the query. A machine group is responsible for the execution of a portion of the query. Operators may execute on one or more machines. Selecting a machine group in the timeline highlights on the visual plan what part of the query was executed on that group.

**Figure 2.** Query plan visualizer showing the visual plan of the original query.

Because of these factors, you decide that an improvement in performance may be possible by changing the join from an apply join, which Spanner chose by default, to a [hash join](/spanner/docs/reference/standard-sql/query-syntax#join-methods) .

### Improve the query

To improve the performance of the query, you use a [join hint](/spanner/docs/reference/standard-sql/query-syntax#join-hints) to change the join method to a hash join. This join implementation executes set-based processing.

Here's the updated query:

``` text
SELECT
  t.title,
  MIN(t.production_year) AS year,
  ANY_VALUE(mc.note HAVING MIN t.production_year) AS note
FROM
  title AS t
JOIN
  @{join_method=hash_join} movie_companies AS mc
ON
  t.id = mc.movie_id
WHERE
  t.title LIKE '% the %'
GROUP BY
  title;
```

The following screenshot illustrates the updated query. As shown in the screenshot, the query completed in less than 5 seconds, a significant improvement over 120 seconds runtime before this change.

**Figure 3.** Query editor displaying the improved query.

Examine the new visual plan, shown in the following diagram, to see what it tells us about this improvement.

**Figure 4.** Query plan visualization after the query improvements ( **Click to zoom in** ).

Immediately, you notice some differences:

  - Only one machine group was involved in this query execution.

  - The number of aggregations has been reduced dramatically.

### Conclusion

In this scenario, we ran a slow query and looked at its visual plan to look for inefficiencies. The following is a summary of the queries and plans before and after any changes were made. Each tab shows the query that was run and a compact view of the full query execution plan visualization.

### Before

``` text
SELECT
  t.title,
  MIN(t.production_year) AS year,
  ANY_VALUE(mc.note
  HAVING
    MIN t.production_year) AS note
FROM
  title AS t
JOIN
  movie_companies AS mc
ON
  t.id = mc.movie_id
WHERE
  t.title LIKE '% the %'
GROUP BY
  title;
```

**Figure 5.** Compact view of the visual plan before improvements.

### After

``` text
SELECT
  t.title,
  MIN(t.production_year) AS year,
  ANY_VALUE(mc.note
  HAVING
    MIN t.production_year) AS note
FROM
  title AS t
JOIN
  @{join_method=hash_join} movie_companies AS mc
ON
  t.id = mc.movie_id
WHERE
  t.title LIKE '% the %'
GROUP BY
  title;
```

**Figure 6.** Compact view of the visual plan after improvements.

An indicator that something could be improved in this scenario was that a large proportion of the rows from the table **title** qualified the filter `  LIKE '% the %'  ` . Seeking into another table with so many rows is likely to be expensive. Changing our join implementation to a hash join improved performance significantly.

## What's next

  - For the complete query plan reference, refer to [Query execution plans](/spanner/docs/query-execution-plans) .

  - For the complete operator reference, refer to [Query execution operators](/spanner/docs/query-execution-operators) .
