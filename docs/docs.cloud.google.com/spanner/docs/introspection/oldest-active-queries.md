*Oldest active queries* , also known as *longest running queries* , is a list of queries that are active in your database, sorted by how long they've been running. Gaining insight into these queries can help identify causes of system latency and high CPU usage as they are happening.

Spanner provides a built-in table, `  SPANNER_SYS.OLDEST_ACTIVE_QUERIES  ` , that lists running queries, including queries containing DML statements, sorted by start time, in ascending order. It does not include change stream queries.

If there are many running queries, the results might be limited to a subset of the total queries because of the memory constraints that the system enforces on the collection of this data. Therefore, Spanner provides an additional table, `  SPANNER_SYS.ACTIVE_QUERIES_SUMMARY  ` , that shows summary statistics for all active queries (except for change stream queries). You can retrieve information from both of these built-in tables using SQL statements.

In this document, we'll describe both tables, show some example queries that use these tables and, finally, demonstrate how to use them to help mitigate issues caused by active queries.

## Access oldest active queries statistics

`  SPANNER_SYS  ` data is available only through SQL interfaces; for example:

  - A database's **Spanner Studio** page in the Google Cloud console

  - The [`  gcloud spanner databases execute-sql  `](/sdk/gcloud/reference/spanner/databases/execute-sql) command

  - The [`  executeSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql) or the [`  executeStreamingSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql) method

Spanner doesn't support `  SPANNER_SYS  ` with the following single read methods:

  - Performing a strong read from a single row or multiple rows in a table.
  - Performing a stale read from a single row or multiple rows in a table.
  - Reading from a single row or multiple rows in a secondary index.

## `     OLDEST_ACTIVE_QUERIES    ` statistics

`  SPANNER_SYS.OLDEST_ACTIVE_QUERIES  ` returns a list of active queries sorted by the start time. If there are many running queries, the results might be limited to a subset of the total queries because of the memory constraints Spanner enforces on the collection of this data. To view summary statistics for all active queries, see [`  ACTIVE_QUERIES_SUMMARY  `](#active-queries-summary) .

### Schema for all oldest active queries statistics table

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       START_TIME      </code></td>
<td><code dir="ltr" translate="no">       TIMESTAMP      </code></td>
<td>Start time of the query.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       TEXT_FINGERPRINT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Fingerprint is a hash of the request tag, or if a tag isn't present, a hash of the query text.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       TEXT      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The query statement text.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       TEXT_TRUNCATED      </code></td>
<td><code dir="ltr" translate="no">       BOOL      </code></td>
<td>If the query text in the <code dir="ltr" translate="no">       TEXT      </code> field is truncated, this value is <code dir="ltr" translate="no">       TRUE      </code> . If the query text is not truncated, this value is <code dir="ltr" translate="no">       FALSE      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       SESSION_ID      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The ID of the session that is executing the query.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       QUERY_ID      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The ID of the query. You can use this ID with <a href="/spanner/docs/reference/standard-sql/dml-syntax#call_statement"><code dir="ltr" translate="no">        CALL cancel_query(query_id)       </code></a> to cancel the query.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       CLIENT_IP_ADDRESS      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The IP address of the client that requested the query. Sometimes, the client IP address might be redacted. The IP address shown here is consistent with audit logs and follows the same redaction guidelines. For more information, see <a href="/logging/docs/audit#caller-ip">IP address of the caller in audit logs</a> . We recommend requesting the client IP address only when the client IP is required, as requests for client IP addresses might incur additional latency.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       API_CLIENT_HEADER      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The <code dir="ltr" translate="no">       api_client      </code> header from the client.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       USER_AGENT_HEADER      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The <code dir="ltr" translate="no">       user_agent      </code> header that Spanner received from the client.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       SERVER_REGION      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The region where the Spanner root server processes the query. For more information, see <a href="/spanner/docs/query-execution-plans#life-of-query">Life of a query</a> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       PRIORITY      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The priority of the query. To view available priorities, see <a href="/spanner/docs/reference/rest/v1/RequestOptions#priority">RequestOptions</a> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       TRANSACTION_TYPE      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The query's transaction type. Possible values are <code dir="ltr" translate="no">       READ_ONLY      </code> , <code dir="ltr" translate="no">       READ_WRITE      </code> , and <code dir="ltr" translate="no">       NONE      </code> .</td>
</tr>
</tbody>
</table>

### Example queries

You can run the following example SQL statements using the [client libraries](/spanner/docs/reference/libraries) , the [Google Cloud CLI](/spanner/docs/gcloud-spanner#execute_sql_statements) , or the [Google Cloud console](/spanner/docs/create-query-database-console#run_a_query) .

#### List oldest running active queries

The following query returns a list of oldest running queries sorted by the start time of the query.

``` text
SELECT start_time,
       text_fingerprint,
       text,
       text_truncated,
       session_id,
       query_id,
       api_client_header,
       server_region,
       priority,
       transaction_type
FROM spanner_sys.oldest_active_queries
ORDER BY start_time ASC;
```

##### Query output

The following table shows the output for running the previously mentioned query:

<table>
<thead>
<tr class="header">
<th>start_time</th>
<th>text_fingerprint</th>
<th>text</th>
<th>text_truncated</th>
<th>session_id</th>
<th>query_id</th>
<th>api_client_header</th>
<th>server_region</th>
<th>priority</th>
<th>transaction_type</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>2025-05-20T03:29:54.287255Z</td>
<td>-3426560921851907385</td>
<td>SELECT a.SingerId, a.AlbumId, a.TrackId, b.SingerId as b_id, b.AlbumId as b_albumid, b.TrackId as b_trackId FROM Songs as a CROSS JOIN Songs as b;</td>
<td>FALSE</td>
<td>AG46FS6K3adF</td>
<td>9023439241169932454</td>
<td>gl-go/1.25.0-20250216-RC00 gccl/1.73.0 gapic/1.73.0 gax/2.14.1 grpc/1.69.2</td>
<td>us-central1</td>
<td>PRIORITY_HIGH</td>
<td>READ_ONLY</td>
</tr>
<tr class="even">
<td>2025-05-20T03:31:52.40808Z</td>
<td>1688332608621812214</td>
<td>SELECT a.SingerId, a.AlbumId, a.TrackId, a.SongName, s.FirstName, s.LastName FROM Songs as a JOIN Singers as s ON s.SingerId = a.SingerId WHERE STARTS_WITH(s.FirstName, 'FirstName') LIMIT 1000000;</td>
<td>FALSE</td>
<td>AG46FS6paJPKDOb</td>
<td>2729381896189388167</td>
<td>gl-go/1.25.0-20250216-RC00 gccl/1.73.0 gapic/1.73.0 gax/2.14.1 grpc/1.69.2</td>
<td>us-central1</td>
<td>PRIORITY_HIGH</td>
<td>READ_WRITE</td>
</tr>
<tr class="odd">
<td>2025-05-20T03:31:52.591212Z</td>
<td>6561582859583559006</td>
<td>SELECT a.SingerId, a.AlbumId, a.TrackId, a.SongName, s.FirstName, s.LastName FROM Songs as a JOIN Singers as s ON s.SingerId = a.SingerId WHERE a.SingerId &gt; 10 LIMIT 1000000;</td>
<td>FALSE</td>
<td>AG46FS7Pb_9H6J6p</td>
<td>9125776389780080794</td>
<td>gl-go/1.25.0-20250216-RC00 gccl/1.73.0 gapic/1.73.0 gax/2.14.1 grpc/1.69.2</td>
<td>us-central1</td>
<td>PRIORITY_LOW</td>
<td>READ_ONLY</td>
</tr>
</tbody>
</table>

#### Listing the top 2 oldest running queries

A slight variation on the preceding query, this example returns the top 2 oldest running queries sorted by the start time of the query.

``` text
SELECT start_time,
       text_fingerprint,
       text,
       text_truncated,
       session_id
FROM spanner_sys.oldest_active_queries
ORDER BY start_time ASC LIMIT 2;
```

##### Query output

The following table shows the output for running the previously mentioned query:

<table>
<thead>
<tr class="header">
<th>start_time</th>
<th>text_fingerprint</th>
<th>text</th>
<th>text_truncated</th>
<th>session_id</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>2039-07-18T07:52:28.225877Z</td>
<td>-3426560921851907385</td>
<td>SELECT a.SingerId, a.AlbumId, a.TrackId, b.SingerId as b_id, b.AlbumId as b_albumid, b.TrackId as b_trackId FROM Songs as a CROSS JOIN Songs as b;</td>
<td>False</td>
<td>ACjbPvYsuRt</td>
</tr>
<tr class="even">
<td>2039-07-18T07:54:08.622081Z</td>
<td>-9206690983832919848</td>
<td>SELECT a.SingerId, a.AlbumId, a.TrackId, a.SongName, s.FirstName, s.LastName FROM Songs as a JOIN Singers as s ON s.SingerId = a.SingerId WHERE STARTS_WITH(s.FirstName, 'FirstName') LIMIT 1000000;</td>
<td>False</td>
<td>ACjbPvaF3yK</td>
</tr>
</tbody>
</table>

## `     ACTIVE_QUERIES_SUMMARY    `

The `  SPANNER_SYS.ACTIVE_QUERIES_SUMMARY  ` statistics table shows summary statistics for all active queries. Queries are grouped into the following buckets:

  - older than 1 second
  - older than 10 seconds
  - older than 100 seconds

### Table schema for `     ACTIVE_QUERIES_SUMMARY    `

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       ACTIVE_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The total number of queries that are running.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       OLDEST_START_TIME      </code></td>
<td><code dir="ltr" translate="no">       TIMESTAMP      </code></td>
<td>An upper bound on the start time of the oldest running query.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       COUNT_OLDER_THAN_1S      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The number of queries older than 1 second.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       COUNT_OLDER_THAN_10S      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The number of queries older than 10 seconds.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       COUNT_OLDER_THAN_100S      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The number of queries older than 100 seconds.</td>
</tr>
</tbody>
</table>

A query can be counted in more than one of these buckets. For example, if a query has been running for 12 seconds, it will be counted in `  COUNT_OLDER_THAN_1S  ` and `  COUNT_OLDER_THAN_10S  ` because it satisfies both criteria.

### Example queries

You can run the following example SQL statements using the [client libraries](/spanner/docs/reference/libraries) , the [gcloud spanner](/spanner/docs/gcloud-spanner#execute_sql_statements) , or the [Google Cloud console](/spanner/docs/create-query-database-console#run_a_query) .

#### Retrieve a summary of active queries

The following query returns the summary stats about running queries.

``` text
SELECT active_count,
       oldest_start_time,
       count_older_than_1s,
       count_older_than_10s,
       count_older_than_100s
FROM spanner_sys.active_queries_summary;
```

##### Query output

<table>
<thead>
<tr class="header">
<th style="text-align: center;">active_count</th>
<th style="text-align: center;">oldest_start_time</th>
<th style="text-align: center;">count_older_than_1s</th>
<th style="text-align: center;">count_older_than_10s</th>
<th style="text-align: center;">count_older_than_100s</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">22</td>
<td style="text-align: center;">2039-07-18T07:52:28.225877Z</td>
<td style="text-align: center;">21</td>
<td style="text-align: center;">21</td>
<td style="text-align: center;">1</td>
</tr>
</tbody>
</table>

## Limitations

While the goal is to give you the most comprehensive insights possible, there are some circumstances under which queries are not included in the data returned in these tables.

  - DML queries ( `  UPDATE  ` , `  INSERT  ` , `  DELETE  ` ) are not included if they're in the [Apply mutations](/spanner/docs/query-execution-operators#apply-mutations) phase.

  - A query is not included if it is in the middle of restarting due to a transient error.

  - Queries from overloaded or unresponsive servers are not included.

  - Reading or querying from the `  OLDEST_ACTIVE_QUERIES  ` table can't be done in a read-write transaction. Even in a read-only transaction, it ignores the transaction timestamp and always returns current data as of its execution. In rare cases, it may return an `  ABORTED  ` error with partial results; in that case, discard the partial results and attempt the query again.

  - If the `  CLIENT_IP_ADDRESS  ` column returns an `  <error>  ` string, it indicates a transient issue that shouldn't affect the rest of the query. Retry the query to retrieve the client IP address.

## Use active queries data to troubleshoot high CPU utilization

[Query statistics](/spanner/docs/introspection/query-statistics) and [transaction statistics](/spanner/docs/introspection/transaction-statistics) provide useful information when troubleshooting latency in a Spanner database. These tools provide information about the queries that have already completed. However, sometimes it is necessary to know what is running in the system. For example, consider the scenario when CPU utilization is quite high and you want to answer the following questions.

  - How many queries are running at the moment?
  - What are these queries?
  - How many queries are running for a long time, that is, greater than 100 seconds?
  - Which session is running the query?

With answers to the preceding questions you could decide to take the following action.

  - Delete the session executing the query for an immediate resolution.
  - Improve the query performance by adding an index.
  - Reduce the frequency of the query if it is associated with a periodic background task.
  - Identify user or component issuing the query which may not be authorized to execute the query.

In this walkthrough, we examine our active queries and determine what action, if any, to take.

### Retrieve a summary of active queries

In our example scenario, we notice higher than normal CPU usage, so we decide to run the following query to return a summary of active queries.

``` text
SELECT active_count,
       oldest_start_time,
       count_older_than_1s,
       count_older_than_10s,
       count_older_than_100s
FROM spanner_sys.active_queries_summary;
```

The query yields the following results.

<table>
<thead>
<tr class="header">
<th style="text-align: center;">active_count</th>
<th style="text-align: center;">oldest_start_time</th>
<th style="text-align: center;">count_older_than_1s</th>
<th style="text-align: center;">count_older_than_10s</th>
<th style="text-align: center;">count_older_than_100s</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;"><code dir="ltr" translate="no">       22      </code></td>
<td style="text-align: center;"><code dir="ltr" translate="no">       2039-07-18T07:52:28.225877Z      </code></td>
<td style="text-align: center;"><code dir="ltr" translate="no">       21      </code></td>
<td style="text-align: center;"><code dir="ltr" translate="no">       21      </code></td>
<td style="text-align: center;"><code dir="ltr" translate="no">       1      </code></td>
</tr>
</tbody>
</table>

It turns out we have one query that is running for more that 100 seconds. This is unusual for our database, so we want to investigate further.

### Retrieve a list of active queries

We determined in the preceding step that we have a query running for over 100 seconds.To investigate further, we run the following query to return more information about the top 5 oldest running queries.

``` text
SELECT start_time,
       text_fingerprint,
       text,
       text_truncated,
       session_id,
       query_id
FROM spanner_sys.oldest_active_queries
ORDER BY start_time ASC LIMIT 5;
```

In this example, we ran the query on March 28, 2024 at approximately 16:44:09 PM EDT and it returned the following results. (You might need to scroll horizontally to see the entire output.)

<table>
<thead>
<tr class="header">
<th style="text-align: left;">start_time</th>
<th style="text-align: left;">text_fingerprint</th>
<th style="text-align: left;">text</th>
<th style="text-align: center;">text_truncated</th>
<th style="text-align: left;">session_id</th>
<th>query_id</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">2024-03-28 16:44:09.356939+00:00</td>
<td style="text-align: left;">-2833175298673875968</td>
<td style="text-align: left;">select * from spanner_sys.oldest_active_queries</td>
<td style="text-align: center;">false</td>
<td style="text-align: left;">ACjbPvYsucrtcffHrRK6aObeIjZf12tSUwOsim-g1WC3IhqF4epzICCQR3GCHw</td>
<td>37190103859320827</td>
</tr>
<tr class="even">
<td style="text-align: left;">2039-07-18T07:52:28.225877Z</td>
<td style="text-align: left;">-3426560921851907385</td>
<td style="text-align: left;">SELECT a.SingerId, a.AlbumId, a.TrackId, b.SingerId as b_id, b.AlbumId as b_albumid, b.TrackId as b_trackId FROM Songs as a CROSS JOIN Songs as b;</td>
<td style="text-align: center;">false</td>
<td style="text-align: left;">ACjbPvaF3yKiNfxXFod2LPoFaXjKR759Bw1o34206vv0t7eOrD3wxZhu8U6ohQ</td>
<td>48946620525959556</td>
</tr>
</tbody>
</table>

The oldest query (fingerprint = `  -2833175298673875968  ` ) is highlighted in the table. It is an expensive [`  CROSS JOIN  `](/spanner/docs/reference/standard-sql/query-syntax#cross_join) . We decide to take action.

### Cancel an expensive query

In this example, we found a query that was running an expensive `  CROSS JOIN  ` so we decide to cancel the query. The query results we received in the preceding step included a `  query_id  ` . We can run the following `  CALL cancel_query(query_id)  ` command for [GoogleSQL](/spanner/docs/reference/standard-sql/dml-syntax#call_statement) and the `  spanner.cancel_query(query_id)  ` command for [PostgreSQL](/spanner/docs/reference/postgresql/dml-syntax#call_statement) to cancel the query.

### GoogleSQL

``` text
CALL cancel_query(query_id)
```

### PostgreSQL

``` text
CALL spanner.cancel_query(query_id)
```

For example, in the following, the `  CALL  ` statement cancels a query with the ID `  37190103859320827  ` :

``` text
CALL cancel_query('37190103859320827')
```

**Note:** In Spanner, the `  CALL  ` operation only supports the `  cancel_query  ` procedure call. For more information about stored procedures, see the Stored procedures page ( [GoogleSQL](/spanner/docs/reference/standard-sql/stored-procedures) or [PostgreSQL](/spanner/docs/reference/postgresql/stored-procedures-pg) ).

You need to [query the `  spanner_sys.oldest_active_queries  ` table](/spanner/docs/introspection/oldest-active-queries#retrieve_a_list_of_active_queries) to verify that the query is cancelled.

**Note:** `  CALL cancel_query  ` is a best effort operation. It might not cancel a query when Spanner servers are busy due to heavy query loads. If this occurs, try executing `  CALL cancel_query  ` a second time.

This walkthrough demonstrates how to use `  SPANNER_SYS.OLDEST_ACTIVE_QUERIES  ` and `  SPANNER_SYS.ACTIVE_QUERIES_SUMMARY  ` to analyze our running queries and take action if necessary on any queries that are contributing to high CPU usage. Of course, it is always cheaper to avoid expensive operations and to design the right schema for your use cases. For more information on constructing SQL statements that run efficiently, see [SQL best practices](/spanner/docs/sql-best-practices) .

## What's next

  - Learn about other [Introspection tools](/spanner/docs/introspection) .
  - Learn about other information Spanner stores for each database in the database's [information schema](/spanner/docs/information-schema) tables.
  - Learn more about [SQL best practices](/spanner/docs/sql-best-practices) for Spanner.
