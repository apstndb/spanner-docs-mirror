*Oldest active queries* , also known as *longest running queries* , is a list of queries that are active in your database, sorted by how long they've been running. Gaining insight into these queries can help identify causes of system latency and high CPU usage as they are happening.

Spanner provides a built-in table, `  SPANNER_SYS.OLDEST_ACTIVE_QUERIES  ` , that lists running queries, including queries containing DML statements, sorted by start time, in ascending order. It does not include change stream queries.

If there are many running queries, the results might be limited to a subset of the total queries because of the memory constraints that the system enforces on the collection of this data. Therefore, Spanner provides an additional table, `  SPANNER_SYS.ACTIVE_QUERIES_SUMMARY  ` , that shows summary statistics for all active queries (except for change stream queries). You can retrieve information from both of these built-in tables using SQL statements.

In this document, we'll describe both tables, show some example queries that use these tables and, finally, demonstrate how to use them to help mitigate issues caused by active queries.

## Access oldest active queries statistics

`  SPANNER_SYS  ` data is available only through SQL interfaces; for example:

  - A database's **Spanner Studio** page in the Google Cloud console

  - The [`  gcloud spanner databases execute-sql  `](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/execute-sql) command

  - The [`  executeSql  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql) or the [`  executeStreamingSql  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql) method

Spanner doesn't support `  SPANNER_SYS  ` with the following single read methods:

  - Performing a strong read from a single row or multiple rows in a table.
  - Performing a stale read from a single row or multiple rows in a table.
  - Reading from a single row or multiple rows in a secondary index.

## `     OLDEST_ACTIVE_QUERIES    ` statistics

`  SPANNER_SYS.OLDEST_ACTIVE_QUERIES  ` returns a list of active queries sorted by the start time. If there are many running queries, the results might be limited to a subset of the total queries because of the memory constraints Spanner enforces on the collection of this data. To view summary statistics for all active queries, see [`  ACTIVE_QUERIES_SUMMARY  `](https://docs.cloud.google.com/spanner/docs/introspection/oldest-active-queries#active-queries-summary) .

### Schema for all oldest active queries statistics table

| Column name                        | Type                       | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ---------------------------------- | -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `        START_TIME       `        | `        TIMESTAMP       ` | Start time of the query.                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `        TEXT_FINGERPRINT       `  | `        INT64       `     | Fingerprint is a hash of the request tag, or if a tag isn't present, a hash of the query text.                                                                                                                                                                                                                                                                                                                                                                                                        |
| `        TEXT       `              | `        STRING       `    | The query statement text.                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `        TEXT_TRUNCATED       `    | `        BOOL       `      | If the query text in the `        TEXT       ` field is truncated, this value is `        TRUE       ` . If the query text is not truncated, this value is `        FALSE       ` .                                                                                                                                                                                                                                                                                                                   |
| `        SESSION_ID       `        | `        STRING       `    | The ID of the session that is executing the query.                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `        QUERY_ID       `          | `        STRING       `    | The ID of the query. You can use this ID with [`         CALL cancel_query(query_id)        `](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/dml-syntax#call_statement) to cancel the query.                                                                                                                                                                                                                                                                                      |
| `        CLIENT_IP_ADDRESS       ` | `        STRING       `    | The IP address of the client that requested the query. Sometimes, the client IP address might be redacted. The IP address shown here is consistent with audit logs and follows the same redaction guidelines. For more information, see [IP address of the caller in audit logs](https://docs.cloud.google.com/logging/docs/audit#caller-ip) . We recommend requesting the client IP address only when the client IP is required, as requests for client IP addresses might incur additional latency. |
| `        API_CLIENT_HEADER       ` | `        STRING       `    | The `        api_client       ` header from the client.                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `        USER_AGENT_HEADER       ` | `        STRING       `    | The `        user_agent       ` header that Spanner received from the client.                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `        SERVER_REGION       `     | `        STRING       `    | The region where the Spanner root server processes the query. For more information, see [Life of a query](https://docs.cloud.google.com/spanner/docs/query-execution-plans#life-of-query) .                                                                                                                                                                                                                                                                                                           |
| `        PRIORITY       `          | `        STRING       `    | The priority of the query. To view available priorities, see [RequestOptions](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/RequestOptions#priority) .                                                                                                                                                                                                                                                                                                                                 |
| `        TRANSACTION_TYPE       `  | `        STRING       `    | The query's transaction type. Possible values are `        READ_ONLY       ` , `        READ_WRITE       ` , and `        NONE       ` .                                                                                                                                                                                                                                                                                                                                                              |

### Example queries

You can run the following example SQL statements using the [client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) , the [Google Cloud CLI](https://docs.cloud.google.com/spanner/docs/gcloud-spanner#execute_sql_statements) , or the [Google Cloud console](https://docs.cloud.google.com/spanner/docs/create-query-database-console#run_a_query) .

#### List oldest running active queries

The following query returns a list of oldest running queries sorted by the start time of the query.

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

##### Query output

The following table shows the output for running the previously mentioned query:

| start\_time                 | text\_fingerprint     | text                                                                                                                                                                                                  | text\_truncated | session\_id       | query\_id           | api\_client\_header                                                        | server\_region | priority       | transaction\_type |
| --------------------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- | ----------------- | ------------------- | -------------------------------------------------------------------------- | -------------- | -------------- | ----------------- |
| 2025-05-20T03:29:54.287255Z | \-3426560921851907385 | SELECT a.SingerId, a.AlbumId, a.TrackId, b.SingerId as b\_id, b.AlbumId as b\_albumid, b.TrackId as b\_trackId FROM Songs as a CROSS JOIN Songs as b;                                                 | FALSE           | AG46FS6K3adF      | 9023439241169932454 | gl-go/1.25.0-20250216-RC00 gccl/1.73.0 gapic/1.73.0 gax/2.14.1 grpc/1.69.2 | us-central1    | PRIORITY\_HIGH | READ\_ONLY        |
| 2025-05-20T03:31:52.40808Z  | 1688332608621812214   | SELECT a.SingerId, a.AlbumId, a.TrackId, a.SongName, s.FirstName, s.LastName FROM Songs as a JOIN Singers as s ON s.SingerId = a.SingerId WHERE STARTS\_WITH(s.FirstName, 'FirstName') LIMIT 1000000; | FALSE           | AG46FS6paJPKDOb   | 2729381896189388167 | gl-go/1.25.0-20250216-RC00 gccl/1.73.0 gapic/1.73.0 gax/2.14.1 grpc/1.69.2 | us-central1    | PRIORITY\_HIGH | READ\_WRITE       |
| 2025-05-20T03:31:52.591212Z | 6561582859583559006   | SELECT a.SingerId, a.AlbumId, a.TrackId, a.SongName, s.FirstName, s.LastName FROM Songs as a JOIN Singers as s ON s.SingerId = a.SingerId WHERE a.SingerId \> 10 LIMIT 1000000;                       | FALSE           | AG46FS7Pb\_9H6J6p | 9125776389780080794 | gl-go/1.25.0-20250216-RC00 gccl/1.73.0 gapic/1.73.0 gax/2.14.1 grpc/1.69.2 | us-central1    | PRIORITY\_LOW  | READ\_ONLY        |

#### Listing the top 2 oldest running queries

A slight variation on the preceding query, this example returns the top 2 oldest running queries sorted by the start time of the query.

    SELECT start_time,
           text_fingerprint,
           text,
           text_truncated,
           session_id
    FROM spanner_sys.oldest_active_queries
    ORDER BY start_time ASC LIMIT 2;

##### Query output

The following table shows the output for running the previously mentioned query:

| start\_time                 | text\_fingerprint     | text                                                                                                                                                                                                  | text\_truncated | session\_id |
| --------------------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- | ----------- |
| 2039-07-18T07:52:28.225877Z | \-3426560921851907385 | SELECT a.SingerId, a.AlbumId, a.TrackId, b.SingerId as b\_id, b.AlbumId as b\_albumid, b.TrackId as b\_trackId FROM Songs as a CROSS JOIN Songs as b;                                                 | False           | ACjbPvYsuRt |
| 2039-07-18T07:54:08.622081Z | \-9206690983832919848 | SELECT a.SingerId, a.AlbumId, a.TrackId, a.SongName, s.FirstName, s.LastName FROM Songs as a JOIN Singers as s ON s.SingerId = a.SingerId WHERE STARTS\_WITH(s.FirstName, 'FirstName') LIMIT 1000000; | False           | ACjbPvaF3yK |

## `     ACTIVE_QUERIES_SUMMARY    `

The `  SPANNER_SYS.ACTIVE_QUERIES_SUMMARY  ` statistics table shows summary statistics for all active queries. Queries are grouped into the following buckets:

  - older than 1 second
  - older than 10 seconds
  - older than 100 seconds

### Table schema for `     ACTIVE_QUERIES_SUMMARY    `

| Column name                            | Type                       | Description                                                   |
| -------------------------------------- | -------------------------- | ------------------------------------------------------------- |
| `        ACTIVE_COUNT       `          | `        INT64       `     | The total number of queries that are running.                 |
| `        OLDEST_START_TIME       `     | `        TIMESTAMP       ` | An upper bound on the start time of the oldest running query. |
| `        COUNT_OLDER_THAN_1S       `   | `        INT64       `     | The number of queries older than 1 second.                    |
| `        COUNT_OLDER_THAN_10S       `  | `        INT64       `     | The number of queries older than 10 seconds.                  |
| `        COUNT_OLDER_THAN_100S       ` | `        INT64       `     | The number of queries older than 100 seconds.                 |

A query can be counted in more than one of these buckets. For example, if a query has been running for 12 seconds, it will be counted in `  COUNT_OLDER_THAN_1S  ` and `  COUNT_OLDER_THAN_10S  ` because it satisfies both criteria.

### Example queries

You can run the following example SQL statements using the [client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) , the [gcloud spanner](https://docs.cloud.google.com/spanner/docs/gcloud-spanner#execute_sql_statements) , or the [Google Cloud console](https://docs.cloud.google.com/spanner/docs/create-query-database-console#run_a_query) .

#### Retrieve a summary of active queries

The following query returns the summary stats about running queries.

    SELECT active_count,
           oldest_start_time,
           count_older_than_1s,
           count_older_than_10s,
           count_older_than_100s
    FROM spanner_sys.active_queries_summary;

##### Query output

| active\_count |     oldest\_start\_time     | count\_older\_than\_1s | count\_older\_than\_10s | count\_older\_than\_100s |
| :-----------: | :-------------------------: | :--------------------: | :---------------------: | :----------------------: |
|      22       | 2039-07-18T07:52:28.225877Z |           21           |           21            |            1             |

## Limitations

While the goal is to give you the most comprehensive insights possible, there are some circumstances under which queries are not included in the data returned in these tables.

  - DML queries ( `  UPDATE  ` , `  INSERT  ` , `  DELETE  ` ) are not included if they're in the [Apply mutations](https://docs.cloud.google.com/spanner/docs/query-execution-operators#apply-mutations) phase.

  - A query is not included if it is in the middle of restarting due to a transient error.

  - Queries from overloaded or unresponsive servers are not included.

  - Reading or querying from the `  OLDEST_ACTIVE_QUERIES  ` table can't be done in a read-write transaction. Even in a read-only transaction, it ignores the transaction timestamp and always returns current data as of its execution. In rare cases, it may return an `  ABORTED  ` error with partial results; in that case, discard the partial results and attempt the query again.

  - If the `  CLIENT_IP_ADDRESS  ` column returns an `  <error>  ` string, it indicates a transient issue that shouldn't affect the rest of the query. Retry the query to retrieve the client IP address.

## Use active queries data to troubleshoot high CPU utilization

[Query statistics](https://docs.cloud.google.com/spanner/docs/introspection/query-statistics) and [transaction statistics](https://docs.cloud.google.com/spanner/docs/introspection/transaction-statistics) provide useful information when troubleshooting latency in a Spanner database. These tools provide information about the queries that have already completed. However, sometimes it is necessary to know what is running in the system. For example, consider the scenario when CPU utilization is quite high and you want to answer the following questions.

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

    SELECT active_count,
           oldest_start_time,
           count_older_than_1s,
           count_older_than_10s,
           count_older_than_100s
    FROM spanner_sys.active_queries_summary;

The query yields the following results.

|    active\_count    |             oldest\_start\_time              | count\_older\_than\_1s | count\_older\_than\_10s | count\_older\_than\_100s |
| :-----------------: | :------------------------------------------: | :--------------------: | :---------------------: | :----------------------: |
| `        22       ` | `        2039-07-18T07:52:28.225877Z       ` |  `        21       `   |   `        21       `   |    `        1       `    |

It turns out we have one query that is running for more that 100 seconds. This is unusual for our database, so we want to investigate further.

### Retrieve a list of active queries

We determined in the preceding step that we have a query running for over 100 seconds.To investigate further, we run the following query to return more information about the top 5 oldest running queries.

    SELECT start_time,
           text_fingerprint,
           text,
           text_truncated,
           session_id,
           query_id
    FROM spanner_sys.oldest_active_queries
    ORDER BY start_time ASC LIMIT 5;

In this example, we ran the query on March 28, 2024 at approximately 16:44:09 PM EDT and it returned the following results. (You might need to scroll horizontally to see the entire output.)

| start\_time                      | text\_fingerprint     | text                                                                                                                                                  | text\_truncated | session\_id                                                    | query\_id         |
| :------------------------------- | :-------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------: | :------------------------------------------------------------- | ----------------- |
| 2024-03-28 16:44:09.356939+00:00 | \-2833175298673875968 | select \* from spanner\_sys.oldest\_active\_queries                                                                                                   |      false      | ACjbPvYsucrtcffHrRK6aObeIjZf12tSUwOsim-g1WC3IhqF4epzICCQR3GCHw | 37190103859320827 |
| 2039-07-18T07:52:28.225877Z      | \-3426560921851907385 | SELECT a.SingerId, a.AlbumId, a.TrackId, b.SingerId as b\_id, b.AlbumId as b\_albumid, b.TrackId as b\_trackId FROM Songs as a CROSS JOIN Songs as b; |      false      | ACjbPvaF3yKiNfxXFod2LPoFaXjKR759Bw1o34206vv0t7eOrD3wxZhu8U6ohQ | 48946620525959556 |

The oldest query (fingerprint = `  -2833175298673875968  ` ) is highlighted in the table. It is an expensive [`  CROSS JOIN  `](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/query-syntax#cross_join) . We decide to take action.

### Cancel an expensive query

In this example, we found a query that was running an expensive `  CROSS JOIN  ` so we decide to cancel the query. The query results we received in the preceding step included a `  query_id  ` . We can run the following `  CALL cancel_query(query_id)  ` command for [GoogleSQL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/dml-syntax#call_statement) and the `  spanner.cancel_query(query_id)  ` command for [PostgreSQL](https://docs.cloud.google.com/spanner/docs/reference/postgresql/dml-syntax#call_statement) to cancel the query.

### GoogleSQL

    CALL cancel_query(query_id)

### PostgreSQL

    CALL spanner.cancel_query(query_id)

For example, in the following, the `  CALL  ` statement cancels a query with the ID `  37190103859320827  ` :

    CALL cancel_query('37190103859320827')

**Note:** In Spanner, the `  CALL  ` operation only supports the `  cancel_query  ` procedure call. For more information about stored procedures, see the Stored procedures page ( [GoogleSQL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/stored-procedures) or [PostgreSQL](https://docs.cloud.google.com/spanner/docs/reference/postgresql/stored-procedures-pg) ).

You need to [query the `  spanner_sys.oldest_active_queries  ` table](https://docs.cloud.google.com/spanner/docs/introspection/oldest-active-queries#retrieve_a_list_of_active_queries) to verify that the query is cancelled.

**Note:** `  CALL cancel_query  ` is a best effort operation. It might not cancel a query when Spanner servers are busy due to heavy query loads. If this occurs, try executing `  CALL cancel_query  ` a second time.

This walkthrough demonstrates how to use `  SPANNER_SYS.OLDEST_ACTIVE_QUERIES  ` and `  SPANNER_SYS.ACTIVE_QUERIES_SUMMARY  ` to analyze our running queries and take action if necessary on any queries that are contributing to high CPU usage. Of course, it is always cheaper to avoid expensive operations and to design the right schema for your use cases. For more information on constructing SQL statements that run efficiently, see [SQL best practices](https://docs.cloud.google.com/spanner/docs/sql-best-practices) .

## What's next

  - Learn about other [Introspection tools](https://docs.cloud.google.com/spanner/docs/introspection) .
  - Learn about other information Spanner stores for each database in the database's [information schema](https://docs.cloud.google.com/spanner/docs/information-schema) tables.
  - Learn more about [SQL best practices](https://docs.cloud.google.com/spanner/docs/sql-best-practices) for Spanner.
