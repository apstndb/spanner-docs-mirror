Spanner provides built-in tables that keep many statistics for the queries and data manipulation language (DML) statements that used the most CPU, and all queries in aggregate (including [change stream](/spanner/docs/change-streams) queries).

## Access query statistics

**Note:** Spanner Studio (formerly labeled **Query** in the Google Cloud console) supports SQL, DML, and DDL operations in a single editor. For more information, see [Manage your data using the Google Cloud console](/spanner/docs/manage-data-using-console) .

Spanner provides the query statistics in the `  SPANNER_SYS  ` schema. You can use the following ways to access `  SPANNER_SYS  ` data:

  - A database's [Spanner Studio page](/spanner/docs/tune-query-with-visualizer#running-a-query) in the Google Cloud console.

  - The `  gcloud spanner databases execute-sql  ` command.

  - [Query insights](/spanner/docs/using-query-insights#the_dashboard) dashboards.

  - The [`  executeSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql) or the [`  executeStreamingSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql) method.

The following single read methods that Spanner provides don't support `  SPANNER_SYS  ` :

  - Performing a strong read from a single row or multiple rows in a table.
  - Performing a stale read from a single row or multiple rows in a table.
  - Reading from a single row or multiple rows in a secondary index.

For more information, see [Single read methods](/spanner/docs/reads#single_read_methods) .

**Note:** On a database's Spanner Studio page, you can copy and paste the example queries on this page to insert queries quickly.

## CPU usage grouped by query

The following tables track the queries with the highest CPU usage during a specific time period:

  - `  SPANNER_SYS.QUERY_STATS_TOP_MINUTE  ` : Queries during 1 minute intervals
  - `  SPANNER_SYS.QUERY_STATS_TOP_10MINUTE  ` : Queries during 10 minute intervals
  - `  SPANNER_SYS.QUERY_STATS_TOP_HOUR  ` : Queries during 1 hour intervals

These tables have the following properties:

  - Each table contains data for non-overlapping time intervals of the length the table name specifies.

  - Intervals are based on clock times. 1 minute intervals end on the minute, 10 minute intervals end every 10 minutes starting on the hour, and 1 hour intervals end on the hour.
    
    For example, at 11:59:30 AM, the most recent intervals available to SQL queries are:
    
      - **1 minute** : 11:58:00–11:58:59 AM
      - **10 minute** : 11:40:00–11:49:59 AM
      - **1 hour** : 10:00:00–10:59:59 AM

  - Spanner groups the statistics by the text of the SQL query. If a query uses [query parameters](/spanner/docs/sql-best-practices#use_query_parameters_to_speed_up_frequently_executed_queries) , Spanner groups all executions of that query into one row. If the query uses string literals, Spanner only groups the statistics if the full query text is identical; when any text differs, each query appears as a separate row. For batch DML, Spanner normalizes the batch by deduplicating consecutive identical statements prior to generating the fingerprint.

  - If a request tag is present, **FPRINT** is the hash of the request tag. Otherwise, it is the hash of the `  TEXT  ` value. For partitioned DMLs, **FPRINT** is always the hash of the `  TEXT  ` value.

  - Each row contains statistics for all executions of a particular SQL query that Spanner captures statistics for during the specified interval.

  - If Spanner is unable to store all queries run during the interval, the system prioritizes queries with the highest CPU usage during the specified interval.

  - Tracked queries include those that completed, failed, or were canceled by the user.

  - A subset of statistics is specific to queries that ran but did not complete:
    
      - Execution count and mean latency in seconds across all the queries that did not succeed.
    
      - Execution count for queries that timed out.
    
      - Execution count for queries that were canceled by the user or failed due to network connectivity issues.

  - All columns in the tables are nullable.

Query statistics for previously executed [partitioned DML](/spanner/docs/dml-partitioned) statements have the following properties:

  - Every successful partitioned DML statement strictly counts as **one** execution. A partitioned DML statement that failed, is canceled, or is executing has an execution count of zero.

  - `  ALL_FAILED_EXECUTION_COUNT  ` , `  ALL_FAILED_AVG_LATENCY_SECONDS  ` , `  CANCELLED_OR_DISCONNECTED_EXECUTION_COUNT  ` , and `  TIMED_OUT_EXECUTION_COUNT  ` statistics aren't tracked for partitioned DMLs.

  - The statistics for each previously executed partitioned DML statement might appear in different intervals. `  SPANNER_SYS.QUERY_STATS_TOP_10MINUTE  ` and `  SPANNER_SYS.QUERY_STATS_TOP_HOUR  ` provide an aggregated view for partitioned DML statements that complete within 10 minutes and 1 hour, respectively. To view statistics for statements whose duration is longer than 1 hour, see [query example](#pdml-example) .

### Table schema

<table>
<colgroup>
<col style="width: 45%" />
<col style="width: 10%" />
<col style="width: 45%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       INTERVAL_END      </code></td>
<td><code dir="ltr" translate="no">       TIMESTAMP      </code></td>
<td>End of the time interval that the included query executions occurred in.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       REQUEST_TAG      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The optional request tag for this query operation. For more information about using tags, see <a href="/spanner/docs/introspection/troubleshooting-with-tags#request_tags">Troubleshooting with request tags</a> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       QUERY_TYPE      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>Indicates if a query is a <code dir="ltr" translate="no">       PARTITIONED_QUERY      </code> or <code dir="ltr" translate="no">       QUERY      </code> . A <code dir="ltr" translate="no">       PARTITIONED_QUERY      </code> is a query with a <code dir="ltr" translate="no">       partitionToken      </code> obtained from the <a href="/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/partitionQuery"><code dir="ltr" translate="no">        PartitionQuery       </code></a> API, or a <a href="/spanner/docs/dml-partitioned">partitioned DML</a> statement. All the other queries and DML statements are denoted by the <code dir="ltr" translate="no">       QUERY      </code> query type.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       TEXT      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>SQL query text, truncated to approximately 64KB.<br />
<br />
Statistics for multiple queries that have the same tag string are grouped in a single row with the <code dir="ltr" translate="no">       REQUEST_TAG      </code> matching that tag string. Only the text of one of those queries is shown in this field, truncated to approximately 64KB. For batch DML, the set of SQL statements are flattened into a single row, concatenated using a semicolon delimiter. Consecutive identical SQL texts are deduplicated before truncating.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       TEXT_TRUNCATED      </code></td>
<td><code dir="ltr" translate="no">       BOOL      </code></td>
<td>Whether or not the query text was truncated.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       TEXT_FINGERPRINT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The hash of the <code dir="ltr" translate="no">       REQUEST_TAG      </code> value if present; Otherwise, the hash of the <code dir="ltr" translate="no">       TEXT      </code> value. Corresponds to the <code dir="ltr" translate="no">       query_fingerprint      </code> field in the audit log</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       EXECUTION_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Number of times Spanner saw the query during the interval.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_LATENCY_SECONDS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average length of time, in seconds, for each query execution within the database. This average excludes the encoding and transmission time for the result set as well as overhead.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_ROWS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of rows that the query returned.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_BYTES      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of data bytes that the query returned, excluding transmission encoding overhead.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_ROWS_SCANNED      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of rows that the query scanned, excluding deleted values.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_CPU_SECONDS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of seconds of CPU time Spanner spent on all operations to execute the query.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       ALL_FAILED_EXECUTION_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Number of times the query failed during the interval.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ALL_FAILED_AVG_LATENCY_SECONDS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average length of time, in seconds, for each query execution that failed within the database. This average excludes the encoding and transmission time for the result set as well as overhead.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       CANCELLED_OR_DISCONNECTED_EXECUTION_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Number of times the query was canceled by the user or failed due to broken network connection during the interval.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       TIMED_OUT_EXECUTION_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Number of times the query timed out during the interval.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_BYTES_WRITTEN      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of bytes written by the statement.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_ROWS_WRITTEN      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of rows modified by the statement.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       STATEMENT_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The sum of the statements aggregated into this entry. For regular queries and DML, this is equal to the execution count. For batch DML, Spanner captures the number of statements in the batch.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       RUN_IN_RW_TRANSACTION_EXECUTION_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The number of times the query was run as part of a read-write transaction. This column helps you determine if you can avoid lock contentions by moving the query to a read-only transaction.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       LATENCY_DISTRIBUTION      </code></td>
<td><code dir="ltr" translate="no">       ARRAY&lt;STRUCT&gt;      </code></td>
<td><p>A histogram of the query execution time. The values are measured in seconds.</p>
<strong>PostgreSQL interface note:</strong> PostgreSQL-dialect databases don't support this column.
<p>The array contains a single element and has the following type:<br />
<code dir="ltr" translate="no">        ARRAY&lt;STRUCT&lt;                COUNT INT64,                MEAN FLOAT64,                SUM_OF_SQUARED_DEVIATION FLOAT64,                NUM_FINITE_BUCKETS INT64,                GROWTH_FACTOR FLOAT64,                SCALE FLOAT64,                BUCKET_COUNTS ARRAY&lt;INT64&gt;&gt;&gt;       </code><br />
For more information about the values, see <a href="/monitoring/api/ref_v3/rest/v3/TypedValue#Distribution">Distribution</a> and <a href="/monitoring/api/ref_v3/rest/v3/TypedValue#exponential">Exponential</a> .</p>
<p>To calculate the percentile latency from the distribution, use the <code dir="ltr" translate="no">        SPANNER_SYS.DISTRIBUTION_PERCENTILE(distribution, n FLOAT64)       </code> function, which returns the estimated <em>n</em> th percentile. For a related example, see <a href="#example-percentile-latency">Find the 99th percentile latency for queries</a> .</p>
<p>For more information, see <a href="/monitoring/api/v3/distribution-metrics">Percentiles and distribution-valued metrics</a> .</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_MEMORY_PEAK_USAGE_BYTES      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td><p>During a distributed query execution, the average peak memory usage (in bytes).</p>
<p>Use this statistic to identify what queries or table data sizes are likely to encounter memory limits.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_MEMORY_USAGE_PERCENTAGE      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td><p>During a distributed query execution, the average memory usage required (as a percentage of the memory limit allowed for this query).</p>
<p>This statistic only tracks memory that is required for the query to execute. Some operators use additional buffering memory to improve performance. The additional buffering memory used is visible in the query plan, but isn't used to calculate <code dir="ltr" translate="no">        AVG_MEMORY_USAGE_PERCENTAGE       </code> because the buffering memory is used for optimization and isn't required.</p>
<p>Use this statistic to identify queries that are nearing the memory usage limit and are at risk of failing if the data size increases. To mitigate the risk of the query failing, see <a href="/spanner/docs/sql-best-practices">SQL best practices</a> to optimize these queries, or split the query into pieces that read less data.</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_QUERY_PLAN_CREATION_TIME_SECS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td><p>The average CPU time in seconds spent on query compilation, including query runtime creation.</p>
<p>If the value of this column is high, use <a href="/spanner/docs/samples/spanner-query-with-parameter">parameterized queries</a> .</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_FILESYSTEM_DELAY_SECS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td><p>The average time the query spends on reading from the file system or being blocked on input/output (I/O).</p>
<p>Use this statistic to identify potential high latency caused by file system I/O. To mitigate, add an <a href="/spanner/docs/secondary-indexes">index</a> or add a <a href="/spanner/docs/secondary-indexes#storing-clause"><code dir="ltr" translate="no">         STORING        </code> (GoogleSQL) or <code dir="ltr" translate="no">         INCLUDE        </code> (PostgreSQL) clause</a> to an existing index.</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_REMOTE_SERVER_CALLS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td><p>The average number of remote server calls (RPC) that have been completed by the query.</p>
<p>Use this statistic to identify if different queries that scan the same number of rows have a vastly different number of RPCs. The query with a higher RPC value might benefit from adding an <a href="/spanner/docs/secondary-indexes">index</a> or adding a <a href="/spanner/docs/secondary-indexes#storing-clause"><code dir="ltr" translate="no">         STORING        </code> (GoogleSQL) or <code dir="ltr" translate="no">         INCLUDE        </code> (PostgreSQL) clause</a> to an existing index.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_ROWS_SPOOLED      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td><p>The average number of rows written to a temporary disk (not in-memory) by the query statement.</p>
<p>Use this statistic to identify potentially high latency queries that are memory-expensive and can't be executed in-memory. To mitigate, change the <code dir="ltr" translate="no">        JOIN       </code> order, or add an <a href="/spanner/docs/secondary-indexes">index</a> that provides a required <code dir="ltr" translate="no">        SORT       </code> .</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_DISK_IO_COST      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td><p>The average cost of this query in terms of Spanner HDD <a href="/monitoring/api/metrics_gcp_p_z#spanner/instance/disk_load">disk load</a> .</p>
<p>Use this value to make relative HDD I/O cost comparisons between reads that you run in the database. Querying data on HDD storage incurs a charge against the HDD disk load capacity of the instance. A higher value indicates that you are using more HDD disk load and your query might be slower than if it was running on SSD. Furthermore, if your HDD disk load is at capacity, the performance of your queries might be further impacted. You can monitor the total <a href="/monitoring/api/metrics_gcp_p_z#spanner/instance/disk_load">HDD disk load</a> capacity of the instance as a percentage. To add more HDD disk load capacity, you can add more processing units or nodes to your instance. For more information, see <a href="/spanner/docs/create-manage-instances#change-compute-capacity">Change the compute capacity</a> . To improve query performance, also consider moving some data to SSD.</p>
<p>For workloads that consume a lot of disk I/O, we recommend that you store frequently accessed data on SSD storage. Data accessed from SSD doesn't consume HDD disk load capacity. You can store selective tables, columns, or secondary indexes on SSD storage as needed, while keeping infrequently accessed data on HDD storage. For more information, see <a href="/spanner/docs/tiered-storage">Tiered storage overview</a> .</p></td>
</tr>
</tbody>
</table>

`  EXECUTION_COUNT  ` , `  AVG_LATENCY_SECONDS  ` , and `  LATENCY_DISTRIBUTION  ` for failed queries include queries that failed due to incorrect syntax or encountered a transient error but succeeded on retrying. These statistics don't track failed and cancelled partitioned DML statements.

## Aggregate statistics

There are also tables that track aggregate data for all the queries for which Spanner captured statistics in a specific time period:

  - `  SPANNER_SYS.QUERY_STATS_TOTAL_MINUTE  ` : Queries during 1 minute intervals
  - `  SPANNER_SYS.QUERY_STATS_TOTAL_10MINUTE  ` : Queries during 10 minute intervals
  - `  SPANNER_SYS.QUERY_STATS_TOTAL_HOUR  ` : Queries during 1 hour intervals

These tables have the following properties:

  - Each table contains data for non-overlapping time intervals of the length the table name specifies.

  - Intervals are based on clock times. 1 minute intervals end on the minute, 10 minute intervals end every 10 minutes starting on the hour, and 1 hour intervals end on the hour.
    
    For example, at 11:59:30 AM, the most recent intervals available to SQL queries are:
    
      - **1 minute** : 11:58:00–11:58:59 AM
      - **10 minute** : 11:40:00–11:49:59 AM
      - **1 hour** : 10:00:00–10:59:59 AM

  - Each row contains statistics for **all** queries executed over the database during the specified interval, aggregated together. There is only one row per time interval and it includes completed queries, failed queries, and queries canceled by the user.

  - The statistics captured in the `  TOTAL  ` tables might include queries that Spanner did not capture in the `  TOP  ` tables.

  - Some columns in these tables are exposed as metrics in Cloud Monitoring. The exposed metrics are:
    
      - Query execution count
      - Query failures
      - Query latencies
      - Rows returned count
      - Rows scanned count
      - Bytes returned count
      - Query CPU time
    
    For more information, see [Spanner metrics](/monitoring/api/metrics_gcp_p_z#gcp-spanner) .

### Table schema

<table>
<colgroup>
<col style="width: 45%" />
<col style="width: 10%" />
<col style="width: 45%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       INTERVAL_END      </code></td>
<td><code dir="ltr" translate="no">       TIMESTAMP      </code></td>
<td>End of the time interval that the included query executions occurred in.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       EXECUTION_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Number of times Spanner saw the query during the interval of time.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_LATENCY_SECONDS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average length of time, in seconds, for each query execution within the database. This average excludes the encoding and transmission time for the result set as well as overhead.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_ROWS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of rows that the query returned.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_BYTES      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of data bytes that the query returned, excluding transmission encoding overhead.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_ROWS_SCANNED      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of rows that the query scanned, excluding deleted values.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_CPU_SECONDS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of seconds of CPU time Spanner spent on all operations to execute the query.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ALL_FAILED_EXECUTION_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Number of times the query failed during the interval.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       ALL_FAILED_AVG_LATENCY_SECONDS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average length of time, in seconds, for each query execution that failed within the database. This average excludes the encoding and transmission time for the result set as well as overhead.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       CANCELLED_OR_DISCONNECTED_EXECUTION_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Number of times the query was canceled by the user or failed due to broken network connection during the interval.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       TIMED_OUT_EXECUTION_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Number of times the query timed out during the interval.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_BYTES_WRITTEN      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of bytes written by the statement.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_ROWS_WRITTEN      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of rows modified by the statement.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       RUN_IN_RW_TRANSACTION_EXECUTION_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The number of times queries were run as part of read-write transactions. This column helps you determine if you can avoid lock contentions by moving some queries to read-only transactions.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       LATENCY_DISTRIBUTION      </code></td>
<td><code dir="ltr" translate="no">       ARRAY&lt;STRUCT&gt;      </code></td>
<td><p>A histogram of the execution time across queries. The values are measured in seconds.</p>
<strong>PostgreSQL interface note:</strong> PostgreSQL-dialect databases don't support this column.
<p>Specify the array as follows:<br />
<code dir="ltr" translate="no">        ARRAY&lt;STRUCT&lt;                COUNT INT64,                MEAN FLOAT64,                SUM_OF_SQUARED_DEVIATION FLOAT64,                NUM_FINITE_BUCKETS INT64,                GROWTH_FACTOR FLOAT64,                SCALE FLOAT64,                BUCKET_COUNTS ARRAY&lt;INT64&gt;&gt;&gt;       </code><br />
For more information about the values, see <a href="/monitoring/api/ref_v3/rest/v3/TypedValue#Distribution">Distribution</a> and <a href="/monitoring/api/ref_v3/rest/v3/TypedValue#exponential">Exponential</a> .</p>
<p>To calculate the percentile latency from the distribution, use the <code dir="ltr" translate="no">        SPANNER_SYS.DISTRIBUTION_PERCENTILE(distribution, n FLOAT64)       </code> function, which returns the estimated <em>n</em> th percentile. For a related example, see <a href="#example-percentile-latency">Find the 99th percentile latency for queries</a> .</p>
<p>For more information, see <a href="/monitoring/api/v3/distribution-metrics">Percentiles and distribution-valued metrics</a> .</p></td>
</tr>
</tbody>
</table>

## Data retention

At a minimum, Spanner keeps data for each table for the following time periods:

  - `  SPANNER_SYS.QUERY_STATS_TOP_MINUTE  ` and `  SPANNER_SYS.QUERY_STATS_TOTAL_MINUTE  ` : Intervals covering the previous 6 hours.

  - `  SPANNER_SYS.QUERY_STATS_TOP_10MINUTE  ` and `  SPANNER_SYS.QUERY_STATS_TOTAL_10MINUTE  ` : Intervals covering the previous 4 days.

  - `  SPANNER_SYS.QUERY_STATS_TOP_HOUR  ` and `  SPANNER_SYS.QUERY_STATS_TOTAL_HOUR  ` : Intervals covering the previous 30 days.

**Note:** You cannot prevent Spanner from collecting query statistics. To delete the data in these tables, you must delete the database associated with the tables or wait until Spanner removes the data automatically.

## Example queries

This section includes several example SQL statements that retrieve query statistics. You can run these SQL statements using the [client libraries](/spanner/docs/reference/libraries) , the [Google Cloud CLI](/spanner/docs/gcloud-spanner#execute_sql_statements) , or the [Google Cloud console](/spanner/docs/create-query-database-console#run_a_query) .

### List the basic statistics for each query in a given time period

The following query returns the raw data for the top queries in the previous minute:

``` text
SELECT text,
       request_tag,
       interval_end,
       execution_count,
       avg_latency_seconds,
       avg_rows,
       avg_bytes,
       avg_rows_scanned,
       avg_cpu_seconds
FROM spanner_sys.query_stats_top_minute
ORDER BY interval_end DESC;
```

### List the statistics for partitioned DML statements that run more than one hour

The following query returns the execution count and average rows written by the top partitioned DML queries in the previous hours:

``` text
SELECT text,
       request_tag,
       interval_end,
       sum(execution_count) as execution_count
       sum(avg_rows_written*execution_count)/sum(execution_count) as avg_rows_written
FROM spanner_sys.query_stats_top_hour
WHERE starts_with(text, "UPDATE") AND query_type = "PARTITIONED_QUERY"
group by text, request_tag, interval_end
ORDER BY interval_end DESC;
```

### List the queries with the highest CPU usage

The following query returns the queries with the highest CPU usage in the previous hour:

``` text
SELECT text,
       request_tag,
       execution_count AS count,
       avg_latency_seconds AS latency,
       avg_cpu_seconds AS cpu,
       execution_count * avg_cpu_seconds AS total_cpu
FROM spanner_sys.query_stats_top_hour
WHERE interval_end =
  (SELECT MAX(interval_end)
   FROM spanner_sys.query_stats_top_hour)
ORDER BY total_cpu DESC;
```

### Find the total execution count in a given time period

The following query returns the total number of queries executed in the most recent complete 1-minute interval:

``` text
SELECT interval_end,
       execution_count
FROM spanner_sys.query_stats_total_minute
WHERE interval_end =
  (SELECT MAX(interval_end)
   FROM spanner_sys.query_stats_top_minute);
```

### Find the average latency for a query

The following query returns the average latency information for a specific query:

``` text
SELECT avg_latency_seconds
FROM spanner_sys.query_stats_top_hour
WHERE text LIKE "SELECT x FROM table WHERE x=@foo;";
```

### Find the 99th percentile latency for queries

The following query returns the 99th percentile of execution time across queries ran in the previous 10 minutes:

``` text
SELECT interval_end, avg_latency_seconds, SPANNER_SYS.DISTRIBUTION_PERCENTILE(latency_distribution[OFFSET(0)], 99.0)
  AS percentile_latency
FROM spanner_sys.query_stats_total_10minute
WHERE interval_end =
  (SELECT MAX(interval_end)
   FROM spanner_sys.query_stats_total_10minute)
ORDER BY interval_end;
```

Comparing the Average latency with the 99th percentile latency helps identify possible outlier queries with high execution times.

### Find the queries that scan the most data

You can use the number of rows scanned by a query as a measure of the amount of data that the query scanned. The following query returns the number of rows scanned by queries executed in the previous hour:

``` text
SELECT text,
       execution_count,
       avg_rows_scanned
FROM spanner_sys.query_stats_top_hour
WHERE interval_end =
  (SELECT MAX(interval_end)
   FROM spanner_sys.query_stats_top_hour)
ORDER BY avg_rows_scanned DESC;
```

### Find the statements that wrote the most data

You can use the number of rows written (or bytes written) by DML as a measure of the amount of data that the query modified. The following query returns the number of rows written by DML statements executed in the previous hour:

``` text
SELECT text,
       execution_count,
       avg_rows_written
FROM spanner_sys.query_stats_top_hour
WHERE interval_end =
  (SELECT MAX(interval_end)
   FROM spanner_sys.query_stats_top_hour)
ORDER BY avg_rows_written DESC;
```

### Total the CPU usage across all queries

The following query returns the number of CPU hours used in the previous hour:

``` text
SELECT (avg_cpu_seconds * execution_count / 60 / 60)
  AS total_cpu_hours
FROM spanner_sys.query_stats_total_hour
WHERE interval_end =
  (SELECT MAX(interval_end)
   FROM spanner_sys.query_stats_total_hour);
```

### List the queries that have failed in a given time period

The following query returns the raw data including execution count and average latency of failed queries for the top queries in the previous minute. These statistics don't track failed and cancelled partitioned DML statements.

``` text
SELECT text,
       request_tag,
       interval_end,
       execution_count,
       all_failed_execution_count,
       all_failed_avg_latency_seconds,
       avg_latency_seconds,
       avg_rows,
       avg_bytes,
       avg_rows_scanned,
       avg_cpu_seconds
FROM spanner_sys.query_stats_top_minute
WHERE all_failed_execution_count > 0
ORDER BY interval_end;
```

### Find the total error count in a given time period

The following query returns the total number of queries that failed to execute in the most recent complete 1 minute interval. These statistics don't track failed and cancelled partitioned DML statements.

``` text
SELECT interval_end,
       all_failed_execution_count
FROM spanner_sys.query_stats_total_minute
WHERE interval_end =
  (SELECT MAX(interval_end)
   FROM spanner_sys.query_stats_top_minute)
ORDER BY interval_end;
```

### List the queries that timeout the most

The following query returns the queries with the highest timeout count in the previous hour.

``` text
SELECT text,
       execution_count AS count,
       timed_out_execution_count AS timeout_count,
       avg_latency_seconds AS latency,
       avg_cpu_seconds AS cpu,
       execution_count * avg_cpu_seconds AS total_cpu
FROM spanner_sys.query_stats_top_hour
WHERE interval_end =
  (SELECT MAX(interval_end)
   FROM spanner_sys.query_stats_top_hour)
ORDER BY timed_out_execution_count DESC;
```

### Find the average latency of successful and failed executions for a query

The following query returns the combined average latency, average latency for successful executions, and average latency for failed executions for a specific query. These statistics don't track failed and cancelled partitioned DML statements.

``` text
SELECT avg_latency_seconds AS combined_avg_latency,
       all_failed_avg_latency_seconds AS failed_execution_latency,
       ( avg_latency_seconds * execution_count -
         all_failed_avg_latency_seconds * all_failed_execution_count
       ) / (
       execution_count - all_failed_execution_count ) AS success_execution_latency
FROM   spanner_sys.query_stats_top_hour
WHERE  text LIKE "select x from table where x=@foo;";
```

## Troubleshoot high CPU usage or elevated query latency with query statistics

Query statistics are useful when you need to investigate high CPU usage on your Spanner database or when you are just trying to understand the CPU- heavy query shapes on your database. Inspecting queries that use significant amounts of database resources gives Spanner users a potential way to reduce operational costs and possibly improve general system latencies.

You can use SQL code or the [Query insights](/spanner/docs/using-query-insights#the_dashboard) dashboard to investigate problematic queries in your database. The following topics show how you can investigate such queries by using SQL code.

While the following example focuses on CPU usage, similar steps can be followed to troubleshoot elevated query latency and find the queries with the highest latencies. Simply select time intervals and queries by latency instead of CPU usage.

### Select a time period to investigate

Start your investigation by looking for a time when your application began to experience high CPU usage. For example, if the issue started occurring around **5:00 pm on July 24, 2020 UTC** .

### Gather query statistics for the selected time period

Having selected a time period to start our investigation, we'll look at statistics gathered in the `  QUERY_STATS_TOTAL_10MINUTE  ` table around that time. The results of this query might indicate how CPU and other query statistics changed over that period of time.

The following query returns the aggregated query statistics from **16:30** to **17:30** UTC inclusive. We are using [`  ROUND  `](/spanner/docs/reference/standard-sql/mathematical_functions#round) in our query to restrict the number of decimal places for display purposes.

``` text
SELECT interval_end,
       execution_count AS count,
       ROUND(avg_latency_seconds,2) AS latency,
       ROUND(avg_rows,2) AS rows_returned,
       ROUND(avg_bytes,2) AS bytes,
       ROUND(avg_rows_scanned,2) AS rows_scanned,
       ROUND(avg_cpu_seconds,3) AS avg_cpu
FROM spanner_sys.query_stats_total_10minute
WHERE
  interval_end >= "2020-07-24T16:30:00Z"
  AND interval_end <= "2020-07-24T17:30:00Z"
ORDER BY interval_end;
```

Running the query produced the following results.

<table>
<thead>
<tr class="header">
<th style="text-align: left;">interval_end</th>
<th style="text-align: right;">count</th>
<th style="text-align: right;">latency</th>
<th style="text-align: right;">rows_returned</th>
<th style="text-align: right;">bytes</th>
<th style="text-align: right;">rows_scanned</th>
<th style="text-align: right;">avg_cpu</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">2020-07-24T16:30:00Z</td>
<td style="text-align: right;">6</td>
<td style="text-align: right;">0.06</td>
<td style="text-align: right;">5.00</td>
<td style="text-align: right;">536.00</td>
<td style="text-align: right;">16.67</td>
<td style="text-align: right;">0.035</td>
</tr>
<tr class="even">
<td style="text-align: left;">2020-07-24T16:40:00Z</td>
<td style="text-align: right;">55</td>
<td style="text-align: right;">0.02</td>
<td style="text-align: right;">0.22</td>
<td style="text-align: right;">25.29</td>
<td style="text-align: right;">0.22</td>
<td style="text-align: right;">0.004</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2020-07-24T16:50:00Z</td>
<td style="text-align: right;">102</td>
<td style="text-align: right;">0.02</td>
<td style="text-align: right;">0.30</td>
<td style="text-align: right;">33.35</td>
<td style="text-align: right;">0.30</td>
<td style="text-align: right;">0.004</td>
</tr>
<tr class="even">
<td style="text-align: left;"><code dir="ltr" translate="no">       2020-07-24T17:00:00Z      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       154      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       1.06      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       4.42      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       486.33      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       7792208.12      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       4.633      </code></td>
</tr>
<tr class="odd">
<td style="text-align: left;">2020-07-24T17:10:00Z</td>
<td style="text-align: right;">94</td>
<td style="text-align: right;">0.02</td>
<td style="text-align: right;">1.68</td>
<td style="text-align: right;">106.84</td>
<td style="text-align: right;">1.68</td>
<td style="text-align: right;">0.006</td>
</tr>
<tr class="even">
<td style="text-align: left;">2020-07-24T17:20:00Z</td>
<td style="text-align: right;">110</td>
<td style="text-align: right;">0.02</td>
<td style="text-align: right;">0.38</td>
<td style="text-align: right;">34.60</td>
<td style="text-align: right;">0.38</td>
<td style="text-align: right;">0.005</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2020-07-24T17:30:00Z</td>
<td style="text-align: right;">47</td>
<td style="text-align: right;">0.02</td>
<td style="text-align: right;">0.23</td>
<td style="text-align: right;">24.96</td>
<td style="text-align: right;">0.23</td>
<td style="text-align: right;">0.004</td>
</tr>
</tbody>
</table>

In the preceding table we see that average CPU time, the **avg\_cpu** column in the results table, is highest in the highlighted intervals ending at 17:00. We also see a much higher number of rows scanned on average. This indicates that more expensive queries ran between 16:50 and 17:00. Choose that interval to investigate further in the next step.

### Find the queries that are causing high CPU usage

With a time interval to investigate selected, we now query the `  QUERY_STATS_TOP_10MINUTE  ` table. The results of this query can help indicate which queries cause high CPU usage.

``` text
SELECT text_fingerprint AS fingerprint,
       execution_count AS count,
       ROUND(avg_latency_seconds,2) AS latency,
       ROUND(avg_cpu_seconds,3) AS cpu,
       ROUND(execution_count * avg_cpu_seconds,3) AS total_cpu
FROM spanner_sys.query_stats_top_10MINUTE
WHERE
  interval_end = "2020-07-24T17:00:00Z"
ORDER BY total_cpu DESC;
```

Running this query yields the following results.

<table>
<thead>
<tr class="header">
<th style="text-align: center;">fingerprint</th>
<th style="text-align: right;">count</th>
<th style="text-align: right;">latency</th>
<th style="text-align: right;">cpu</th>
<th style="text-align: right;">total_cpu</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;"><code dir="ltr" translate="no">       5505124206529314852      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       30      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       3.88      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       17.635      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       529.039      </code></td>
</tr>
<tr class="even">
<td style="text-align: center;"><code dir="ltr" translate="no">       1697951036096498470      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       10      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       4.49      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       18.388      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       183.882      </code></td>
</tr>
<tr class="odd">
<td style="text-align: center;">2295109096748351518</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.33</td>
<td style="text-align: right;">0.048</td>
<td style="text-align: right;">0.048</td>
</tr>
<tr class="even">
<td style="text-align: center;">11618299167612903606</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.25</td>
<td style="text-align: right;">0.021</td>
<td style="text-align: right;">0.021</td>
</tr>
<tr class="odd">
<td style="text-align: center;">10302798842433860499</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.04</td>
<td style="text-align: right;">0.006</td>
<td style="text-align: right;">0.006</td>
</tr>
<tr class="even">
<td style="text-align: center;">123771704548746223</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.04</td>
<td style="text-align: right;">0.006</td>
<td style="text-align: right;">0.006</td>
</tr>
<tr class="odd">
<td style="text-align: center;">4216063638051261350</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.04</td>
<td style="text-align: right;">0.006</td>
<td style="text-align: right;">0.006</td>
</tr>
<tr class="even">
<td style="text-align: center;">3654744714919476398</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.04</td>
<td style="text-align: right;">0.006</td>
<td style="text-align: right;">0.006</td>
</tr>
<tr class="odd">
<td style="text-align: center;">2999453161628434990</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.04</td>
<td style="text-align: right;">0.006</td>
<td style="text-align: right;">0.006</td>
</tr>
<tr class="even">
<td style="text-align: center;">823179738756093706</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0.02</td>
<td style="text-align: right;">0.005</td>
<td style="text-align: right;">0.0056</td>
</tr>
</tbody>
</table>

The top 2 queries, highlighted in the results table, are outliers in terms of average CPU and latency, as well as number of executions and total CPU. Investigate the first query listed in these results.

### Compare query runs over time

Having narrowed down the investigation, we can turn our attention to the `  QUERY_STATS_TOP_MINUTE  ` table. By comparing runs over time for a particular query, we can look for correlations between the number of rows or bytes returned, or the number of rows scanned and elevated CPU or latency. A deviation may indicate non-uniformity in the data. Consistently high numbers of rows scanned may indicate the lack of appropriate indexes or sub-optimal join ordering.

Investigate the query exhibiting highest average CPU usage and highest latency by running the following statement which filters on the text\_fingerprint of that query.

``` text
SELECT interval_end,
       ROUND(avg_latency_seconds,2) AS latency,
       avg_rows AS rows_returned,
       avg_bytes AS bytes_returned,
       avg_rows_scanned AS rows_scanned,
       ROUND(avg_cpu_seconds,3) AS cpu,
FROM spanner_sys.query_stats_top_minute
WHERE text_fingerprint = 5505124206529314852
ORDER BY interval_end DESC;
```

Running this query returns the following results.

<table>
<thead>
<tr class="header">
<th style="text-align: left;">interval_end</th>
<th style="text-align: right;">latency</th>
<th style="text-align: right;">rows_returned</th>
<th style="text-align: right;">bytes_returned</th>
<th style="text-align: right;">rows_scanned</th>
<th style="text-align: right;">cpu</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">2020-07-24T17:00:00Z</td>
<td style="text-align: right;">4.55</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">2365</td>
<td style="text-align: right;">30000000</td>
<td style="text-align: right;">19.255</td>
</tr>
<tr class="even">
<td style="text-align: left;">2020-07-24T16:00:00Z</td>
<td style="text-align: right;">3.62</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">2365</td>
<td style="text-align: right;">30000000</td>
<td style="text-align: right;">17.255</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2020-07-24T15:00:00Z</td>
<td style="text-align: right;">4.37</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">2365</td>
<td style="text-align: right;">30000000</td>
<td style="text-align: right;">18.350</td>
</tr>
<tr class="even">
<td style="text-align: left;">2020-07-24T14:00:00Z</td>
<td style="text-align: right;">4.02</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">2365</td>
<td style="text-align: right;">30000000</td>
<td style="text-align: right;">17.748</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2020-07-24T13:00:00Z</td>
<td style="text-align: right;">3.12</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">2365</td>
<td style="text-align: right;">30000000</td>
<td style="text-align: right;">16.380</td>
</tr>
<tr class="even">
<td style="text-align: left;">2020-07-24T12:00:00Z</td>
<td style="text-align: right;">3.45</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">2365</td>
<td style="text-align: right;">30000000</td>
<td style="text-align: right;">15.476</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2020-07-24T11:00:00Z</td>
<td style="text-align: right;">4.94</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">2365</td>
<td style="text-align: right;">30000000</td>
<td style="text-align: right;">22.611</td>
</tr>
<tr class="even">
<td style="text-align: left;">2020-07-24T10:00:00Z</td>
<td style="text-align: right;">6.48</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">2365</td>
<td style="text-align: right;">30000000</td>
<td style="text-align: right;">21.265</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2020-07-24T09:00:00Z</td>
<td style="text-align: right;">0.23</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">2365</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">0.040</td>
</tr>
<tr class="even">
<td style="text-align: left;">2020-07-24T08:00:00Z</td>
<td style="text-align: right;">0.04</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">2365</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">0.021</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2020-07-24T07:00:00Z</td>
<td style="text-align: right;">0.09</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">2365</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">0.030</td>
</tr>
</tbody>
</table>

Examining the preceding results, we see that the number of rows scanned, CPU used, and latency all changed significantly around 9:00 am. To understand why these numbers increased so dramatically, we'll examine the query text and see whether any changes in the schema might have impacted the query.

Use the following query to retrieve the query text for the query we are investigating.

``` text
SELECT text,
       text_truncated
FROM spanner_sys.query_stats_top_hour
WHERE text_fingerprint = 5505124206529314852
LIMIT 1;
```

This returns the following result.

<table>
<thead>
<tr class="header">
<th>text</th>
<th>text_truncated</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>select * from orders where o_custkey = 36901;</td>
<td>false</td>
</tr>
</tbody>
</table>

Examining the query text which is returned, we realize that the query is filtering on a field called `  o_custkey  ` . This is a non-key column on the `  orders  ` table. As it happens, there used to be an index on that column that was dropped around 9 am. This explains the change in cost for this query. We can add the index back in or, if the query is infrequently run, decide to not have the index and accept the higher read cost.

Our investigation focused so far on queries that completed successfully and we found one reason why the database was experiencing some performance degradation. In the next step, we'll focus on failed or canceled queries and show how to examine that data for more insights.

### Investigate failed queries

Queries that don't complete successfully still consume resources before they time out, are cancelled, or otherwise fail. Spanner tracks the execution count and resources consumed by failed queries along with successful ones. These statistics don't track failed and cancelled partitioned DML statements.

To check whether failed queries are a significant contributor to system utilization we can first check how many queries failed in the time interval of interest.

``` text
SELECT interval_end,
       all_failed_execution_count AS failed_count,
       all_failed_avg_latency_seconds AS latency
FROM spanner_sys.query_stats_total_minute
WHERE
  interval_end >= "2020-07-24T16:50:00Z"
  AND interval_end <= "2020-07-24T17:00:00Z"
ORDER BY interval_end;
```

<table>
<thead>
<tr class="header">
<th>interval_end</th>
<th>failed_count</th>
<th>latency</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>2020-07-24T16:52:00Z</td>
<td>1</td>
<td>15.211391</td>
</tr>
<tr class="even">
<td>2020-07-24T16:53:00Z</td>
<td>3</td>
<td>58.312232</td>
</tr>
</tbody>
</table>

Investigating further, we can look for queries that are most likely to fail using the following query.

``` text
SELECT interval_end,
       text_fingerprint,
       execution_count,
       avg_latency_seconds AS avg_latency,
       all_failed_execution_count AS failed_count,
       all_failed_avg_latency_seconds AS failed_latency,
       cancelled_or_disconnected_execution_count AS cancel_count,
       timed_out_execution_count AS to_count
FROM spanner_sys.query_stats_top_minute
WHERE all_failed_execution_count > 0
ORDER BY interval_end;
```

<table>
<thead>
<tr class="header">
<th style="text-align: center;">interval_end</th>
<th style="text-align: center;">text_fingerprint</th>
<th style="text-align: right;">execution_count</th>
<th style="text-align: right;">failed_count</th>
<th style="text-align: right;">cancel_count</th>
<th style="text-align: right;">to_count</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">2020-07-24T16:52:00Z</td>
<td style="text-align: center;">5505124206529314852</td>
<td style="text-align: right;">3</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="even">
<td style="text-align: center;">2020-07-24T16:53:00Z</td>
<td style="text-align: center;">1697951036096498470</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">0</td>
</tr>
<tr class="odd">
<td style="text-align: center;">2020-07-24T16:53:00Z</td>
<td style="text-align: center;">5505124206529314852</td>
<td style="text-align: right;">5</td>
<td style="text-align: right;">2</td>
<td style="text-align: right;">1</td>
<td style="text-align: right;">1</td>
</tr>
</tbody>
</table>

As the preceding table shows, the query with fingerprint `  5505124206529314852  ` has failed multiple times during different time intervals. Given a pattern of failures such as this, it is then interesting to compare latency of successful and unsuccessful runs.

``` text
SELECT interval_end,
       avg_latency_seconds AS combined_avg_latency,
       all_failed_avg_latency_seconds AS failed_execution_latency,
       ( avg_latency_seconds * execution_count -
         all_failed_avg_latency_seconds * all_failed_execution_count
       ) / (
       execution_count - all_failed_execution_count ) AS success_execution_latency
FROM   spanner_sys.query_stats_top_hour
WHERE  text_fingerprint = 5505124206529314852;
```

<table>
<thead>
<tr class="header">
<th style="text-align: center;">interval_end</th>
<th style="text-align: right;">combined_avg_latency</th>
<th style="text-align: right;">failed_execution_latency</th>
<th style="text-align: right;">success_execution_latency</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">2020-07-24T17:00:00Z</td>
<td style="text-align: right;">3.880420</td>
<td style="text-align: right;">13.830709</td>
<td style="text-align: right;">2.774832</td>
</tr>
</tbody>
</table>

### Apply best practices

Having identified a candidate query for optimization, we can next look at the query profile and try to optimize using [SQL best practices](/spanner/docs/sql-best-practices) .

## What's next

  - Use [Oldest active queries](/spanner/docs/introspection/oldest-active-queries) to determine the longest running active queries.

  - Learn more about [Investigating high CPU utilization](/spanner/docs/introspection/investigate-cpu-utilization) .

  - Learn about other [Introspection tools](/spanner/docs/introspection) .

  - Learn about other information Spanner stores for each database in the database's [information schema](/spanner/docs/information-schema) tables.

  - Learn more about [SQL best practices](/spanner/docs/sql-best-practices) for Spanner.
