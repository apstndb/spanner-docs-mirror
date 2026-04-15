This page describes how to monitor and troubleshoot queries that are running in your Spanner instance. Active queries are long-running queries that might affect the performance of your instance. Monitoring these queries can help you identify causes of instance latency and high CPU usage.

Using the Google Cloud console, you can view active queries on the Spanner **Query insights** page. These queries are sorted by the start time of the query. If there are many active queries, the results might be limited to a subset of total queries because of the memory constraints that Spanner enforces on data collection.

## Before you begin

To get the permissions that you need to view active queries, ask your administrator to grant you the following IAM roles on the instance:

  - [Cloud Spanner Viewer](https://docs.cloud.google.com/iam/docs/roles-permissions/spanner#spanner.viewer) ( `roles/spanner.viewer` )
  - [Cloud Spanner Database Reader](https://docs.cloud.google.com/iam/docs/roles-permissions/spanner#spanner.databaseReader) ( `roles/spanner.databaseReader` )

For more information about granting roles, see [Manage access to projects, folders, and organizations](https://docs.cloud.google.com/iam/docs/granting-changing-revoking-access) .

You might also be able to get the required permissions through [custom roles](https://docs.cloud.google.com/iam/docs/creating-custom-roles) or other [predefined roles](https://docs.cloud.google.com/iam/docs/roles-overview#predefined) .

## View active queries

To view active queries for each database, do the following:

1.  In the Google Cloud console, go to the Spanner **Instances** page.

2.  Click the instance that contains the queries that you want to monitor.

3.  In the navigation menu, click **Query insights** .

4.  Use the **Database** menu to specify the database that you want to monitor.

5.  Click the **Active queries** tab. The tab contains a summary of active queries, along with a table of the top 50 longest-running active queries.

## View longest-running queries

The following table describes the default columns in the **Longest running queries** table on the **Active queries** tab:

| Column name    | Description                                                                       |
| -------------- | --------------------------------------------------------------------------------- |
| Query ID       | The unique identifier for the query.                                              |
| Query          | The SQL query text.                                                               |
| Fingerprint    | Hash of the request tag, or if a tag isn't present, a hash of the SQL query text. |
| Start time     | The timestamp for when the query started.                                         |
| Query duration | The duration the active query has been running.                                   |
| Action         | Contains a link to terminate the query.                                           |

When the **Auto refresh** toggle is on, the display refreshes every 60 seconds.

You can use the **Column display options** button to select any of the following optional columns to show up in the Longest running queries table:

| Column name       | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Client IP address | The IP address of the client that requested the query. Sometimes, the client IP address might be redacted. The IP address shown here is consistent with audit logs and follows the same redaction guidelines. For more information, see [IP address of the caller in audit logs](https://docs.cloud.google.com/logging/docs/audit#caller-ip) . Spanner recommends requesting the client IP address only when the client IP is required, as requests for client IP addresses might incur additional latency. |
| Server region     | The region where the Spanner root server processes the query. For more information, see [Life of a query](https://docs.cloud.google.com/spanner/docs/query-execution-plans#life-of-query) .                                                                                                                                                                                                                                                                                                                 |
| Transaction type  | The query's transaction type. Possible values are `READ_ONLY` , `READ_WRITE` , and `NONE` .                                                                                                                                                                                                                                                                                                                                                                                                                 |
| API client header | The `api_client` header from the client.                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Priority          | The priority of the query. To view available priorities, see [RequestOptions](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/RequestOptions#priority) .                                                                                                                                                                                                                                                                                                                                       |
| User agent header | The `user_agent` header that Spanner receives from the client.                                                                                                                                                                                                                                                                                                                                                                                                                                              |

## Terminate a query

You can terminate a query that is running in your instance. Terminating a query might help free up resources and reduce the load on your instance. Terminating a query is a best-effort operation.

The Google Cloud console page refreshes after you initiate the termination. If the termination is successful, the query is removed from the table. If the termination fails, it does so in the background, and the query continues to appear in the **Longest running queries** table.

Spanner might not cancel a query when servers are busy. You can try to terminate a query again to cancel it.

To get the permission that you need to terminate a query, ask your administrator to grant you the Cloud Spanner Database Reader ( `spanner.databaseReader` ) IAM role on the instance.

This predefined role contains the `spanner.sessions.delete` permission, which is required to terminate a query.

To terminate a query, do the following:

1.  Select the query that you want to terminate from the **Longest running queries** table.
2.  In the **Actions** column, click **Terminate** .
3.  In the **Terminate query** window, click **Confirm** .

To terminate multiple queries, do the following:

1.  Select the queries you want to terminate from the **Longest running queries** table.
2.  Click **Terminate selected queries** .
3.  In the **Terminate the following queries** window, click **Confirm** .

## What's next

  - Learn more about [Analyze query performance](https://docs.cloud.google.com/spanner/docs/using-query-insights) .
  - Learn more about [Oldest active queries statistics](https://docs.cloud.google.com/spanner/docs/introspection/oldest-active-queries) .
