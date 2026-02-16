Spanner provides built-in tables that store statistics about transactions. You can retrieve statistics from these `  SPANNER_SYS.TXN_STATS*  ` tables using SQL statements.

## When to use transaction statistics

Transaction statistics are useful when investigating performance issues. For example, you can check whether there are any slow running transactions that might be impacting performance or Queries Per Second (QPS) in your database. Another scenario is when your client applications are experiencing high transaction execution latency. Analyzing transaction statistics may help to discover potential bottlenecks, such as large volumes of updates to a particular column, which might be impacting latency.

## Access transaction statistics

**Note:** Spanner Studio (formerly labeled **Query** in the Google Cloud console) supports SQL, DML, and DDL operations in a single editor. For more information, see [Manage your data using the Google Cloud console](/spanner/docs/manage-data-using-console) .

Spanner provides the table transaction statistics in the `  SPANNER_SYS  ` schema. You can use the following ways to access `  SPANNER_SYS  ` data:

  - A database's Spanner Studio page in the Google Cloud console.

  - The `  gcloud spanner databases execute-sql  ` command.

  - The [Transaction insights](/spanner/docs/use-lock-and-transaction-insights#txn-insights) dashboard.

  - The [`  executeSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql) or the [`  executeStreamingSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql) method.

The following single read methods that Spanner provides don't support `  SPANNER_SYS  ` :

  - Performing a strong read from a single row or multiple rows in a table.
  - Performing a stale read from a single row or multiple rows in a table.
  - Reading from a single row or multiple rows in a secondary index.

## Latency statistics grouped by transaction

The following tables track the statistics for `  TOP  ` resource consuming transactions during a specific time period.

  - `  SPANNER_SYS.TXN_STATS_TOP_MINUTE  ` : Transaction statistics aggregated across 1 minute intervals.

  - `  SPANNER_SYS.TXN_STATS_TOP_10MINUTE  ` : Transaction statistics aggregated across 10 minute intervals.

  - `  SPANNER_SYS.TXN_STATS_TOP_HOUR  ` : Transaction statistics aggregated across 1 hour intervals.

These tables have the following properties:

  - Each table contains data for non-overlapping time intervals of the length the table name specifies.

  - Intervals are based on clock times. 1 minute intervals end on the minute, 10 minute intervals end every 10 minutes starting on the hour, and 1 hour intervals end on the hour.
    
    For example, at 11:59:30 AM, the most recent intervals available to SQL queries are:
    
      - **1 minute** : 11:58:00–11:58:59 AM
      - **10 minute** : 11:40:00–11:49:59 AM
      - **1 hour** : 10:00:00–10:59:59 AM

  - Spanner groups the statistics by **FPRINT** (Fingerprint) of the transactions. If a transaction tag is present, **FPRINT** is the hash of the tag. Otherwise, it is the hash calculated based on the operations involved in the transaction.

  - Since statistics are grouped based on **FPRINT** , if the same transaction is executed multiple times within any time interval, we still see only one entry for that transaction in these tables.

  - Each row contains statistics for all executions of a particular transaction that Spanner captures statistics for during the specified interval.

  - If Spanner is unable to store statistics for all transactions run during the interval in these tables, the system prioritizes transactions with the highest latency, commit attempts, and bytes written during the specified interval.

  - All columns in the tables are nullable.

### Table schema

<table>
<colgroup>
<col style="width: 35%" />
<col style="width: 10%" />
<col style="width: 55%" />
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
<td>End of the time interval in which the included transaction executions occurred.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       TRANSACTION_TAG      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The optional transaction tag for this transaction operation. For more information about using tags, see <a href="/spanner/docs/introspection/troubleshooting-with-tags#transaction_tags">Troubleshooting with transaction tags</a> . Statistics for multiple transactions that have the same tag string are grouped in a single row with the `TRANSACTION_TAG` matching that tag string.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       FPRINT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The hash of the <code dir="ltr" translate="no">       TRANSACTION_TAG      </code> if present; Otherwise, the hash is calculated based on the operations involved in the transaction. <code dir="ltr" translate="no">       INTERVAL_END      </code> and <code dir="ltr" translate="no">       FPRINT      </code> together act as an unique key for these tables.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       READ_COLUMNS      </code></td>
<td><code dir="ltr" translate="no">       ARRAY&lt;STRING&gt;      </code></td>
<td>The set of columns that were read by the transaction.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       WRITE_CONSTRUCTIVE_COLUMNS      </code></td>
<td><code dir="ltr" translate="no">       ARRAY&lt;STRING&gt;      </code></td>
<td>The set of columns that were constructively written (i.e. assigned to new values) by the transaction.<br />
<br />
For change streams, if the transaction involved writes to columns and tables watched by a change stream, <code dir="ltr" translate="no">       WRITE_CONSTRUCTIVE_COLUMNS      </code> will contain two columns - <code dir="ltr" translate="no">       .data      </code> and <code dir="ltr" translate="no">       ._exists      </code> <sup>1</sup> , prefixed with a change stream name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       WRITE_DELETE_TABLES      </code></td>
<td><code dir="ltr" translate="no">       ARRAY&lt;STRING&gt;      </code></td>
<td>The set of tables that had rows deleted or replaced by the transaction.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       ATTEMPT_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Total number of times that the transaction is attempted, including the attempts that abort before calling `commit`.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       COMMIT_ATTEMPT_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Total number of transaction commit attempts. This must match the number of calls to the transaction's <code dir="ltr" translate="no">       commit      </code> method.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       COMMIT_ABORT_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Total number of transaction attempts that were aborted, including those that were aborted before calling the transaction's <code dir="ltr" translate="no">       commit      </code> method.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       COMMIT_RETRY_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Total number of attempts that are retries from previously aborted attempts. A Spanner transaction might be tried multiple times before it commits due to lock contentions or transient events. A high number of retries relative to commit attempts indicates that there might be issues worth investigating. For more information, see <a href="#commit-counts">Understanding transactions and commit counts</a> on this page.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       COMMIT_FAILED_PRECONDITION_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Total number of transaction commit attempts that returned failed precondition errors, such as <code dir="ltr" translate="no">       UNIQUE      </code> index violations, row already exists, row not found, and so on.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       SERIALIZABLE_PESSIMISTIC_TXN_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Total number of transaction attempts initiated with the <a href="/spanner/docs/isolation-levels#serializable"><code dir="ltr" translate="no">        SERIALIZABLE       </code> isolation level</a> and <code dir="ltr" translate="no">       PESSIMISTIC      </code> locking. It's a subset of <code dir="ltr" translate="no">       ATTEMPT_COUNT      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       REPEATABLE_READ_OPTIMISTIC_TXN_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Total number of transaction attempts initiated with the <a href="/spanner/docs/isolation-levels#repeatable-read"><code dir="ltr" translate="no">        REPEATABLE READ       </code> isolation level</a> and <code dir="ltr" translate="no">       OPTIMISTIC      </code> locking. It's a subset of <code dir="ltr" translate="no">       ATTEMPT_COUNT      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_PARTICIPANTS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of participants in each commit attempt. To learn more about participants, see <a href="/spanner/docs/whitepapers/life-of-reads-and-writes">Life of Spanner Reads &amp; Writes</a> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_TOTAL_LATENCY_SECONDS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average seconds taken from the first operation of the transaction to commit/abort.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_COMMIT_LATENCY_SECONDS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average seconds taken to perform the commit operation.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_BYTES      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of bytes written by the transaction.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       TOTAL_LATENCY_DISTRIBUTION      </code></td>
<td><code dir="ltr" translate="no">       ARRAY&lt;STRUCT&gt;      </code></td>
<td><p>A histogram of the total commit latency, which is the time from the first transactional operation start time to the commit or abort time, for all attempts of a transaction.</p>
<strong>PostgreSQL interface note:</strong> PostgreSQL-dialect databases do not support this column.
<p>If a transaction aborts multiple times and then successfully commits, latency is measured for each attempt until the final successful commit. The values are measured in seconds.</p>
<p>The array contains a single element and has the following type:<br />
<code dir="ltr" translate="no">        ARRAY&lt;STRUCT&lt;                COUNT INT64,                MEAN FLOAT64,                SUM_OF_SQUARED_DEVIATION FLOAT64,                NUM_FINITE_BUCKETS INT64,                GROWTH_FACTOR FLOAT64,                SCALE FLOAT64,                BUCKET_COUNTS ARRAY&lt;INT64&gt;&gt;&gt;       </code><br />
For more information about the values, see <a href="/monitoring/api/ref_v3/rest/v3/TypedValue#Distribution">Distribution</a> and <a href="/monitoring/api/ref_v3/rest/v3/TypedValue#exponential">Exponential</a> .</p>
<p>To calculate the desired percentile latency from the distribution, use the <code dir="ltr" translate="no">        SPANNER_SYS.DISTRIBUTION_PERCENTILE(distribution, n FLOAT64)       </code> function, which returns the estimated <em>n</em> th percentile. For a related example, see <a href="#example-percentile-latency">Find the 99th percentile latency for transactions</a> .</p>
<p>For more information, see <a href="/monitoring/api/v3/distribution-metrics">Percentiles and distribution-valued metrics</a> .</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       OPERATIONS_BY_TABLE      </code></td>
<td><code dir="ltr" translate="no">       ARRAY&lt;STRUCT&gt;      </code></td>
<td><p>Impact of <code dir="ltr" translate="no">        INSERT       </code> or <code dir="ltr" translate="no">        UPDATE       </code> operations by the transaction on a per-table basis. This is indicated by the number of times that rows are affected and the number of bytes that are written.</p>
<strong>PostgreSQL interface note:</strong> PostgreSQL-dialect databases do not support this column.
<p>This column helps visualize the load on tables and provides insights into the rate at which a transaction writes to tables.</p>
<p>Specify the array as follows:<br />
<code dir="ltr" translate="no">        ARRAY&lt;STRUCT&lt;                TABLE STRING(MAX),                INSERT_OR_UPDATE_COUNT INT64,                INSERT_OR_UPDATE_BYTES INT64&gt;&gt;       </code></p></td>
</tr>
</tbody>
</table>

<sup>1</sup> `  _exists  ` is an internal field that is used to check whether a certain row exists or not.

### Example queries

This section includes several example SQL statements that retrieve transaction statistics. You can run these SQL statements using the [client libraries](/spanner/docs/reference/libraries) , the [gcloud spanner](/spanner/docs/gcloud-spanner#execute_sql_statements) , or the [Google Cloud console](/spanner/docs/create-query-database-console#run_a_query) .

#### List the basic statistics for each transaction in a given time period

The following query returns the raw data for the top transactions in the previous minute.

``` text
SELECT fprint,
       read_columns,
       write_constructive_columns,
       write_delete_tables,
       avg_total_latency_seconds,
       avg_commit_latency_seconds,
       operations_by_table,
       avg_bytes
FROM spanner_sys.txn_stats_top_minute
WHERE interval_end =
  (SELECT MAX(interval_end)
   FROM spanner_sys.txn_stats_top_minute);
```

##### Query output

<table>
<thead>
<tr class="header">
<th>fprint</th>
<th>read_columns</th>
<th>write_constructive_columns</th>
<th>write_delete_tables</th>
<th>avg_total_latency_seconds</th>
<th>avg_commit_latency_seconds</th>
<th>operations_by_table</th>
<th>avg_bytes</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       40015598317      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       ["Routes.name", "Cars.model"]      </code></td>
<td><code dir="ltr" translate="no">       ["Users"]      </code></td>
<td><code dir="ltr" translate="no">       0.006578737      </code></td>
<td><code dir="ltr" translate="no">       0.006547737      </code></td>
<td><code dir="ltr" translate="no">       [["Cars",1107,30996],["Routes",560,26880]]      </code></td>
<td><code dir="ltr" translate="no">       25286      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       20524969030      </code></td>
<td><code dir="ltr" translate="no">       ["id", "no"]      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       0.001732442      </code></td>
<td><code dir="ltr" translate="no">       0.000247442      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       0      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       77848338483      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       ["Cars", "Routes"]      </code></td>
<td><code dir="ltr" translate="no">       0.033467418      </code></td>
<td><code dir="ltr" translate="no">       0.000251418      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       0      </code></td>
</tr>
</tbody>
</table>

#### List the transactions with the highest average commit latency

The following query returns the transactions with high average commit latency in the previous hour, ordered from highest to lowest average commit latency.

``` text
SELECT fprint,
       read_columns,
       write_constructive_columns,
       write_delete_tables,
       avg_total_latency_seconds,
       avg_commit_latency_seconds,
       avg_bytes
FROM spanner_sys.txn_stats_top_hour
WHERE interval_end =
  (SELECT MAX(interval_end)
   FROM spanner_sys.txn_stats_top_hour)
ORDER BY avg_commit_latency_seconds DESC;
```

##### Query output

<table>
<thead>
<tr class="header">
<th>fprint</th>
<th>read_columns</th>
<th>write_constructive_columns</th>
<th>write_delete_tables</th>
<th>avg_total_latency_seconds</th>
<th>avg_commit_latency_seconds</th>
<th>avg_bytes</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       40015598317      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       ["Routes.name", "Cars.model"]      </code></td>
<td><code dir="ltr" translate="no">       ["Users"]      </code></td>
<td><code dir="ltr" translate="no">       0.006578737      </code></td>
<td><code dir="ltr" translate="no">       0.006547737      </code></td>
<td><code dir="ltr" translate="no">       25286      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       77848338483      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       ["Cars", "Routes"]      </code></td>
<td><code dir="ltr" translate="no">       0.033467418      </code></td>
<td><code dir="ltr" translate="no">       0.000251418      </code></td>
<td><code dir="ltr" translate="no">       0      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       20524969030      </code></td>
<td><code dir="ltr" translate="no">       ["id", "no"]      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       0.001732442      </code></td>
<td><code dir="ltr" translate="no">       0.000247442      </code></td>
<td><code dir="ltr" translate="no">       0      </code></td>
</tr>
</tbody>
</table>

#### Find the average latency of transactions which read certain columns

The following query returns the average latency information for transactions that read the column **ADDRESS** from 1 hour stats:

``` text
SELECT fprint,
       read_columns,
       write_constructive_columns,
       write_delete_tables,
       avg_total_latency_seconds
FROM spanner_sys.txn_stats_top_hour
WHERE 'ADDRESS' IN UNNEST(read_columns)
ORDER BY avg_total_latency_seconds DESC;
```

##### Query output

<table>
<thead>
<tr class="header">
<th>fprint</th>
<th>read_columns</th>
<th>write_constructive_columns</th>
<th>write_delete_tables</th>
<th>avg_total_latency_seconds</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       77848338483      </code></td>
<td><code dir="ltr" translate="no">       ["ID", "ADDRESS"]      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       ["Cars", "Routes"]      </code></td>
<td><code dir="ltr" translate="no">       0.033467418      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       40015598317      </code></td>
<td><code dir="ltr" translate="no">       ["ID", "NAME", "ADDRESS"]      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       ["Users"]      </code></td>
<td><code dir="ltr" translate="no">       0.006578737      </code></td>
</tr>
</tbody>
</table>

#### List transactions by the average number of bytes modified

The following query returns the transactions sampled in the last hour, ordered by the average number of bytes modified by the transaction.

``` text
SELECT fprint,
       read_columns,
       write_constructive_columns,
       write_delete_tables,
       avg_bytes
FROM spanner_sys.txn_stats_top_hour
ORDER BY avg_bytes DESC;
```

##### Query output

<table>
<thead>
<tr class="header">
<th>fprint</th>
<th>read_columns</th>
<th>write_constructive_columns</th>
<th>write_delete_tables</th>
<th>avg_bytes</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       40015598317      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       ["Users"]      </code></td>
<td><code dir="ltr" translate="no">       25286      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       77848338483      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       ["Cars", "Routes"]      </code></td>
<td><code dir="ltr" translate="no">       12005      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       20524969030      </code></td>
<td><code dir="ltr" translate="no">       ["ID", "ADDRESS"]      </code></td>
<td><code dir="ltr" translate="no">       []      </code></td>
<td><code dir="ltr" translate="no">       ["Users"]      </code></td>
<td><code dir="ltr" translate="no">       10923      </code></td>
</tr>
</tbody>
</table>

## Aggregate statistics

`  SPANNER_SYS  ` also contains tables to store aggregate data for all the transactions for which Spanner captured statistics in a specific time period:

  - `  SPANNER_SYS.TXN_STATS_TOTAL_MINUTE  ` : Aggregate statistics for all transactions during 1 minute intervals
  - `  SPANNER_SYS.TXN_STATS_TOTAL_10MINUTE  ` : Aggregate statistics for all transactions during 10 minute intervals
  - `  SPANNER_SYS.TXN_STATS_TOTAL_HOUR  ` : Aggregate statistics for all transactions during 1 hour intervals

Aggregate statistics tables have the following properties:

  - Each table contains data for non-overlapping time intervals of the length the table name specifies.

  - Intervals are based on clock times. 1 minute intervals end on the minute, 10 minute intervals end every 10 minutes starting on the hour, and 1 hour intervals end on the hour.
    
    For example, at 11:59:30 AM, the most recent intervals available to SQL queries on aggregate transaction statistics are:
    
      - **1 minute** : 11:58:00–11:58:59 AM
      - **10 minute** : 11:40:00–11:49:59 AM
      - **1 hour** : 10:00:00–10:59:59 AM

  - Each row contains statistics for **all** transactions executed over the database during the specified interval, aggregated together. There is only one row per time interval.

  - The statistics captured in the `  SPANNER_SYS.TXN_STATS_TOTAL_*  ` tables might include transactions that Spanner did not capture in the `  SPANNER_SYS.TXN_STATS_TOP_*  ` tables.

  - Some columns in these tables are exposed as metrics in Cloud Monitoring. The exposed metrics are:
    
      - Commit attempts count
      - Commit retries count
      - Transaction participants
      - Transaction latencies
      - Bytes written
    
    For more information, see [Spanner metrics](/monitoring/api/metrics_gcp_p_z#gcp-spanner) .

### Table schema

<table>
<colgroup>
<col style="width: 35%" />
<col style="width: 10%" />
<col style="width: 55%" />
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
<td>End of the time interval in which this statistic was captured.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ATTEMPT_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Total number of times that the transactions are attempted, including the attempts that abort before calling `commit`.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       COMMIT_ATTEMPT_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Total number of transaction commit attempts. This must match the number of calls to the transaction's <code dir="ltr" translate="no">       commit      </code> method.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       COMMIT_ABORT_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Total number of transaction attempts that were aborted, including those that are aborted before calling the transaction's <code dir="ltr" translate="no">       commit      </code> method.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       COMMIT_RETRY_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Number of commit attempts that are retries from previously aborted attempts. A Spanner transaction may have been tried multiple times before it commits due to lock contentions or transient events. A high number of retries relative to commit attempts indicates that there might be issues worth investigating. For more information, see <a href="#commit-counts">Understanding transactions and commit counts</a> on this page.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       COMMIT_FAILED_PRECONDITION_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Total number of transaction commit attempts that returned failed precondition errors, such as <code dir="ltr" translate="no">       UNIQUE      </code> index violations, row already exists, row not found, and so on.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       SERIALIZABLE_PESSIMISTIC_TXN_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Total number of transaction attempts initiated with the <a href="/spanner/docs/isolation-levels#serializable"><code dir="ltr" translate="no">        SERIALIZABLE       </code> isolation level</a> and <code dir="ltr" translate="no">       PESSIMISTIC      </code> locking. It's a subset of <code dir="ltr" translate="no">       ATTEMPT_COUNT      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       REPEATABLE_READ_OPTIMISTIC_TXN_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Total number of transaction attempts initiated with the <a href="/spanner/docs/isolation-levels#repeatable-read"><code dir="ltr" translate="no">        REPEATABLE READ       </code> isolation level</a> and <code dir="ltr" translate="no">       OPTIMISTIC      </code> locking. It's a subset of <code dir="ltr" translate="no">       ATTEMPT_COUNT      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_PARTICIPANTS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of participants in each commit attempt. To learn more about participants, see <a href="/spanner/docs/whitepapers/life-of-reads-and-writes">Life of Spanner Reads &amp; Writes</a> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_TOTAL_LATENCY_SECONDS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average seconds taken from the first operation of the transaction to commit/abort.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_COMMIT_LATENCY_SECONDS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average seconds taken to perform the commit operation.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_BYTES      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of bytes written by the transaction.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       TOTAL_LATENCY_DISTRIBUTION      </code></td>
<td><code dir="ltr" translate="no">       ARRAY&lt;STRUCT&gt;      </code></td>
<td><p>A histogram of the total commit latency, which is the time from the first transactional operation start time to the commit or abort time for all transaction attempts.</p>
<strong>PostgreSQL interface note:</strong> PostgreSQL-dialect databases do not support this column.
<p>If a transaction aborts multiple times and then successfully commits, latency is measured for each attempt until the final successful commit. The values are measured in seconds.</p>
<p>The array contains a single element and has the following type:<br />
<code dir="ltr" translate="no">        ARRAY&lt;STRUCT&lt;                COUNT INT64,                MEAN FLOAT64,                SUM_OF_SQUARED_DEVIATION FLOAT64,                NUM_FINITE_BUCKETS INT64,                GROWTH_FACTOR FLOAT64,                SCALE FLOAT64,                BUCKET_COUNTS ARRAY&lt;INT64&gt;&gt;&gt;       </code><br />
For more information about the values, see <a href="/monitoring/api/ref_v3/rest/v3/TypedValue#Distribution">Distribution</a> and <a href="/monitoring/api/ref_v3/rest/v3/TypedValue#exponential">Exponential</a> .</p>
<p>To calculate the desired percentile latency from the distribution, use the <code dir="ltr" translate="no">        SPANNER_SYS.DISTRIBUTION_PERCENTILE(distribution, n FLOAT64)       </code> function, which returns the estimated <em>n</em> th percentile. For an example, see <a href="#example-percentile-latency">Find the 99th percentile latency for transactions</a> .</p>
<p>For more information, see <a href="/monitoring/api/v3/distribution-metrics">Percentiles and distribution-valued metrics</a> .</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       OPERATIONS_BY_TABLE      </code></td>
<td><code dir="ltr" translate="no">       ARRAY&lt;STRUCT&gt;      </code></td>
<td><p>Impact of <code dir="ltr" translate="no">        INSERT       </code> or <code dir="ltr" translate="no">        UPDATE       </code> operations by all transactions on a per-table basis. This is indicated by the number of times that rows are affected and the number of bytes that are written.</p>
<strong>PostgreSQL interface note:</strong> PostgreSQL-dialect databases do not support this column.
<p>This column helps visualize the load on tables and provides insights into the rate at which the transactions write to tables.</p>
<p>Specify the array as follows:<br />
<code dir="ltr" translate="no">        ARRAY&lt;STRUCT&lt;                TABLE STRING(MAX),                INSERT_OR_UPDATE_COUNT INT64,                INSERT_OR_UPDATE_BYTES INT64&gt;&gt;       </code></p></td>
</tr>
</tbody>
</table>

### Example queries

This section includes several example SQL statements that retrieve transaction statistics. You can run these SQL statements using the [client libraries](/spanner/docs/reference/libraries) , the [gcloud spanner](/spanner/docs/gcloud-spanner#execute_sql_statements) , or the [Google Cloud console](/spanner/docs/create-query-database-console#run_a_query) .

#### Find the total number of commit attempts for all transactions

The following query returns the total number of commit attempts for all transactions in the most recent complete 1-minute interval:

``` text
SELECT interval_end,
       commit_attempt_count
FROM spanner_sys.txn_stats_total_minute
WHERE interval_end =
  (SELECT MAX(interval_end)
   FROM spanner_sys.txn_stats_total_minute)
ORDER BY interval_end;
```

##### Query output

<table>
<thead>
<tr class="header">
<th>interval_end</th>
<th>commit_attempt_count</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       2020-01-17 11:46:00-08:00      </code></td>
<td><code dir="ltr" translate="no">       21      </code></td>
</tr>
</tbody>
</table>

Note that there is only one row in the result because aggregated stats have only one entry per `  interval_end  ` for any time duration.

#### Find the total commit latency across all transactions

The following query returns the total commit latency across all transactions in the previous 10 minutes:

``` text
SELECT (avg_commit_latency_seconds * commit_attempt_count / 60 / 60)
  AS total_commit_latency_hours
FROM spanner_sys.txn_stats_total_10minute
WHERE interval_end =
  (SELECT MAX(interval_end)
   FROM spanner_sys.txn_stats_total_10minute);
```

##### Query output

<table>
<thead>
<tr class="header">
<th>total_commit_latency_hours</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       0.8967      </code></td>
</tr>
</tbody>
</table>

Note that there is only one row in the result because aggregated stats have only one entry per `  interval_end  ` for any time duration.

#### Find the 99th percentile latency for transactions

The following query returns the 99th percentile latency for transactions ran in the previous 10 minutes:

``` text
SELECT interval_end, avg_total_latency_seconds,
       SPANNER_SYS.DISTRIBUTION_PERCENTILE(total_latency_distribution[OFFSET(0)], 99.0)
  AS percentile_latency
FROM spanner_sys.txn_stats_total_10minute
WHERE interval_end =
  (SELECT MAX(interval_end)
   FROM spanner_sys.txn_stats_total_10minute)
ORDER BY interval_end;
```

##### Query output

<table>
<thead>
<tr class="header">
<th>interval_end</th>
<th>avg_total_latency_seconds</th>
<th>percentile_latency</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       2022-08-17 11:46:00-08:00      </code></td>
<td><code dir="ltr" translate="no">       0.34576998305986395      </code></td>
<td><code dir="ltr" translate="no">       9.00296190476190476      </code></td>
</tr>
</tbody>
</table>

Notice the large difference between the average and the 99th percentile latency. The 99th percentile latency helps identify possible outlier transactions with high latency.

There's only one row in the result because aggregated stats have only one entry per `  interval_end  ` for any time duration.

## Data retention

At a minimum, Spanner keeps data for each table for the following time periods:

  - `  SPANNER_SYS.TXN_STATS_TOP_MINUTE  ` and `  SPANNER_SYS.TXN_STATS_TOTAL_MINUTE  ` : Intervals covering the previous 6 hours.

  - `  SPANNER_SYS.TXN_STATS_TOP_10MINUTE  ` and `  SPANNER_SYS.TXN_STATS_TOTAL_10MINUTE  ` : Intervals covering the previous 4 days.

  - `  SPANNER_SYS.TXN_STATS_TOP_HOUR  ` and `  SPANNER_SYS.TXN_STATS_TOTAL_HOUR  ` : Intervals covering the previous 30 days.

**Note:** You cannot prevent Spanner from collecting transaction statistics. To delete the data in these tables, you must delete the database associated with the tables or wait until Spanner removes the data automatically. You can't extend the retention period for these tables.

Transaction statistics in Spanner give insight into how an application is using the database, and are useful when investigating performance issues. For example, you can check whether there are any slow running transactions that might be causing contention, or you can identify potential sources of high load, such as large volumes of updates to a particular column. Using the following steps, we'll show you how to use transaction statistics to investigate contentions in your database.

## Understand transactions and commit counts

A Spanner transaction may have to be tried multiple times before it commits. This most commonly occurs when two transactions attempt to work on the same data at the same time, and one of the transactions needs to be aborted to preserve the [isolation property](/spanner/docs/transactions#isolation) of the transaction. Some other transient events which can also cause a transaction to be aborted include:

  - Transient network issues.

  - Database schema changes being applied while a transaction is in the process of committing.

  - Spanner instance doesn't have the capacity to handle all the requests it is receiving.

In such scenarios, a client should retry the aborted transaction until it commits successfully or times out. For users of the official Spanner [client libraries](/spanner/docs/reference/libraries) , each library has implemented an automatic retry mechanism. If you are using a custom version of the client code, wrap your transaction commits in a retry loop.

A Spanner transaction may also be aborted due to a non-retriable error such as a transaction timeout, permission issues, or an invalid table/column name. There is no need to retry such transactions and the Spanner client library will return the error immediately.

The following table describes some examples of how `  COMMIT_ATTEMPT_COUNT  ` , `  COMMIT_ABORT_COUNT  ` , and `  COMMIT_RETRY_COUNT  ` are logged in different scenarios.

Scenario

COMMIT\_ATTEMPT\_COUNT

COMMIT\_ABORT\_COUNT

COMMIT\_RETRY\_COUNT

Transaction successfully committed on first attempt.

1

0

0

Transaction aborted due to time out error.

1

1

0

Transaction aborted due to transient network issue and successfully committed after one retry.

2

1

1

5 transactions with the same FPRINT are executed within a 10-minute interval. 3 of the transactions successfully committed on first attempt, while 2 transactions were aborted and then committed successfully on the first retry.

7

2

2

The data in the transactions-stats tables are aggregated data for a time interval. For a particular interval, it is possible that a transaction abort and retry happens around the boundaries and fall into different buckets. As a result, in a particular time interval aborts and retries may not be equal.

These stats are designed for troubleshooting and introspection and are not guaranteed to be 100% accurate. Statistics are aggregated in memory before being stored in Spanner tables. During an upgrade or other maintenance activities, Spanner servers can restart, affecting the accuracy of the numbers.

## Troubleshoot database contentions using transaction statistics

You can use SQL code or the [Transaction insights](/spanner/docs/use-lock-and-transaction-insights#txn-insights) dashboard to view the transactions in your database that might cause high latencies due to lock contentions.

The following topics show how you can investigate such transactions by using SQL code.

### Select a time period to investigate

This can be found from the application which is using Spanner.

For the purposes of this exercise, let's say that the issue started occurring at around 5:20pm on the 17th May, 2020.

You can use Transaction Tags to identify the source of the transaction and correlate between Transaction Statistics Table and Lock Statistics tables for effective lock contention troubleshooting. Read more at [Troubleshooting with transaction tags](/spanner/docs/introspection/troubleshooting-with-tags#transaction_tags) .

### Gather transaction statistics for the selected time period

To start our investigation, we'll query the `  TXN_STATS_TOTAL_10MINUTE  ` table around the start of the issue. The results of this query will show us how latency and other transaction statistics changed over that period of time.

For example, the following query returns the aggregated transaction statistics from `  4:30 pm  ` to `  7:40 pm  ` (inclusive).

``` text
SELECT
  interval_end,
  ROUND(avg_total_latency_seconds,4) as avg_total_latency_seconds,
  commit_attempt_count,
  commit_abort_count
FROM SPANNER_SYS.TXN_STATS_TOTAL_10MINUTE
WHERE
  interval_end >= "2020-05-17T16:40:00"
  AND interval_end <= "2020-05-17T19:40:00"
ORDER BY interval_end;
```

The following table lists example data returned from our query.

<table>
<thead>
<tr class="header">
<th style="text-align: right;">interval_end</th>
<th style="text-align: right;">avg_total_latency_seconds</th>
<th style="text-align: right;">commit_attempt_count</th>
<th style="text-align: right;">commit_abort_count</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: right;">2020-05-17 16:40:00-07:00</td>
<td style="text-align: right;">0.0284</td>
<td style="text-align: right;">315691</td>
<td style="text-align: right;">5170</td>
</tr>
<tr class="even">
<td style="text-align: right;">2020-05-17 16:50:00-07:00</td>
<td style="text-align: right;">0.0250</td>
<td style="text-align: right;">302124</td>
<td style="text-align: right;">3828</td>
</tr>
<tr class="odd">
<td style="text-align: right;">2020-05-17 17:00:00-07:00</td>
<td style="text-align: right;">0.0460</td>
<td style="text-align: right;">346087</td>
<td style="text-align: right;">11382</td>
</tr>
<tr class="even">
<td style="text-align: right;">2020-05-17 17:10:00-07:00</td>
<td style="text-align: right;">0.0864</td>
<td style="text-align: right;">379964</td>
<td style="text-align: right;">33826</td>
</tr>
<tr class="odd">
<td style="text-align: right;"><strong>2020-05-17 17:20:00-07:00</strong></td>
<td style="text-align: right;"><strong>0.1291</strong></td>
<td style="text-align: right;"><strong>390343</strong></td>
<td style="text-align: right;"><strong>52549</strong></td>
</tr>
<tr class="even">
<td style="text-align: right;"><strong>2020-05-17 17:30:00-07:00</strong></td>
<td style="text-align: right;"><strong>0.1314</strong></td>
<td style="text-align: right;"><strong>456455</strong></td>
<td style="text-align: right;"><strong>76392</strong></td>
</tr>
<tr class="odd">
<td style="text-align: right;"><strong>2020-05-17 17:40:00-07:00</strong></td>
<td style="text-align: right;"><strong>0.1598</strong></td>
<td style="text-align: right;"><strong>507774</strong></td>
<td style="text-align: right;"><strong>121458</strong></td>
</tr>
<tr class="even">
<td style="text-align: right;"><strong>2020-05-17 17:50:00-07:00</strong></td>
<td style="text-align: right;"><strong>0.1641</strong></td>
<td style="text-align: right;"><strong>516587</strong></td>
<td style="text-align: right;"><strong>115875</strong></td>
</tr>
<tr class="odd">
<td style="text-align: right;"><strong>2020-05-17 18:00:00-07:00</strong></td>
<td style="text-align: right;"><strong>0.1578</strong></td>
<td style="text-align: right;"><strong>552711</strong></td>
<td style="text-align: right;"><strong>122626</strong></td>
</tr>
<tr class="even">
<td style="text-align: right;"><strong>2020-05-17 18:10:00-07:00</strong></td>
<td style="text-align: right;"><strong>0.1750</strong></td>
<td style="text-align: right;"><strong>569460</strong></td>
<td style="text-align: right;"><strong>154205</strong></td>
</tr>
<tr class="odd">
<td style="text-align: right;"><strong>2020-05-17 18:20:00-07:00</strong></td>
<td style="text-align: right;"><strong>0.1727</strong></td>
<td style="text-align: right;"><strong>613571</strong></td>
<td style="text-align: right;"><strong>160772</strong></td>
</tr>
<tr class="even">
<td style="text-align: right;"><strong>2020-05-17 18:30:00-07:00</strong></td>
<td style="text-align: right;"><strong>0.1588</strong></td>
<td style="text-align: right;"><strong>601994</strong></td>
<td style="text-align: right;"><strong>143044</strong></td>
</tr>
<tr class="odd">
<td style="text-align: right;"><strong>2020-05-17 18:40:00-07:00</strong></td>
<td style="text-align: right;"><strong>0.2025</strong></td>
<td style="text-align: right;"><strong>604211</strong></td>
<td style="text-align: right;"><strong>170019</strong></td>
</tr>
<tr class="even">
<td style="text-align: right;"><strong>2020-05-17 18:50:00-07:00</strong></td>
<td style="text-align: right;"><strong>0.1615</strong></td>
<td style="text-align: right;"><strong>601622</strong></td>
<td style="text-align: right;"><strong>135601</strong></td>
</tr>
<tr class="odd">
<td style="text-align: right;"><strong>2020-05-17 19:00:00-07:00</strong></td>
<td style="text-align: right;"><strong>0.1653</strong></td>
<td style="text-align: right;"><strong>596804</strong></td>
<td style="text-align: right;"><strong>129511</strong></td>
</tr>
<tr class="even">
<td style="text-align: right;"><strong>2020-05-17 19:10:00-07:00</strong></td>
<td style="text-align: right;"><strong>0.1414</strong></td>
<td style="text-align: right;"><strong>560023</strong></td>
<td style="text-align: right;"><strong>112247</strong></td>
</tr>
<tr class="odd">
<td style="text-align: right;"><strong>2020-05-17 19:20:00-07:00</strong></td>
<td style="text-align: right;"><strong>0.1367</strong></td>
<td style="text-align: right;"><strong>570864</strong></td>
<td style="text-align: right;"><strong>100596</strong></td>
</tr>
<tr class="even">
<td style="text-align: right;">2020-05-17 19:30:00-07:00</td>
<td style="text-align: right;">0.0894</td>
<td style="text-align: right;">539729</td>
<td style="text-align: right;">65316</td>
</tr>
<tr class="odd">
<td style="text-align: right;">2020-05-17 19:40:00-07:00</td>
<td style="text-align: right;">0.0820</td>
<td style="text-align: right;">479151</td>
<td style="text-align: right;">40398</td>
</tr>
</tbody>
</table>

Here we see that aggregated latency and abort count is higher in the **highlighted** periods. We can pick any 10 minute interval where aggregated latency and/or abort count are high. Let's choose the interval ending at `  2020-05-17T18:40:00  ` and use it in the next step to identify which transactions are contributing to high latency and abort count.

### Identify transactions that are experiencing high latency

Let's now query the `  TXN_STATS_TOP_10MINUTE  ` table for the interval which was picked in the previous step. Using this data, we can start to identify which transactions are experiencing high latency and/or high abort count.

Execute the following query to get top performance-impacting transactions in descending order of total latency for our example interval ending at `  2020-05-17T18:40:00  ` .

``` text
SELECT
  interval_end,
  fprint,
  ROUND(avg_total_latency_seconds,4) as avg_total_latency_seconds,
  ROUND(avg_commit_latency_seconds,4) as avg_commit_latency_seconds,
  commit_attempt_count,
  commit_abort_count,
  commit_retry_count
FROM SPANNER_SYS.TXN_STATS_TOP_10MINUTE
WHERE
  interval_end = "2020-05-17T18:40:00"
ORDER BY avg_total_latency_seconds DESC;
```

<table>
<thead>
<tr class="header">
<th>interval_end</th>
<th>fprint</th>
<th>avg_total_latency_seconds</th>
<th>avg_commit_latency_seconds</th>
<th>commit_attempt_count</th>
<th>commit_abort_count</th>
<th>commit_retry_count</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>2020-05-17 18:40:00-07:00</strong></td>
<td><strong>15185072816865185658</strong></td>
<td><strong>0.3508</strong></td>
<td><strong>0.0139</strong></td>
<td><strong>278802</strong></td>
<td><strong>142205</strong></td>
<td><strong>129884</strong></td>
</tr>
<tr class="even">
<td>2020-05-17 18:40:00-07:00</td>
<td>15435530087434255496</td>
<td>0.1633</td>
<td>0.0142</td>
<td>129012</td>
<td>27177</td>
<td>24559</td>
</tr>
<tr class="odd">
<td>2020-05-17 18:40:00-07:00</td>
<td>14175643543447671202</td>
<td>0.1423</td>
<td>0.0133</td>
<td>5357</td>
<td>636</td>
<td>433</td>
</tr>
<tr class="even">
<td>2020-05-17 18:40:00-07:00</td>
<td>898069986622520747</td>
<td>0.0198</td>
<td>0.0158</td>
<td>6</td>
<td>0</td>
<td>0</td>
</tr>
<tr class="odd">
<td>2020-05-17 18:40:00-07:00</td>
<td>10510121182038036893</td>
<td>0.0168</td>
<td>0.0125</td>
<td>7</td>
<td>0</td>
<td>0</td>
</tr>
<tr class="even">
<td>2020-05-17 18:40:00-07:00</td>
<td>9287748709638024175</td>
<td>0.0159</td>
<td>0.0118</td>
<td>4269</td>
<td>1</td>
<td>0</td>
</tr>
<tr class="odd">
<td>2020-05-17 18:40:00-07:00</td>
<td>7129109266372596045</td>
<td>0.0142</td>
<td>0.0102</td>
<td>182227</td>
<td>0</td>
<td>0</td>
</tr>
<tr class="even">
<td>2020-05-17 18:40:00-07:00</td>
<td>15630228555662391800</td>
<td>0.0120</td>
<td>0.0107</td>
<td>58</td>
<td>0</td>
<td>0</td>
</tr>
<tr class="odd">
<td>2020-05-17 18:40:00-07:00</td>
<td>7907238229716746451</td>
<td>0.0108</td>
<td>0.0097</td>
<td>65</td>
<td>0</td>
<td>0</td>
</tr>
<tr class="even">
<td>2020-05-17 18:40:00-07:00</td>
<td>10158167220149989178</td>
<td>0.0095</td>
<td>0.0047</td>
<td>3454</td>
<td>0</td>
<td>0</td>
</tr>
<tr class="odd">
<td>2020-05-17 18:40:00-07:00</td>
<td>9353100217060788102</td>
<td>0.0093</td>
<td>0.0045</td>
<td>725</td>
<td>0</td>
<td>0</td>
</tr>
<tr class="even">
<td>2020-05-17 18:40:00-07:00</td>
<td>9521689070912159706</td>
<td>0.0093</td>
<td>0.0045</td>
<td>164</td>
<td>0</td>
<td>0</td>
</tr>
<tr class="odd">
<td>2020-05-17 18:40:00-07:00</td>
<td>11079878968512225881</td>
<td>0.0064</td>
<td>0.0019</td>
<td>65</td>
<td>0</td>
<td>0</td>
</tr>
</tbody>
</table>

We can see clearly that the first row ( **highlighted** ) in the preceding table shows a transaction experiencing high latency because of a high number of commit aborts. We can also see a high number of commit retries which indicates the aborted commits were subsequently retried. In the next step, we'll investigate further to see what is causing this issue.

### Identify the columns involved in a transaction experiencing high latency

In this step, we'll check whether high latency transactions are operating on the same set of columns by fetching `  read_columns  ` , `  write_constructive_columns  ` and `  write_delete_tables  ` data for transactions with high abort count. The `  FPRINT  ` value will also be useful in the next step.

``` text
SELECT
  fprint,
  read_columns,
  write_constructive_columns,
  write_delete_tables
FROM SPANNER_SYS.TXN_STATS_TOP_10MINUTE
WHERE
  interval_end = "2020-05-17T18:40:00"
ORDER BY avg_total_latency_seconds DESC LIMIT 3;
```

<table>
<thead>
<tr class="header">
<th style="text-align: right;">fprint</th>
<th style="text-align: right;">read_columns</th>
<th style="text-align: right;">write_constructive_columns</th>
<th>write_delete_tables</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: right;">15185072816865185658</td>
<td style="text-align: right;"><code dir="ltr" translate="no">       [TestHigherLatency._exists,TestHigherLatency.lang_status,TestHigherLatency.score,globalTagAffinity.shares]      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       [TestHigherLatency._exists,TestHigherLatency.shares,TestHigherLatency_lang_status_score_index.shares]      </code></td>
<td>[]</td>
</tr>
<tr class="even">
<td style="text-align: right;">15435530087434255496</td>
<td style="text-align: right;"><code dir="ltr" translate="no">       [TestHigherLatency._exists,TestHigherLatency.lang_status,TestHigherLatency.likes,globalTagAffinity.score]      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       [TestHigherLatency._exists,TestHigherLatency.likes,TestHigherLatency_lang_status_score_index.likes]      </code></td>
<td>[]</td>
</tr>
<tr class="odd">
<td style="text-align: right;">14175643543447671202</td>
<td style="text-align: right;"><code dir="ltr" translate="no">       [TestHigherLatency._exists,TestHigherLatency.lang_status,TestHigherLatency.score,globalTagAffinity.ugcCount]      </code></td>
<td style="text-align: right;"><code dir="ltr" translate="no">       [TestHigherLatency._exists,TestHigherLatency.ugcCount,TestHigherLatency_lang_status_score_index.ugcCount]      </code></td>
<td>[]</td>
</tr>
</tbody>
</table>

As the output shows in the preceding table, the transactions with the highest average total latency are reading the same columns. We can also observe some write contention since the transactions are writing to the same column, that is to say, `  TestHigherLatency._exists  ` .

### Determine how the transaction performance has changed over time

We can see how the statistics associated with this transaction shape have changed over a period of time. Use the following query, where $FPRINT is the fingerprint of the high latency transaction from the previous step.

``` text
SELECT
  interval_end,
  ROUND(avg_total_latency_seconds, 3) AS latency,
  ROUND(avg_commit_latency_seconds, 3) AS commit_latency,
  commit_attempt_count,
  commit_abort_count,
  commit_retry_count,
  commit_failed_precondition_count,
  avg_bytes
FROM SPANNER_SYS.TXN_STATS_TOP_10MINUTE
WHERE
  interval_end >= "2020-05-17T16:40:00"
  AND interval_end <= "2020-05-17T19:40:00"
  AND fprint = $FPRINT
ORDER BY interval_end;
```

<table>
<thead>
<tr class="header">
<th>interval_end</th>
<th>latency</th>
<th>commit_latency</th>
<th>commit_attempt_count</th>
<th>commit_abort_count</th>
<th>commit_retry_count</th>
<th>commit_failed_precondition_count</th>
<th>avg_bytes</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>2020-05-17 16:40:00-07:00</td>
<td>0.095</td>
<td>0.010</td>
<td>53230</td>
<td>4752</td>
<td>4330</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="even">
<td>2020-05-17 16:50:00-07:00</td>
<td>0.069</td>
<td>0.009</td>
<td>61264</td>
<td>3589</td>
<td>3364</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="odd">
<td>2020-05-17 17:00:00-07:00</td>
<td>0.150</td>
<td>0.010</td>
<td>75868</td>
<td>10557</td>
<td>9322</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="even">
<td>2020-05-17 17:10:00-07:00</td>
<td>0.248</td>
<td>0.013</td>
<td>103151</td>
<td>30220</td>
<td>28483</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="odd">
<td>2020-05-17 17:20:00-07:00</td>
<td>0.310</td>
<td>0.012</td>
<td>130078</td>
<td>45655</td>
<td>41966</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="even">
<td>2020-05-17 17:30:00-07:00</td>
<td>0.294</td>
<td>0.012</td>
<td>160064</td>
<td>64930</td>
<td>59933</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="odd">
<td>2020-05-17 17:40:00-07:00</td>
<td>0.315</td>
<td>0.013</td>
<td>209614</td>
<td>104949</td>
<td>96770</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="even">
<td>2020-05-17 17:50:00-07:00</td>
<td>0.322</td>
<td>0.012</td>
<td>215682</td>
<td>100408</td>
<td>95867</td>
<td>0</td>
<td>90</td>
</tr>
<tr class="odd">
<td>2020-05-17 18:00:00-07:00</td>
<td>0.310</td>
<td>0.012</td>
<td>230932</td>
<td>106728</td>
<td>99462</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="even">
<td>2020-05-17 18:10:00-07:00</td>
<td>0.309</td>
<td>0.012</td>
<td>259645</td>
<td>131049</td>
<td>125889</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="odd">
<td>2020-05-17 18:20:00-07:00</td>
<td>0.315</td>
<td>0.013</td>
<td>272171</td>
<td>137910</td>
<td>129411</td>
<td>0</td>
<td>90</td>
</tr>
<tr class="even">
<td>2020-05-17 18:30:00-07:00</td>
<td>0.292</td>
<td>0.013</td>
<td>258944</td>
<td>121475</td>
<td>115844</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="odd">
<td>2020-05-17 18:40:00-07:00</td>
<td>0.350</td>
<td>0.013</td>
<td>278802</td>
<td>142205</td>
<td>134229</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="even">
<td>2020-05-17 18:50:00-07:00</td>
<td>0.302</td>
<td>0.013</td>
<td>256259</td>
<td>115626</td>
<td>109756</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="odd">
<td>2020-05-17 19:00:00-07:00</td>
<td>0.315</td>
<td>0.014</td>
<td>250560</td>
<td>110662</td>
<td>100322</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="even">
<td>2020-05-17 19:10:00-07:00</td>
<td>0.271</td>
<td>0.014</td>
<td>238384</td>
<td>99025</td>
<td>90187</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="odd">
<td>2020-05-17 19:20:00-07:00</td>
<td>0.273</td>
<td>0.014</td>
<td>219687</td>
<td>84019</td>
<td>79874</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="even">
<td>2020-05-17 19:30:00-07:00</td>
<td>0.198</td>
<td>0.013</td>
<td>195357</td>
<td>59370</td>
<td>55909</td>
<td>0</td>
<td>91</td>
</tr>
<tr class="odd">
<td>2020-05-17 19:40:00-07:00</td>
<td>0.181</td>
<td>0.013</td>
<td>167514</td>
<td>35705</td>
<td>32885</td>
<td>0</td>
<td>91</td>
</tr>
</tbody>
</table>

In above output we can observe that total latency is high for the highlighted period of time. And, wherever total latency is high, `  commit_attempt_count  ` `  commit_abort_count  ` , and `  commit_retry_count  ` are also high even though commit latency ( `  commit_latency  ` ) has not changed very much. Since transaction commits are getting aborted more frequently, commit attempts are also high because of commit retries.

### Conclusion

In this example, we saw that high commit abort count was the cause of high latency. The next step is to look at the commit abort error messages received by the application to know the reason for aborts. By inspecting logs in the application, we see the application actually changed its workload during this time, i.e. some other transaction shape showed up with high `  attempts_per_second  ` , and that different transaction (maybe a nightly cleanup job) was responsible for the additional lock conflicts.

## Identify transactions that were not retried correctly

The following query returns the transactions sampled in the last ten minutes that have a high commit abort count, but no retries.

``` text
SELECT
  *
FROM (
  SELECT
    fprint,
    SUM(commit_attempt_count) AS total_commit_attempt_count,
    SUM(commit_abort_count) AS total_commit_abort_count,
    SUM(commit_retry_count) AS total_commit_retry_count
  FROM
    SPANNER_SYS.TXN_STATS_TOP_10MINUTE
  GROUP BY
    fprint )
WHERE
  total_commit_retry_count = 0
  AND total_commit_abort_count > 0
ORDER BY
  total_commit_abort_count DESC;
```

<table>
<thead>
<tr class="header">
<th>fprint</th>
<th>total_commit_attempt_count</th>
<th>total_commit_abort_count</th>
<th>total_commit_retry_count</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1557557373282541312</td>
<td>3367894</td>
<td>44232</td>
<td>0</td>
</tr>
<tr class="even">
<td>5776062322886969344</td>
<td>13566</td>
<td>14</td>
<td>0</td>
</tr>
</tbody>
</table>

We can see that the transaction with fprint **1557557373282541312** was aborted 44232 times, but it was never retried. This looks suspicious because the abort count is high and it is unlikely that every abort was caused by a non-retriable error. On the other hand, for the transaction with fprint **5776062322886969344** , it is less suspicious because the total abort count is not that high.

The following query returns more details about the transaction with fprint **1557557373282541312** including the `  read_columns  ` , `  write_constructive_columns  ` , and `  write_delete_tables  ` . This information helps to identify the transaction in client code, where the retry logic can be reviewed for this scenario.

``` text
SELECT
  interval_end,
  fprint,
  read_columns,
  write_constructive_columns,
  write_delete_tables,
  commit_attempt_count,
  commit_abort_count,
  commit_retry_count
FROM
  SPANNER_SYS.TXN_STATS_TOP_10MINUTE
WHERE
  fprint = 1557557373282541312
ORDER BY
  interval_end DESC;
```

<table>
<thead>
<tr class="header">
<th>interval_end</th>
<th>fprint</th>
<th>read_columns</th>
<th>write_constructive_columns</th>
<th>write_delete_tables</th>
<th>commit_attempt_count</th>
<th>commit_abort_count</th>
<th>commit_retry_count</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>2021-01-27T18:30:00Z</td>
<td>1557557373282541312</td>
<td>['Singers._exists']</td>
<td>['Singers.FirstName', 'Singers.LastName', 'Singers._exists']</td>
<td>[]</td>
<td>805228</td>
<td>1839</td>
<td>0</td>
</tr>
<tr class="even">
<td>2021-01-27T18:20:00Z</td>
<td>1557557373282541312</td>
<td>['Singers._exists']</td>
<td>['Singers.FirstName', 'Singers.LastName', 'Singers._exists']</td>
<td>[]</td>
<td>1034429</td>
<td>38779</td>
<td>0</td>
</tr>
<tr class="odd">
<td>2021-01-27T18:10:00Z</td>
<td>1557557373282541312</td>
<td>['Singers._exists']</td>
<td>['Singers.FirstName', 'Singers.LastName', 'Singers._exists']</td>
<td>[]</td>
<td>833677</td>
<td>2266</td>
<td>0</td>
</tr>
<tr class="even">
<td>2021-01-27T18:00:00Z</td>
<td>1557557373282541312</td>
<td>['Singers._exists']</td>
<td>['Singers.FirstName', 'Singers.LastName', 'Singers._exists']</td>
<td>[]</td>
<td>694560</td>
<td>1348</td>
<td>0</td>
</tr>
</tbody>
</table>

We can see that the transaction involves a read to the `  Singers._exists  ` hidden column to check the existence of a row. The transaction also writes to the `  Singers.FirstName  ` and `  Singer.LastName  ` columns. This information can help determine whether the transaction retry mechanism implemented in your custom client library is working as expected.

## What's next

  - Learn about other [Introspection tools](/spanner/docs/introspection) .
  - Learn about other information Spanner stores for each database in the database's [information schema](/spanner/docs/information-schema) tables.
  - Learn more about [SQL best practices](/spanner/docs/sql-best-practices) for Spanner.
  - Learn more about [Investigating high CPU utilization](/spanner/docs/introspection/investigate-cpu-utilization) .
