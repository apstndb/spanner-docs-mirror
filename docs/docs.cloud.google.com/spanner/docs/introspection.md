Built-in statistics tables for Spanner help you investigate issues in your database. You can query these tables to gain insight about queries, transactions, and reads. The following list of built-in statistics tables (or introspection tools) are available in Spanner:

  - [Active partitioned DML statistics](#active-partitioned-DML-statistics)
  - [Column operations statistics](#column-operations-statistics)
  - [Lock statistics](#lock-statistics)
  - [Oldest active queries statistics](#oldest-active-queries)
  - [Query statistics](#query-statistics)
  - [Read statistics](#read-statistics)
  - [Split statistics](#split-statistics)
  - [Table operations statistics](#table-operations-statistics)
  - [Table sizes statistics](#table-sizes-statistics)
  - [Transaction statistics](#transaction-statistics)
  - [Vector index statistics](#vector-index-statistics)

These tables access `  SPANNER_SYS  ` data, which is available only through the following Spanner SQL interfaces:

  - A database's Spanner Studio page in the Google Cloud console.
  - The [`  gcloud spanner databases execute-sql  `](/sdk/gcloud/reference/spanner/databases/execute-sql) command.
  - The [`  executeSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql) or the [`  executeStreamingSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql) method.

You can't use other single read methods to query the `  SPANNER_SYS  ` tables.

## API methods included for each tool

In Spanner there is some overlap between [transactions](/spanner/docs/transactions) , [reads](/spanner/docs/reads) and [queries](/spanner/docs/reads#single_read_methods) . Therefore, it might not be clear which API methods are included when compiling results for each introspection tool. The following table lists the main API methods and their relationship to each tool.

API Methods

Transaction Modes

Query statistics

Oldest active queries

Read statistics

Transaction statistics

Lock statistics

Read, StreamingRead

Read-only transaction <sup>1</sup>

No

No

**Yes**

No

No

Read-write transaction

No

No

**Yes**

**Yes**

**Yes**

ExecuteSql, ExecuteStreamingSql

Read-only transaction <sup>1</sup>

**Yes <sup>2</sup>**

**Yes <sup>2</sup>**

No

No

No

Read-write transaction

**Yes**

**Yes**

No

**Yes**

**Yes**

ExecuteBatchDml

Read-write transactions

**Yes** <sup>3</sup>

**Yes** <sup>4</sup>

No

**Yes**

**Yes**

Commit

Read-write transactions (DML <sup>5</sup> , Mutations <sup>6</sup> )

No

No

No

**Yes**

**Yes**

Notes:

<sup>1</sup> Read-only transactions are not included in transaction statistics or lock statistics. Only read-write transactions are included in transaction statistics and lock statistics.

<sup>2</sup> Queries run with the PartitionQuery API aren't included in [oldest active queries statistics](/spanner/docs/introspection/oldest-active-queries) .

<sup>3</sup> A batch of DML statements appear in the query statistics as a single entry.

<sup>4</sup> Statements within the batch will appear in oldest active queries, rather than the entire batch.

<sup>5</sup> Uncommitted DML operations are not included in transaction statistics.

<sup>6</sup> Empty mutations that are effectively no-op are not included in transaction statistics.

## Active partitioned DML statistics

[Active partitioned data manipulation language (DML) statistics](/spanner/docs/introspection/active-partitioned-dmls) provide real time progress for the partitioned DMLs that are active in your database.

## Column operations statistics

You can use [Column operations statistics](/spanner/docs/introspection/column-operations-statistics) to do the following:

  - Monitor the statistics of columns in your table.
  - Find trends in how your table columns are used.

## Lock statistics

[Lock statistics](/spanner/docs/introspection/lock-statistics) can be used to investigate lock conflicts in your database. Used with transactions statistics, you can find transactions that are causing lock conflicts by trying to acquire locks on the same cells at the same time.

You can visualize these metrics on a time series by using the [Lock insights](/spanner/docs/use-lock-and-transaction-insights#lock-insights) dashboard. The pre-built dashboard helps you view the lock wait time and confirm if latencies are due to lock contentions with high lock wait time.

## Oldest active queries statistics

Sometimes you want to look at the current workload on the system by examining running queries. Use the [Oldest active queries](/spanner/docs/introspection/oldest-active-queries) tool to investigate long running queries that may be having an impact on database performance. This tool tells you what the queries are, when they started running and to which session they belong.

Change stream queries are not included in oldest active queries.

## Query statistics

When investigating issues in your database, it is helpful to know which queries are expensive, run frequently or scan a lot of data.

[Query statistics](/spanner/docs/introspection/query-statistics) are aggregated statistics for queries (including DML statements and [change stream](/spanner/docs/change-streams) queries), gathered in 1, 10, and 60 minute intervals. Statistics are collected for statements that completed successfully as well as those that failed, timed out, or were canceled by the user.

The statistics include highest CPU usage, total query execution counts, average latency, most data scanned, and additional basic query statistics. Use these statistics to help identify expensive, frequently run or data-intensive queries.

You can visualize these metrics on a time series by using [Query insights](/spanner/docs/using-query-insights#the_dashboard) dashboards. These pre-built dashboards help you view spikes in CPU utilization and identify inefficient queries.

## Read statistics

[Read statistics](/spanner/docs/introspection/read-statistics) can be used to investigate the most common and most resource-consuming reads on your database using the Spanner [Reads API](/spanner/docs/reads) . These statistics are collected and stored in 3 different time intervals - minute, 10 minutes and an hour. For each time interval, Spanner tracks the reads that are using the most resources.

Use read statistics to find out the combined resource usage by all reads, find the most CPU consuming reads, and find out how a specific read's frequency changes over time.

## Split statistics

You can [debug hotspots in your database](/spanner/docs/introspection/hot-split-statistics) to find splits in the database that are *warm* or *hot* , meaning that a high percentage of the load on a split is constrained by the available resources. You can view statistics for splits that had the highest split CPU usage scores over the last 6 hours, by 1-minute intervals.

## Table operations statistics

You can use [Table operations statistics](/spanner/docs/introspection/table-operations-statistics) to do the following:

  - Monitor the usages of your tables and indexes in your database.
  - Find trends in the usage of your tables and indexes.
  - Identify changes in traffic.

Also, you can correlate the changes in your table storage with the changes in your write traffic.

## Table sizes statistics

You can use [Table sizes statistics](/spanner/docs/introspection/table-sizes-statistics) to monitor the historical sizes of the tables and indexes in your database.

Use table sizes statistics to find trends in the sizes of your tables, indexes, and change stream tables. You can also keep a track of your biggest tables and indexes.

Please note that this feature provides a historical perspective only. It is not for real-time monitoring.

## Transaction statistics

You can use [Transaction statistics](/spanner/docs/introspection/transaction-statistics) to investigate transaction-related issues. For example, you can check for slow-running transactions that might be causing contention or identify changes in transaction shapes that are leading to performance regressions. Each row contains statistics of all transactions executed over the database during 1, 10, and 60 minute intervals.

You can visualize these metrics on a time series by using the [Transaction insights](/spanner/docs/use-lock-and-transaction-insights#txn-insights) dashboard. The pre-built dashboard helps you view the latencies in transactions and identify problematic transactions.

## Vector index statistics

You can use [vector index statistics](/spanner/docs/introspection/vector-index-statistics) to review the performance of your vector index, identify areas for improvement, and tune your index based on the metrics provided.
