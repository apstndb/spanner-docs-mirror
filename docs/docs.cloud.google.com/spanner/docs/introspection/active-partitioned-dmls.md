*Active partitioned Data Manipulation Language (DML)* provides real time progress for the [partitioned DMLs](/spanner/docs/dml-partitioned) that are active in your database.

Spanner provides a built-in table, `  SPANNER_SYS.ACTIVE_PARTITIONED_DMLS  ` , that lists running partitioned DMLs and the progress made on them.

This page describes the table in detail, show some example queries that use this table and, finally, demonstrate how to use these queries to help mitigate issues caused by active partitioned DMLs. The information on this page is applicable to GoogleSQL-dialect databases and PostgreSQL-dialect databases.

## Access active partitioned DML statistics

Spanner provides the active partitioned DML statistics in the `  SPANNER_SYS  ` schema. You can use the following ways to access `  SPANNER_SYS  ` data:

  - A database's **Spanner Studio** page in the Google Cloud console.

  - The [`  gcloud spanner databases execute-sql  `](/sdk/gcloud/reference/spanner/databases/execute-sql) command.

  - The [`  executeSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql) or the [`  executeStreamingSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql) method.

The following single read methods that Spanner provides don't support `  SPANNER_SYS  ` :

  - Performing a strong read from a single row or multiple rows in a table.
  - Performing a stale read from a single row or multiple rows in a table.
  - Reading from a single row or multiple rows in a secondary index.

## `     ACTIVE_PARTITIONED_DMLS    `

`  SPANNER_SYS.ACTIVE_PARTITIONED_DMLS  ` returns a list of active partitioned DMLs sorted by their start time.

### Table schema

The following shows the table schema for SPANNER\_SYS.ACTIVE\_PARTITIONED\_DMLS.

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
<td><code dir="ltr" translate="no">       TEXT      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The partitioned DML query statement text.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       TEXT_FINGERPRINT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Fingerprint is a hash of the partitioned DML text.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       SESSION_ID      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The ID of the session that's executing the partitioned DML. <a href="/sdk/gcloud/reference/spanner/databases/sessions/delete">Deleting the session ID</a> will cancel the query.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       NUM_PARTITIONS_TOTAL      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The total number of partitions in the partitioned DML.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       NUM_PARTITIONS_COMPLETE      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The number of partitions that the partitioned DML has completed.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       NUM_TRIVIAL_PARTITIONS_COMPLETE      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The number of complete partitions where no rows were processed.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       PROGRESS      </code></td>
<td><code dir="ltr" translate="no">       DOUBLE      </code></td>
<td>The progress of a partitioned DML is calculated as the number of completed non-trivial partitions divided by the total number of non-trivial partitions.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ROWS_PROCESSED      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The number of rows processed so far, updated after each partition completes.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       START_TIMESTAMP      </code> .</td>
<td><code dir="ltr" translate="no">       TIMESTAMP      </code></td>
<td>An upper bound on the start time of a partitioned DML.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       LAST_UPDATE_TIMESTAMP      </code></td>
<td><code dir="ltr" translate="no">       TIMESTAMP      </code></td>
<td>Last timestamp when the partitioned DML made progress. Updated after a partition completes.</td>
</tr>
</tbody>
</table>

### Example queries

You can run the following example SQL statements using the [client libraries](/spanner/docs/reference/libraries) , the [Google Cloud CLI](/spanner/docs/gcloud-spanner#execute_sql_statements) , or the [Google Cloud console](/spanner/docs/create-query-database-console#run_a_query) .

#### Listing oldest running queries

The following query returns a list of running partitioned DMLs sorted by the start time of the query.

``` text
SELECT text,
       session_id,
       num_partitions_total,
       num_partitions_complete,
       num_trivial_partitions_complete,
       progress,
       rows_processed,
       start_timestamp,
       last_update_timestamp
FROM spanner_sys.active_partitioned_dmls
ORDER BY start_timestamp ASC;
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">text</th>
<th style="text-align: center;">session_id</th>
<th style="text-align: center;">num_partitions_total</th>
<th style="text-align: center;">num_partitions_complete</th>
<th style="text-align: center;">num_trivial_partitions_complete</th>
<th style="text-align: center;">progress</th>
<th style="text-align: center;">rows_processed</th>
<th style="text-align: center;">start_timestamp</th>
<th style="text-align: center;">last_update_timestamp</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">UPDATE Concerts SET VenueId = \'amazing venue\' WHERE SingerId &lt; 900000</td>
<td style="text-align: center;">5bd37a99-200c-5d2e-9021-15d0dbbd97e6</td>
<td style="text-align: center;">27</td>
<td style="text-align: center;">15</td>
<td style="text-align: center;">3</td>
<td style="text-align: center;">50.00%</td>
<td style="text-align: center;">2398654</td>
<td style="text-align: center;">2024-01-21 15:56:30.498744-08:00</td>
<td style="text-align: center;">2024-01-22 15:56:39.049799-08:00</td>
</tr>
<tr class="even">
<td style="text-align: left;">UPDATE Singers SET LastName = NULL WHERE LastName = ''</td>
<td style="text-align: center;">0028284f-0190-52f9-b396-aa588e034806</td>
<td style="text-align: center;">8</td>
<td style="text-align: center;">4</td>
<td style="text-align: center;">4</td>
<td style="text-align: center;">00.00%</td>
<td style="text-align: center;">0</td>
<td style="text-align: center;">2024-01-22 15:55:18.498744-08:00</td>
<td style="text-align: center;">2024-01-22 15:56:28.049799-08:00</td>
</tr>
<tr class="odd">
<td style="text-align: left;">DELETE from Singers WHERE SingerId &gt; 1000000</td>
<td style="text-align: center;">0071a85e-7e5c-576b-8a17-f9bc3d157eea</td>
<td style="text-align: center;">8</td>
<td style="text-align: center;">4</td>
<td style="text-align: center;">3</td>
<td style="text-align: center;">20.00%</td>
<td style="text-align: center;">238654</td>
<td style="text-align: center;">2024-01-22 15:56:30.498744-08:00</td>
<td style="text-align: center;">2024-01-22 15:56:19.049799-08:00</td>
</tr>
<tr class="even">
<td style="text-align: left;">UPDATE Singers SET MarketingBudget = 1000 WHERE true</td>
<td style="text-align: center;">036097a9-91d4-566a-a399-20c754eabdc2</td>
<td style="text-align: center;">8</td>
<td style="text-align: center;">5</td>
<td style="text-align: center;">0</td>
<td style="text-align: center;">62.50%</td>
<td style="text-align: center;">238654</td>
<td style="text-align: center;">2024-01-22 15:57:47.498744-08:00</td>
<td style="text-align: center;">2024-01-22 15:57:39.049799-08:00</td>
</tr>
</tbody>
</table>

## Limitations

Using the `  SPANNER_SYS.ACTIVE_PARTITIONED_DMLS  ` table has the following limitations:

  - `  PROGRESS  ` , `  ROWS_PROCESSED  ` , and `  LAST_UPDATE_TIMESTAMP  ` results are incremented at completed partition boundaries so the partitioned DML might keep updating rows while the values in these three fields stay the same.

  - If there are millions of partitions in a partitioned DML, the value in the `  PROGRESS  ` column might not capture all incremental progress. Use `  NUM_PARTITIONS_COMPLETE  ` and `  NUM_TRIVIAL_PARTITIONS_COMPLETE  ` to refer finer granularity progress.

  - If you cancel a partitioned DML using an RPC request, the cancelled partitioned DML might still appear in the table. If you cancel a partitioned DML using session deletion, it's removed from the table immediately. For more information, see [Deleting the session ID](/sdk/gcloud/reference/spanner/databases/sessions/delete) .

## Use active partitioned DML queries data to troubleshoot high CPU utilization

[Query statistics](/spanner/docs/introspection/query-statistics) and [transaction statistics](/spanner/docs/introspection/transaction-statistics) provide useful information when troubleshooting latency in a Spanner database. These tools provide information about the queries that have already completed. However, sometimes it's necessary to know what is running in the system. For example, consider the scenario when CPU utilization is high and you want to answer the following questions.

  - How many partitioned DMLs are running at the moment?
  - What are these partitioned DMLs?
  - How many of those partitioned DMLs are running for a long time?
  - Which session is running the query?

If you have answers for the preceding questions, you can decide to take the following action.

  - Delete the session executing the query for an immediate resolution.
  - Reduce the frequency of a partitioned DML.

In the following walkthrough, we examine active partitioned DMLs and determine what action, if any, to take.

### Retrieve a summary of active partitioned DMLs

In our example scenario, we notice higher than normal CPU usage, so we decide to run the following query to return the count of active partitioned DMLs.

``` text
SELECT count(*) as active_count
FROM spanner_sys.active_partitioned_dmls;
```

The query yields the following result.

<table>
<thead>
<tr class="header">
<th>active_count</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       22      </code></td>
</tr>
</tbody>
</table>

#### Listing the top 2 oldest running partitioned DMLs

We can then run a query to find more information about the top 2 oldest running partitioned DMLs sorted by the start time of the partitioned DML.

``` text
SELECT text,
       session_id,
       num_partitions_total,
       num_partitions_complete,
       num_trivial_partitions_complete,
       progress,
       rows_processed,
       start_timestamp,
       last_update_timestamp
FROM spanner_sys.active_partitioned_dmls
ORDER BY start_timestamp ASC LIMIT 2;
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">text</th>
<th style="text-align: center;">session_id</th>
<th style="text-align: center;">num_partitions_total</th>
<th style="text-align: center;">num_partitions_complete</th>
<th style="text-align: center;">num_trivial_partitions_complete</th>
<th style="text-align: center;">progress</th>
<th style="text-align: center;">rows_processed</th>
<th style="text-align: center;">start_timestamp</th>
<th style="text-align: center;">last_update_timestamp</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">UPDATE Concerts SET VenueId = \'amazing venue\' WHERE SingerId &lt; 900000</td>
<td style="text-align: center;">5bd37a99-200c-5d2e-9021-15d0dbbd97e6</td>
<td style="text-align: center;">27</td>
<td style="text-align: center;">15</td>
<td style="text-align: center;">3</td>
<td style="text-align: center;">50.00%</td>
<td style="text-align: center;">2398654</td>
<td style="text-align: center;">2024-01-21 15:56:30.498744-08:00</td>
<td style="text-align: center;">2024-01-22 15:56:39.049799-08:00</td>
</tr>
<tr class="even">
<td style="text-align: left;">UPDATE Singers SET LastName = NULL WHERE LastName = ''</td>
<td style="text-align: center;">0028284f-0190-52f9-b396-aa588e034806</td>
<td style="text-align: center;">8</td>
<td style="text-align: center;">4</td>
<td style="text-align: center;">4</td>
<td style="text-align: center;">00.00%</td>
<td style="text-align: center;">0</td>
<td style="text-align: center;">2024-01-22 15:55:18.498744-08:00</td>
<td style="text-align: center;">2024-01-22 15:56:28.049799-08:00</td>
</tr>
</tbody>
</table>

### Cancel an expensive query

We found a partitioned DML that has been running for days and isn't making progress. We can therefore run the following [`  gcloud spanner databases sessions delete  `](/sdk/gcloud/reference/spanner/databases/sessions/delete) command to delete the session using the session ID which cancels the partitioned DML.

``` text
gcloud spanner databases sessions delete\
   5bd37a99-200c-5d2e-9021-15d0dbbd97e6 \
    --database=singer_db --instance=test-instance
```

## What's next

  - Learn about other [Introspection tools](/spanner/docs/introspection) .
  - Learn about other information Spanner stores for each database in the database's [information schema](/spanner/docs/information-schema) tables.
  - Learn more about [SQL best practices](/spanner/docs/sql-best-practices) for Spanner.
