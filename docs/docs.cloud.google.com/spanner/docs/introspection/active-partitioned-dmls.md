*Active partitioned Data Manipulation Language (DML)* provides real time progress for the [partitioned DMLs](https://docs.cloud.google.com/spanner/docs/dml-partitioned) that are active in your database.

Spanner provides a built-in table, `  SPANNER_SYS.ACTIVE_PARTITIONED_DMLS  ` , that lists running partitioned DMLs and the progress made on them.

This page describes the table in detail, show some example queries that use this table and, finally, demonstrate how to use these queries to help mitigate issues caused by active partitioned DMLs. The information on this page is applicable to GoogleSQL-dialect databases and PostgreSQL-dialect databases.

## Access active partitioned DML statistics

Spanner provides the active partitioned DML statistics in the `  SPANNER_SYS  ` schema. You can use the following ways to access `  SPANNER_SYS  ` data:

  - A database's **Spanner Studio** page in the Google Cloud console.

  - The [`  gcloud spanner databases execute-sql  `](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/execute-sql) command.

  - The [`  executeSql  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql) or the [`  executeStreamingSql  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql) method.

The following single read methods that Spanner provides don't support `  SPANNER_SYS  ` :

  - Performing a strong read from a single row or multiple rows in a table.
  - Performing a stale read from a single row or multiple rows in a table.
  - Reading from a single row or multiple rows in a secondary index.

## `     ACTIVE_PARTITIONED_DMLS    `

`  SPANNER_SYS.ACTIVE_PARTITIONED_DMLS  ` returns a list of active partitioned DMLs sorted by their start time.

### Table schema

The following shows the table schema for SPANNER\_SYS.ACTIVE\_PARTITIONED\_DMLS.

| Column name                                      | Type                       | Description                                                                                                                                                                                        |
| ------------------------------------------------ | -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `        TEXT       `                            | `        STRING       `    | The partitioned DML query statement text.                                                                                                                                                          |
| `        TEXT_FINGERPRINT       `                | `        INT64       `     | Fingerprint is a hash of the partitioned DML text.                                                                                                                                                 |
| `        SESSION_ID       `                      | `        STRING       `    | The ID of the session that's executing the partitioned DML. [Deleting the session ID](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/sessions/delete) will cancel the query. |
| `        NUM_PARTITIONS_TOTAL       `            | `        INT64       `     | The total number of partitions in the partitioned DML.                                                                                                                                             |
| `        NUM_PARTITIONS_COMPLETE       `         | `        INT64       `     | The number of partitions that the partitioned DML has completed.                                                                                                                                   |
| `        NUM_TRIVIAL_PARTITIONS_COMPLETE       ` | `        INT64       `     | The number of complete partitions where no rows were processed.                                                                                                                                    |
| `        PROGRESS       `                        | `        DOUBLE       `    | The progress of a partitioned DML is calculated as the number of completed non-trivial partitions divided by the total number of non-trivial partitions.                                           |
| `        ROWS_PROCESSED       `                  | `        INT64       `     | The number of rows processed so far, updated after each partition completes.                                                                                                                       |
| `        START_TIMESTAMP       ` .               | `        TIMESTAMP       ` | An upper bound on the start time of a partitioned DML.                                                                                                                                             |
| `        LAST_UPDATE_TIMESTAMP       `           | `        TIMESTAMP       ` | Last timestamp when the partitioned DML made progress. Updated after a partition completes.                                                                                                        |

### Example queries

You can run the following example SQL statements using the [client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) , the [Google Cloud CLI](https://docs.cloud.google.com/spanner/docs/gcloud-spanner#execute_sql_statements) , or the [Google Cloud console](https://docs.cloud.google.com/spanner/docs/create-query-database-console#run_a_query) .

#### Listing oldest running queries

The following query returns a list of running partitioned DMLs sorted by the start time of the query.

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

| text                                                                       |             session\_id              | num\_partitions\_total | num\_partitions\_complete | num\_trivial\_partitions\_complete | progress | rows\_processed |         start\_timestamp         |     last\_update\_timestamp      |
| :------------------------------------------------------------------------- | :----------------------------------: | :--------------------: | :-----------------------: | :--------------------------------: | :------: | :-------------: | :------------------------------: | :------------------------------: |
| UPDATE Concerts SET VenueId = \\'amazing venue\\' WHERE SingerId \< 900000 | 5bd37a99-200c-5d2e-9021-15d0dbbd97e6 |           27           |            15             |                 3                  |  50.00%  |     2398654     | 2024-01-21 15:56:30.498744-08:00 | 2024-01-22 15:56:39.049799-08:00 |
| UPDATE Singers SET LastName = NULL WHERE LastName = ''                     | 0028284f-0190-52f9-b396-aa588e034806 |           8            |             4             |                 4                  |  00.00%  |        0        | 2024-01-22 15:55:18.498744-08:00 | 2024-01-22 15:56:28.049799-08:00 |
| DELETE from Singers WHERE SingerId \> 1000000                              | 0071a85e-7e5c-576b-8a17-f9bc3d157eea |           8            |             4             |                 3                  |  20.00%  |     238654      | 2024-01-22 15:56:30.498744-08:00 | 2024-01-22 15:56:19.049799-08:00 |
| UPDATE Singers SET MarketingBudget = 1000 WHERE true                       | 036097a9-91d4-566a-a399-20c754eabdc2 |           8            |             5             |                 0                  |  62.50%  |     238654      | 2024-01-22 15:57:47.498744-08:00 | 2024-01-22 15:57:39.049799-08:00 |

## Limitations

Using the `  SPANNER_SYS.ACTIVE_PARTITIONED_DMLS  ` table has the following limitations:

  - `  PROGRESS  ` , `  ROWS_PROCESSED  ` , and `  LAST_UPDATE_TIMESTAMP  ` results are incremented at completed partition boundaries so the partitioned DML might keep updating rows while the values in these three fields stay the same.

  - If there are millions of partitions in a partitioned DML, the value in the `  PROGRESS  ` column might not capture all incremental progress. Use `  NUM_PARTITIONS_COMPLETE  ` and `  NUM_TRIVIAL_PARTITIONS_COMPLETE  ` to refer finer granularity progress.

  - If you cancel a partitioned DML using an RPC request, the cancelled partitioned DML might still appear in the table. If you cancel a partitioned DML using session deletion, it's removed from the table immediately. For more information, see [Deleting the session ID](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/sessions/delete) .

## Use active partitioned DML queries data to troubleshoot high CPU utilization

[Query statistics](https://docs.cloud.google.com/spanner/docs/introspection/query-statistics) and [transaction statistics](https://docs.cloud.google.com/spanner/docs/introspection/transaction-statistics) provide useful information when troubleshooting latency in a Spanner database. These tools provide information about the queries that have already completed. However, sometimes it's necessary to know what is running in the system. For example, consider the scenario when CPU utilization is high and you want to answer the following questions.

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

    SELECT count(*) as active_count
    FROM spanner_sys.active_partitioned_dmls;

The query yields the following result.

| active\_count       |
| ------------------- |
| `        22       ` |

#### Listing the top 2 oldest running partitioned DMLs

We can then run a query to find more information about the top 2 oldest running partitioned DMLs sorted by the start time of the partitioned DML.

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

| text                                                                       |             session\_id              | num\_partitions\_total | num\_partitions\_complete | num\_trivial\_partitions\_complete | progress | rows\_processed |         start\_timestamp         |     last\_update\_timestamp      |
| :------------------------------------------------------------------------- | :----------------------------------: | :--------------------: | :-----------------------: | :--------------------------------: | :------: | :-------------: | :------------------------------: | :------------------------------: |
| UPDATE Concerts SET VenueId = \\'amazing venue\\' WHERE SingerId \< 900000 | 5bd37a99-200c-5d2e-9021-15d0dbbd97e6 |           27           |            15             |                 3                  |  50.00%  |     2398654     | 2024-01-21 15:56:30.498744-08:00 | 2024-01-22 15:56:39.049799-08:00 |
| UPDATE Singers SET LastName = NULL WHERE LastName = ''                     | 0028284f-0190-52f9-b396-aa588e034806 |           8            |             4             |                 4                  |  00.00%  |        0        | 2024-01-22 15:55:18.498744-08:00 | 2024-01-22 15:56:28.049799-08:00 |

### Cancel an expensive query

We found a partitioned DML that has been running for days and isn't making progress. We can therefore run the following [`  gcloud spanner databases sessions delete  `](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/sessions/delete) command to delete the session using the session ID which cancels the partitioned DML.

    gcloud spanner databases sessions delete\
       5bd37a99-200c-5d2e-9021-15d0dbbd97e6 \
        --database=singer_db --instance=test-instance

## What's next

  - Learn about other [Introspection tools](https://docs.cloud.google.com/spanner/docs/introspection) .
  - Learn about other information Spanner stores for each database in the database's [information schema](https://docs.cloud.google.com/spanner/docs/information-schema) tables.
  - Learn more about [SQL best practices](https://docs.cloud.google.com/spanner/docs/sql-best-practices) for Spanner.
