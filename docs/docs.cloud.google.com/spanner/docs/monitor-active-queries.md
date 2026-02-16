This page describes how to monitor and troubleshoot queries that are running in your Spanner instance. Active queries are long-running queries that might affect the performance of your instance. Monitoring these queries can help you identify causes of instance latency and high CPU usage.

Using the Google Cloud console, you can view active queries on the Spanner **Query insights** page. These queries are sorted by the start time of the query. If there are many active queries, the results might be limited to a subset of total queries because of the memory constraints that Spanner enforces on data collection.

## Before you begin

To get the permissions that you need to view active queries, ask your administrator to grant you the following IAM roles on the instance:

  - [Cloud Spanner Viewer](/iam/docs/roles-permissions/spanner#spanner.viewer) ( `  roles/spanner.viewer  ` )
  - [Cloud Spanner Database Reader](/iam/docs/roles-permissions/spanner#spanner.databaseReader) ( `  roles/spanner.databaseReader  ` )

For more information about granting roles, see [Manage access to projects, folders, and organizations](/iam/docs/granting-changing-revoking-access) .

You might also be able to get the required permissions through [custom roles](/iam/docs/creating-custom-roles) or other [predefined roles](/iam/docs/roles-overview#predefined) .

## View active queries

To view active queries for each database, do the following:

1.  In the Google Cloud console, go to the Spanner **Instances** page.

2.  Click the instance that contains the queries that you want to monitor.

3.  In the navigation menu, click **Query insights** .

4.  Use the **Database** menu to specify the database that you want to monitor.

5.  Click the **Active queries** tab. The tab contains a summary of active queries, along with a table of the top 50 longest-running active queries.

## View longest-running queries

The following table describes the default columns in the **Longest running queries** table on the **Active queries** tab:

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Query ID</td>
<td>The unique identifier for the query.</td>
</tr>
<tr class="even">
<td>Query</td>
<td>The SQL query text.</td>
</tr>
<tr class="odd">
<td>Fingerprint</td>
<td>Hash of the request tag, or if a tag isn't present, a hash of the SQL query text.</td>
</tr>
<tr class="even">
<td>Start time</td>
<td>The timestamp for when the query started.</td>
</tr>
<tr class="odd">
<td>Query duration</td>
<td>The duration the active query has been running.</td>
</tr>
<tr class="even">
<td>Action</td>
<td>Contains a link to terminate the query.</td>
</tr>
</tbody>
</table>

When the **Auto refresh** toggle is on, the display refreshes every 60 seconds.

You can use the **Column display options** button to select any of the following optional columns to show up in the Longest running queries table:

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Client IP address</td>
<td>The IP address of the client that requested the query. Sometimes, the client IP address might be redacted. The IP address shown here is consistent with audit logs and follows the same redaction guidelines. For more information, see <a href="/logging/docs/audit#caller-ip">IP address of the caller in audit logs</a> . Spanner recommends requesting the client IP address only when the client IP is required, as requests for client IP addresses might incur additional latency.</td>
</tr>
<tr class="even">
<td>Server region</td>
<td>The region where the Spanner root server processes the query. For more information, see <a href="/spanner/docs/query-execution-plans#life-of-query">Life of a query</a> .</td>
</tr>
<tr class="odd">
<td>Transaction type</td>
<td>The query's transaction type. Possible values are <code dir="ltr" translate="no">       READ_ONLY      </code> , <code dir="ltr" translate="no">       READ_WRITE      </code> , and <code dir="ltr" translate="no">       NONE      </code> .</td>
</tr>
<tr class="even">
<td>API client header</td>
<td>The <code dir="ltr" translate="no">       api_client      </code> header from the client.</td>
</tr>
<tr class="odd">
<td>Priority</td>
<td>The priority of the query. To view available priorities, see <a href="/spanner/docs/reference/rest/v1/RequestOptions#priority">RequestOptions</a> .</td>
</tr>
<tr class="even">
<td>User agent header</td>
<td>The <code dir="ltr" translate="no">       user_agent      </code> header that Spanner receives from the client.</td>
</tr>
</tbody>
</table>

## Terminate a query

You can terminate a query that is running in your instance. Terminating a query might help free up resources and reduce the load on your instance. Terminating a query is a best-effort operation.

The Google Cloud console page refreshes after you initiate the termination. If the termination is successful, the query is removed from the table. If the termination fails, it does so in the background, and the query continues to appear in the **Longest running queries** table.

Spanner might not cancel a query when servers are busy. You can try to terminate a query again to cancel it.

To get the permission that you need to terminate a query, ask your administrator to grant you the Cloud Spanner Database Reader ( `  spanner.databaseReader  ` ) IAM role on the instance.

This predefined role contains the `  spanner.sessions.delete  ` permission, which is required to terminate a query.

To terminate a query, do the following:

1.  Select the query that you want to terminate from the **Longest running queries** table.
2.  In the **Actions** column, click **Terminate** .
3.  In the **Terminate query** window, click **Confirm** .

To terminate multiple queries, do the following:

1.  Select the queries you want to terminate from the **Longest running queries** table.
2.  Click **Terminate selected queries** .
3.  In the **Terminate the following queries** window, click **Confirm** .

## What's next

  - Learn more about [Analyze query performance](/spanner/docs/using-query-insights) .
  - Learn more about [Oldest active queries statistics](/spanner/docs/introspection/oldest-active-queries) .
