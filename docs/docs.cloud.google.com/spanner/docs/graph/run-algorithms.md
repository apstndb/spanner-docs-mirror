---
name: documents/docs.cloud.google.com/spanner/docs/graph/run-algorithms
uri: https://docs.cloud.google.com/spanner/docs/graph/run-algorithms
title: Run Spanner Graph algorithms
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

> **Preview — [Spanner Graph algorithms](https://docs.cloud.google.com/spanner/docs/graph/graph-algorithms-overview)**
> 
> This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](https://docs.cloud.google.com/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

> **Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

> **NOTE:** If you are running graph algorithms on a Spanner instance in `us-east1` region, you must specify one of these as `zone` in your algorithm [input parameters](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#common-algorithm-input-parameters) : `us-east1-b` , `us-east1-c` or `us-east1-d` . For all other regions, specifying a zone is not required.

This document explains how to run algorithms on Spanner Graph.

## Spanner Graph algorithm query structure

A Spanner Graph algorithm query has the following structure:

    EXPORT DATA OPTIONS (<export_option_list>) AS
    GRAPH graph_name
    <match_clause>
    <call_statement> algorithm_name(<common_input>, <algorithm_specific_input>)
      YIELD <algorithm_specific_output>
    RETURN <results>

  - `<export_option_list>` : Options that define how to persist algorithm query results. See [Cloud Storage Options](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#persisting-to-gcs) and [Spanner Options](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#persisting-to-spanner) .

  - `graph_name` : The name of the graph.

  - `<match_clause>` : Optional [MATCH](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-query-statements#gql_match) statements to define algorithm input elements.

  - `<call_statement>` : Use `CALL` when you omit `<match_clause>` and want to operate on the whole graph. Use `CALL PER()` when `<match_clause>` is present and you want to operate on the [working table](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-intro#working_table) . For more information, see [GQL CALL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-query-statements#gql_call) .

  - `algorithm_name` : The name of the algorithm to run. For available algorithms, see [Spanner Graph algorithms](https://docs.cloud.google.com/spanner/docs/graph/algorithms) .

  - `<common_input>` : Named input parameters common to all algorithm queries. For more information, see [common algorithm input parameters](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#common-algorithm-input-parameters) .

  - `<algorithm_specific_input>` : Named input parameters for the algorithm. For more information, see input parameters defined in [Spanner Graph algorithms](https://docs.cloud.google.com/spanner/docs/graph/algorithms) .

  - `<algorithm_specific_output>` : The output of the algorithm call. For more information, see the outputs defined in [Spanner Graph algorithms](https://docs.cloud.google.com/spanner/docs/graph/algorithms) and `YIELD` in [CALL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-query-statements#gql_call) statement.

  - `<results>` : Defines what to return in query results.

The query is composed of an `EXPORT DATA` statement, which defines how to persist results, and a [GRAPH CLAUSE](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-query-statements#graph_query) that produces the algorithm query result.

In its simplest form, the graph clause identifies the graph, `CALL` s an algorithm that yields predefined output, and then specifies what to `RETURN` from the algorithm's output.

Optionally, the graph clause can use supported [MATCH](https://docs.cloud.google.com/spanner/docs/graph/algorithm-schema-requirements-and-feature-compatibility#supported-match-clause) statements to select elements of interest. In this case, use a `PER ()` clause to group all rows returned by `MATCH` as input to the algorithm. The algorithm operates on a logical subgraph composed of the unique set of nodes and edges selected.

The query doesn't return any data. Results are persisted according to export\_option\_list.

For more information about Spanner Graph algorithm queries, see the following sections in this document:

  - [Common algorithm input parameters](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#common-algorithm-input-parameters)
  - [Handle algorithm output](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#handle-algorithm-output)
  - [Run example algorithm queries](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#run-example-algorithm-queries)

## Common algorithm input parameters

Specify these named input parameters in the following format: `NAME => VALUE, ...` .

| Name                   | Value Type | Required | Default Value | Description                                                                                                                                                                                                                                                                      |
| ---------------------- | ---------- | -------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `node_labels`          | ` ARRAY  ` | No       | (none)        | Only supported when `CALL` is used. A list of node labels to include in the algorithm input. If specified, only nodes with at least one matching label are included.                                                                                                             |
| `edge_labels`          | ` ARRAY  ` | No       | (none)        | Only supported when `CALL` is used. A list of edge labels to include in the algorithm input. If specified, only edges with at least one matching label are included.                                                                                                             |
| `edge_weight_property` | `STRING`   | No       | (none)        | The name of the edge property that contains the weights. If undefined, the system assigns a default weight of 1 to all edges. The property value type must be numeric.                                                                                                           |
| `machine_category`     | `STRING`   | No       | default       | The machine category to use for the algorithm execution. Supported values are: `default` , `large`                                                                                                                                                                               |
| `zone`                 | `STRING`   | No       | (none)        | The zone where the algorithm execution takes place. Must be one of the zones in the region the query is received in.                                                                                                                                                             |
| `max_idle_time`        | `STRING`   | No       | 30m           | Specifies how long the compute instance should remain active for reuse after the algorithm completes. Format is a sequence of decimal numbers, each with a unit suffix, such as `4m` , `1.5h` or `1h45m` . Valid time units are `ns` , `us` (or `µs` ), `ms` , `s` , `m` , `h` . |

## Handle algorithm output

You must persist algorithm query results before you can inspect them. Use the `export_data_option` to describe how to persist the results. You can persist the results to Cloud Storage or back to the same Spanner instance the query originated from.

### Persist results to Cloud Storage

To use this option, make sure [Storage Object Admin ( `roles/storage.objectAdmin` )](https://docs.cloud.google.com/iam/docs/roles-permissions/storage#storage.objectAdmin) role is granted to Google-managed [Spanner service account](https://docs.cloud.google.com/iam/docs/service-account-types#google-managed) `service- PROJECT_NUMBER @gcp-sa-spanner.iam.gserviceaccount.com` .

The following `EXPORT DATA` options are supported when persisting results to Cloud Storage. Specify the options in the following format: `NAME=VALUE, ...` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
</colgroup>
<thead>
<tr class="header">
<th>Name</th>
<th>Value Type</th>
<th>Required</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">uri</code></td>
<td><code dir="ltr" translate="no">STRING</code></td>
<td>Yes</td>
<td>The destination URI for the export, in <code dir="ltr" translate="no">gs://bucket/path/file</code> format. If you export a large amount of data, use a wildcard in <code dir="ltr" translate="no">uri</code> to export data into multiple files. For example, <code dir="ltr" translate="no">gs://bucket/path/file_*.csv</code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">format</code></td>
<td><code dir="ltr" translate="no">STRING</code></td>
<td>Yes</td>
<td>The format of exported data. Supported values: <code dir="ltr" translate="no">CSV</code> , <code dir="ltr" translate="no">PARQUET</code> , <code dir="ltr" translate="no">AVRO</code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">header</code></td>
<td><code dir="ltr" translate="no">BOOL</code></td>
<td>No</td>
<td>If <code dir="ltr" translate="no">true</code> , the system prints column headers for the first row of each data file. The default is <code dir="ltr" translate="no">false</code> . Applies only to CSV.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">overwrite</code></td>
<td><code dir="ltr" translate="no">BOOL</code></td>
<td>No</td>
<td>If <code dir="ltr" translate="no">true</code> , the system overwrites any existing files with the same URI. Otherwise, if files with the same URI exist, the statement returns an error. The default is <code dir="ltr" translate="no">false</code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">field_delimiter</code></td>
<td><code dir="ltr" translate="no">STRING</code></td>
<td>No</td>
<td>The delimiter that separates fields. Default: <code dir="ltr" translate="no">,</code> (comma). Applies only to CSV.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">compression</code></td>
<td><code dir="ltr" translate="no">STRING</code></td>
<td>No</td>
<td>Specifies compression format. If you don't specify a compression format, files remain uncompressed.
<ul>
<li>For <code dir="ltr" translate="no">CSV</code> , supported value is <code dir="ltr" translate="no">GZIP</code> .</li>
<li>For <code dir="ltr" translate="no">PARQUET</code> , supported values are: <code dir="ltr" translate="no">SNAPPY</code> , <code dir="ltr" translate="no">GZIP</code> , <code dir="ltr" translate="no">ZSTD</code> .</li>
<li>For <code dir="ltr" translate="no">AVRO</code> , supported values are: <code dir="ltr" translate="no">DEFLATE</code> , <code dir="ltr" translate="no">SNAPPY</code> .</li>
</ul></td>
</tr>
</tbody>
</table>

The column names in `RETURN` clause define the column names in Cloud Storage output files.

#### Export data into one or more files

Spanner Graph queries support a single wildcard operator ( `*` ) in the `uri` . The wildcard can appear in the filename component, but not in bucket name, folder name, or file extension. Using the wildcard operator instructs Spanner Graph to create multiple sharded files based on the pattern you supply if the result set is large. The system replaces the wildcard operator with a number, starting at zero, left-padded to 12 digits. For example, a URI `gs://my-bucket/file-*.csv` creates files like `gs://my-bucket/file-000000000000.csv` , `gs://my-bucket/file-000000000001.csv` , and similar files.

If you use a `uri` without wildcard, the result is a single file, like `gs://my-bucket/file.csv` .

#### Data types

When you export data, Spanner graph data types convert as follows, depending on the format:

**CSV**

All data types convert to their string representation:

  - `BOOL` values convert to `true` or `false` .
  - `BYTES` values base64-encode.
  - `TIMESTAMP` values format as `YYYY-MM-DD HH:MM:SS.ffffff UTC` .
  - `NULL` values appear as empty strings.

You cannot export nested and repeated data in CSV format.

**Avro**

| Spanner data type | Avro data type                            |
| ----------------- | ----------------------------------------- |
| `BOOL`            | `BOOLEAN`                                 |
| `INT64`           | `LONG`                                    |
| `FLOAT`           | `FLOAT`                                   |
| `DOUBLE`          | `DOUBLE`                                  |
| `NUMERIC`         | `BYTES` with logical type `DECIMAL(38,9)` |
| `STRING`          | `STRING`                                  |
| `BYTES`           | `BYTES`                                   |
| `TIMESTAMP`       | `LONG` (microseconds since epoch)         |
| `NULL`            | `null`                                    |

**Parquet**

| Spanner data type | Parquet data type  |
| ----------------- | ------------------ |
| `BOOL`            | `BOOLEAN`          |
| `INT64`           | `INT64`            |
| `FLOAT`           | `FLOAT`            |
| `DOUBLE`          | `DOUBLE`           |
| `NUMERIC`         | `DECIMAL(38,9)`    |
| `STRING`          | `STRING`           |
| `BYTES`           | `BYTE_ARRAY`       |
| `TIMESTAMP`       | `TIMESTAMP_MICROS` |
| `NULL`            | `null`             |

### Persist results to Spanner

The following `EXPORT DATA` options are supported when persisting results back to your source Spanner instance. Specify the options in the following format: `NAME=VALUE, ...` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
</colgroup>
<thead>
<tr class="header">
<th>Name</th>
<th>Value Type</th>
<th>Required</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">format</code></td>
<td><code dir="ltr" translate="no">STRING</code></td>
<td>Yes</td>
<td>The format of exported data. Must be <code dir="ltr" translate="no">CLOUD_SPANNER</code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">table</code></td>
<td><code dir="ltr" translate="no">STRING</code></td>
<td>Yes</td>
<td>The name of the destination Spanner table to write results to. This can be any table in the Spanner instance.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">write_mode</code></td>
<td><code dir="ltr" translate="no">STRING</code></td>
<td>Yes</td>
<td>The write mode to use. Supported values are:
<ul>
<li><code dir="ltr" translate="no">update_ignore_all</code> : Updates existing rows in the destination table.</li>
<li><code dir="ltr" translate="no">upsert_ignore_all</code> : Inserts new rows or updates existing rows in the destination table.</li>
</ul>
<br />
In both modes, Spanner skips any record that would introduce a constraint violation (for example, missing keys on an update, unique index violation, foreign key constraint violation). However, the write fails for non-constraint violation errors (for example, column type mismatch, missing values for NOT NULL columns).</td>
</tr>
</tbody>
</table>

#### Requirements

When you persist algorithm results back to Spanner, your algorithm query must satisfy the following:

  - **The destination table must exist** .
  - **Columns must exist with a matching type** : All column names specified in the `RETURN` clause must already exist in the destination table with a matching data type. Use aliases to match the destination table column names if needed. Example: `RETURN node.id AS person_id` .
  - **Include all primary key columns** : The `RETURN` clause must include all primary key columns of the destination table.

#### Write semantics

Persisting results back to Spanner is a non-transactional operation. It provides row-level atomicity. This means the system either successfully writes all columns from the same row or writes none of them. It follows at-least-once semantics. This means a row can be written to multiple times. Reading from the destination table while the execution is underway might yield incomplete results.

If the overall execution fails, the system does not roll back changes that have already been committed. The write process fails on the first non-retryable error. When a write failure occurs, the `ERROR_MESSAGE` in [GRAPH\_OPERATION\_EXECUTION\_STATUS](https://docs.cloud.google.com/spanner/docs/graph/run-algorithms#graph-operation-execution-status-schema) indicates the primary key of the row that failed along with the specific reason for the failure.

The system writes algorithm results back to Spanner Graph using `MEDIUM` [priority](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/RequestOptions#priority) .

## Run example algorithm queries

This section shows example Spanner Graph algorithm queries you can run on a test instance. For a full list of algorithms Spanner Graph supports, see [Spanner Graph algorithms](https://docs.cloud.google.com/spanner/docs/graph/algorithms) .

### Before you begin

To run the example Spanner Graph algorithm queries, you must first complete the following:

1.  Follow [Set up and query Spanner Graph](https://docs.cloud.google.com/spanner/docs/graph/set-up) to create a Spanner Graph.
2.  Ensure you have the [required permissions](https://docs.cloud.google.com/spanner/docs/graph/graph-algorithms-overview#permission) .
3.  Optional: Augment the Spanner Graph schema if you are persisting output to Spanner.

Add a new column named `page_rank` to the `Account` table. Spanner writes algorithm results to this new column. Then, refresh the graph definition so you can access `page_rank` as a node property.

    -- Add `page_rank` as a column. Data type of this column matches the data type defined in `PageRank` output signature.
    ALTER TABLE Account ADD COLUMN page_rank FLOAT64;
    
    -- Rerun the graph definition DDL to pickup `page_rank` as a new property.
    CREATE OR REPLACE PROPERTY GRAPH FinGraph
      NODE TABLES (`Account`, `Person`)
      EDGE TABLES (
        `PersonOwnAccount`
          SOURCE KEY (id) REFERENCES `Person` (id)
          DESTINATION KEY (account_id) REFERENCES `Account` (id)
          LABEL `Owns`,
        `AccountTransferAccount`
          SOURCE KEY (id) REFERENCES `Account` (id)
          DESTINATION KEY (to_id) REFERENCES `Account` (id)
          LABEL `Transfers`
      );

### Run algorithm on full graph with label filter and persist results to Cloud Storage

This example runs `PageRank` to rank `Account` s based on the `Transactions` they participate in and persists results to a Cloud Storage in CSV format as " `my-bucket-name/my-output.csv` "

    EXPORT DATA OPTIONS (
      uri = "gs://my-bucket-name/my-output.csv",
      format = "csv"
    ) AS
    GRAPH FinGraph
    CALL PageRank(node_labels => ['Account'], edge_labels => ['Transfers']) YIELD node, score
    RETURN node.id, score AS page_rank

In Cloud Storage, you should see a CSV file with two columns ( `id` and `page_rank` ) when this query completes successfully.

### Run algorithm on subgraph defined by `MATCH` and persist results to graph

This example uses the `MATCH` pattern to dynamically match a logical subgraph containing all `Account` nodes and only `Transfer` edges with an amount less than 500. This logical subgraph is the input to the `PageRank` algorithm. Spanner persists algorithm results back to the `Account` table.

    EXPORT DATA OPTIONS (
      format = "CLOUD_SPANNER",
      table = "Account",
      write_mode = 'update_ignore_all'
    ) AS
    GRAPH FinGraph
    MATCH (n:Account)
    RETURN n
    FULL UNION ALL
    MATCH -[e:Transfers WHERE e.amount < 500]->
    RETURN e
    NEXT
    CALL PER () PageRank() YIELD node, score
    RETURN node.id, score AS page_rank

After the query completes successfully, run the following query:

    GRAPH FinGraph
    MATCH (n:Account)
    RETURN n.id, ROUND(n.page_rank, 2) AS page_rank
    ORDER BY page_rank DESC, id ASC

You should see results similar to the following:

| id | page\_rank |
| -- | ---------- |
| 20 | 0.49       |
| 16 | 0.46       |
| 7  | 0.05       |

## Check algorithm execution status

When a graph algorithm query completes successfully, it returns zero rows and `Success` status. Depending on the input graph size and specific algorithm configurations, the algorithm execution might take a while to complete. You can check the progress and execution status of a graph algorithm query in `SPANNER_SYS.GRAPH_OPERATION_EXECUTION_STATUS` table. This table retains information for 30 days.

### `GRAPH_OPERATION_EXECUTION_STATUS` schema

| Column name             | Type        | Description                                                                                                                        |
| ----------------------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `QUERY_ID`              | `STRING`    | The ID for the graph algorithm query.                                                                                              |
| `QUERY_TEXT`            | `STRING`    | The query statement text.                                                                                                          |
| `START_TIMESTAMP`       | `TIMESTAMP` | The time at which the query started execution.                                                                                     |
| `LAST_UPDATE_TIMESTAMP` | `TIMESTAMP` | The time at which the status was last updated.                                                                                     |
| `PROGRESS`              | `FLOAT`     | Estimated percentage of completion. The value is between `0` and `1` , where `0` means started and `1` means completed.            |
| `STATUS`                | `STRING`    | Current state of execution. Possible values are `PENDING` , `IN_PROGRESS` , `OK` , `CANCELLED` , `DEADLINE_EXCEEDED` , `UNKNOWN` . |
| `ERROR_MESSAGE`         | `STRING`    | Error message if the query execution failed.                                                                                       |

The following sample query lists graph queries that have not yet completed successfully:

    SELECT
      query_id,
      query_text,
      start_timestamp,
      last_update_timestamp,
      progress,
      status,
      error_message
    FROM
      SPANNER_SYS.GRAPH_OPERATION_EXECUTION_STATUS
    WHERE
      status != "OK"
    ORDER BY
      start_timestamp DESC;

## Cancel algorithm execution

To cancel an in-flight graph algorithm query, locate the `query_id` from `SPANNER_SYS.GRAPH_OPERATION_EXECUTION_STATUS` table, then call [`cancel_query`](https://docs.cloud.google.com/spanner/docs/introspection/oldest-active-queries#cancel_an_expensive_query) for that `query_id` .

## What's next

  - [Spanner Graph algorithms catalog](https://docs.cloud.google.com/spanner/docs/graph/algorithms) .

  - [Spanner Graph algorithm schema requirements and feature compatibility](https://docs.cloud.google.com/spanner/docs/graph/algorithm-schema-requirements-and-feature-compatibility) .

  - [Spanner Graph algorithm best practices](https://docs.cloud.google.com/spanner/docs/graph/algorithm-best-practices) .
