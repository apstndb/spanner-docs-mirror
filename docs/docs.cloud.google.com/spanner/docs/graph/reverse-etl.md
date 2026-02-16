**Note:** Reverse ETL requires BigQuery slot reservations that use the BigQuery Enterprise or Enterprise Plus edition. Spanner Graph requires Spanner Enterprise or Enterprise Plus edition. For more information, see [Spanner editions overview](/spanner/docs/editions-overview) and [BigQuery editions overview](/bigquery/docs/editions-intro) .

This document describes how to use reverse extract, transform, and load (ETL) pipelines to move and continuously synchronize graph data from BigQuery to Spanner Graph. It covers the following key aspects:

  - [Common use cases for reverse ETL with graph data](#reverse-etl-use-cases) .
  - [The steps involved in a reverse ETL pipeline](#reverse-etl-pipeline) .
  - [Strategies for managing graph data changes](#manage-graph) , including insertions, updates, and deletions.
  - [Methods for orchestrating and maintaining reverse ETL pipelines](#pipeline-orchestration) .
  - [Best practices for optimizing your reverse ETL process](#reverse-etl-best-practices) .

To use reverse ETL to export data from BigQuery to Spanner, see [Export data to Spanner](/bigquery/docs/export-to-spanner) .

BigQuery performs complex data manipulation at scale as an analytical processing platform, while Spanner is optimized for use cases that require high QPS and low serving latency. Spanner Graph and BigQuery integrate effectively to prepare graph data in BigQuery analytics pipelines, enabling Spanner to serve low-latency graph traversals.

## Before you begin

1.  Create a Spanner instance with a database that contains graph data. For more information, see [Set up and query Spanner Graph](/spanner/docs/graph/set-up) .

2.  In BigQuery, create an [Enterprise or Enterprise Plus tier slot reservation](/bigquery/docs/reservations-tasks#create_reservations) . You can reduce BigQuery compute costs when you run exports to Spanner Graph. To do this, set a baseline slot capacity of zero and enable [autoscaling](/bigquery/docs/slots-autoscaling-intro) .

3.  Grant [Identity and Access Management (IAM) roles](#required_roles) that give users the necessary permissions to perform each task in this document.

### Required roles

To get the permissions that you need to export BigQuery graph data to Spanner Graph, ask your administrator to grant you the following IAM roles on your project:

  - Export data from a BigQuery table: [BigQuery Data Viewer](/iam/docs/roles-permissions/bigquery#bigquery.dataViewer) ( `  roles/bigquery.dataViewer  ` )
  - Run an export job: [BigQuery User](/iam/docs/roles-permissions/bigquery#bigquery.user) ( `  roles/bigquery.user  ` )
  - View parameters of the Spanner instance: [Cloud Spanner Viewer](/iam/docs/roles-permissions/spanner#spanner.viewer) ( `  roles/spanner.viewer  ` )
  - Write data to a Spanner Graph table: [Cloud Spanner Database User](/iam/docs/roles-permissions/spanner#spanner.databaseUser) ( `  roles/spanner.databaseUser  ` )

For more information about granting roles, see [Manage access to projects, folders, and organizations](/iam/docs/granting-changing-revoking-access) .

You might also be able to get the required permissions through [custom roles](/iam/docs/creating-custom-roles) or other [predefined roles](/iam/docs/roles-overview#predefined) .

## Reverse ETL use cases

The following are example use cases. After you analyze and process data in BigQuery, you can move the data to Spanner Graph using reverse ETL.

**Data aggregation and summarization** - Use BigQuery to compute aggregates over granular data to make it more suitable for operational use cases.

**Data transformation and enrichment** - Use BigQuery to cleanse and standardize data received from different data sources.

**Data filtering and selection** - Use BigQuery to filter a large dataset for analytical purposes. For example, you might filter out data that is not required for real-time applications.

**Feature preprocessing and engineering** - In BigQuery, use the [ML.TRANSFORM](/bigquery/docs/reference/standard-sql/bigqueryml-syntax-transform) function to transform data, or the [ML.FEATURE\_CROSS](/bigquery/docs/reference/standard-sql/bigqueryml-syntax-feature-cross) function to create [feature crosses](https://developers.google.com/machine-learning/crash-course/categorical-data/feature-crosses) of input features. Then, use reverse ETL to move the resulting data into Spanner Graph.

## Understand the reverse ETL pipeline

Data moves from BigQuery to Spanner Graph in a reverse ETL pipeline in two steps:

1.  BigQuery uses [slots](/bigquery/docs/slots) assigned to the pipeline job to extract and transform source data.

2.  The BigQuery reverse ETL pipeline uses Spanner APIs to load data into a provisioned Spanner instance.

The following diagram shows the steps in a reverse ETL pipeline:

**Figure 1.** BigQuery reverse ETL pipeline process

## Manage graph data changes

You can use reverse ETL to do the following:

  - Load a graph dataset from BigQuery to Spanner Graph.

  - Synchronize Spanner Graph data with ongoing updates from a dataset in BigQuery.

You configure a reverse ETL pipeline with a SQL query to specify the source data and the transformation to apply. The pipeline loads all data that satisfies the `  WHERE  ` clause of the `  SELECT  ` statement into Spanner using an *upsert* operation. An upsert operation is equivalent to [`  INSERT OR UPDATE  `](/spanner/docs/reference/standard-sql/dml-syntax#insert-or-update) statements. It inserts new rows and updates existing rows in tables that store graph data. The pipeline bases new and updated rows on a Spanner table primary key.

### Insert and update data for tables with load order dependencies

Spanner Graph schema design best practices recommend using interleaved tables and foreign keys. If you use interleaved tables or enforced foreign keys, you must load node and edge data in a specific order. This ensures referenced rows exist before you create the referencing row. For more information, see [Create interleaved tables](/spanner/docs/schema-and-data-model#create-interleaved-tables) .

The following example graph input table schema uses an interleaved table and a foreign key constraint to model the relationship between a person and their accounts:

``` text
CREATE TABLE Person (
  id    INT64 NOT NULL,
  name  STRING(MAX)
) PRIMARY KEY (id);

CREATE TABLE Account (
  id           INT64 NOT NULL,
  create_time  TIMESTAMP,
  is_blocked   BOOL,
  type        STRING(MAX)
) PRIMARY KEY (id);

CREATE TABLE PersonOwnAccount (
  id           INT64 NOT NULL,
  account_id   INT64 NOT NULL,
  create_time  TIMESTAMP,
  CONSTRAINT FK_Account FOREIGN KEY (account_id) REFERENCES Account (id)
) PRIMARY KEY (id, account_id),
  INTERLEAVE IN PARENT Person ON DELETE CASCADE;

CREATE PROPERTY GRAPH FinGraph
  NODE TABLES (
    Person,
    Account
  )
  EDGE TABLES (
    PersonOwnAccount
      SOURCE KEY (id) REFERENCES Person
      DESTINATION KEY (account_id) REFERENCES Account
      LABEL Owns
  );
```

In this example schema, `  PersonOwnAccount  ` is an interleaved table in `  Person  ` . Load elements in the `  Person  ` table before elements in the `  PersonOwnAccount  ` table. Additionally, the foreign key constraint on `  PersonOwnAccount  ` ensures a matching row exists in `  Account  ` , the edge relationship target. Therefore, load the `  Account  ` table before the `  PersonOwnAccount  ` table. The following list summarizes this schema's load order dependencies:

Follow these steps to load the data:

1.  Load `  Person  ` before `  PersonOwnAccount  ` .
2.  Load `  Account  ` before `  PersonOwnAccount  ` .

Spanner enforces the referential integrity constraints in the example schema. If the pipeline attempts to create a row in the `  PersonOwnAccount  ` table without a matching row in either the `  Person  ` table or `  Account  ` table, Spanner returns an error. The pipeline then fails.

This example reverse ETL pipeline uses [`  EXPORTDATA  `](/bigquery/docs/reference/standard-sql/export-statements) statements in BigQuery to export data from the `  Person  ` , `  Account  ` , and `  PersonOwnAccount  ` tables in a dataset to meet load order dependencies:

``` text
BEGIN
EXPORT DATA OPTIONS (
    uri="https://spanner.googleapis.com/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID",
    format='CLOUD_SPANNER',
    spanner_options="""{
      "table": "Person",
      "priority": "HIGH",
      "tag" : "graph_data_load_person"
    }"""
  ) AS
  SELECT
    id,
    name
  FROM
    DATASET_NAME.Person;

EXPORT DATA OPTIONS (
  uri="https://spanner.googleapis.com/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID",
  format='CLOUD_SPANNER',
  spanner_options="""{
    "table": "Account",
    "priority": "HIGH",
    "tag" : "graph_data_load_account"
  }"""
) AS
SELECT
  id,
  create_time,
  is_blocked,
  type
FROM
  DATASET_NAME.Account;

EXPORT DATA OPTIONS (
  uri="https://spanner.googleapis.com/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID",
  format='CLOUD_SPANNER',
  spanner_options="""{
    "table": "PersonOwnAccount",
    "priority": "HIGH",
    "tag" : "graph_data_load_person_own_account"
  }"""
) AS
SELECT
  id,
  account_id,
  create_time
FROM
  DATASET_NAME.PersonOwnAccount;
END;
```

### Synchronize data

To synchronize BigQuery with Spanner Graph, use reverse ETL pipelines. You can configure a pipeline to do one of the following:

  - Apply any insertions and updates from the BigQuery source to the Spanner Graph target table. You can add schema elements to the target tables to logically communicate deletes and remove target table rows on a schedule.

  - Use a time series function that applies insert and update operations and identifies delete operations.

### Referential integrity constraints

Unlike Spanner, BigQuery doesn't enforce primary and foreign key constraints. If your BigQuery data doesn't conform to the constraints you create on your Spanner tables, the reverse ETL pipeline might fail when loading that data.

Reverse ETL automatically groups data into batches that don't exceed the [maximum mutation per commit limit](/spanner/quotas#limits-for) and atomically applies the batches to a Spanner table in an arbitrary order. If a batch contains data that fails a referential integrity check, Spanner doesn't load that batch. Examples of such failures include an interleaved child row lacking a parent row or an enforced foreign key column without a matching value in the referenced column. If a batch fails a check, the pipeline fails with an error, and the pipeline stops loading batches.

#### Understand referential integrity constraint errors

The following examples show referential integrity constraint errors that you might encounter:

##### Resolve foreign key constraint errors

  - Error: "Foreign key constraint `  FK_Account  ` is violated on table `  PersonOwnAccount  ` . Can't find referenced values in `  Account(id)  ` "

  - Cause: A row insert into the `  PersonOwnAccount  ` table failed because a matching row in the `  Account  ` table, which the `  FK_Account  ` foreign key requires, is missing.

##### Resolve parent row missing errors

  - Error: "Parent row for row \[15,1\] in table `  PersonOwnAccount  ` is missing"

  - Cause: A row insert into `  PersonOwnAccount  ` ( `  id: 15  ` and `  account_id: 1  ` ) failed because a parent row in the `  Person  ` table ( `  id: 15  ` ) is missing.

To reduce the risk of referential integrity errors, consider the following options. Each option has tradeoffs.

  - Relax the constraints to allow Spanner Graph to load data.
  - Add logic to your pipeline to omit rows that violate referential integrity constraints.

### Relax referential integrity

One option to avoid referential integrity errors when loading data is to relax the constraints so that Spanner doesn't enforce referential integrity.

  - You can create [interleaved tables](/spanner/docs/schema-and-data-model#parent-child) with the `  INTERLEAVE IN  ` clause to use the same physical row interleaving characteristics. If you use `  INTERLEAVE IN  ` instead of `  INTERLEAVE IN PARENT  ` , Spanner doesn't enforce referential integrity, though queries benefit from the co-location of related tables.

  - You can create [informational foreign keys](/spanner/docs/foreign-keys/overview#informational-foreign-keys) by using the `  NOT ENFORCED  ` option. The `  NOT ENFORCED  ` option provides query optimization benefits. Spanner doesn't, however, enforce referential integrity.

For example, to create the edge input table without referential integrity checks, you can use this DDL:

``` text
CREATE TABLE PersonOwnAccount (
  id          INT64 NOT NULL,
  account_id  INT64 NOT NULL,
  create_time TIMESTAMP,
  CONSTRAINT FK_Account FOREIGN KEY (account_id) REFERENCES Account (id) NOT ENFORCED
) PRIMARY KEY (id, account_id),
INTERLEAVE IN Person;
```

### Respect referential integrity in reverse ETL pipelines

To ensure the pipeline loads only rows that satisfy the referential integrity checks, include only `  PersonOwnAccount  ` rows that have matching rows in the `  Person  ` and `  Account  ` tables. Then, preserve the load order, so Spanner loads `  Person  ` and `  Account  ` rows before the `  PersonOwnAccount  ` rows that refer to them.

``` text
EXPORT DATA OPTIONS (
  uri="https://spanner.googleapis.com/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID",
  format='CLOUD_SPANNER',
    spanner_options="""{
      "table": "PersonOwnAccount",
      "priority": "HIGH",
      "tag" : "graph_data_load_person_own_account"
    }"""
  ) AS
  SELECT
    poa.id,
    poa.account_id,
    poa.create_time
  FROM `PROJECT_ID.DATASET_NAME.PersonOwnAccount` poa
    JOIN `PROJECT_ID.DATASET_NAME.Person` p ON (poa.id = p.id)
    JOIN `PROJECT_ID.DATASET_NAME.Account` a ON (poa.account_id = a.id)
  WHERE poa.id = p.id
    AND poa.account_id = a.id;
```

**Note:** If you add [unenforced primary keys and foreign key constraints](https://cloud.google.com/blog/products/data-analytics/join-optimizations-with-bigquery-primary-and-foreign-keys?e=48754805) to your BigQuery tables, this query doesn't apply the joins. BigQuery requires you to maintain the constraints.

### Delete graph elements

Reverse ETL pipelines use *upsert* operations. Because upsert operations are equivalent to [`  INSERT OR UPDATE  `](/spanner/docs/reference/standard-sql/dml-syntax#insert-or-update) statements, a pipeline can only synchronize rows that exist in the source data at runtime. This means the pipeline excludes deleted rows. If you delete data from BigQuery, a reverse ETL pipeline can't directly remove the same data from Spanner Graph.

You can use one of the following options to handle deletions from BigQuery source tables:

#### Perform a logical or soft delete in the source

To logically mark rows for deletion, use a deleted flag in BigQuery. Then create a column in the target Spanner table to which you can propagate the flag. When the reverse ETL applies the pipeline updates, delete rows that have this flag in Spanner. You can find and delete such rows explicitly using [partitioned DML](/spanner/docs/dml-partitioned) . Alternatively, implicitly delete rows by configuring a [TTL (time to live)](/spanner/docs/ttl) column with a date that depends on the delete flag column. Write Spanner queries to exclude these logically deleted rows. This ensures that Spanner excludes these rows from results before scheduled deletion. After the reverse ETL pipeline runs to completion, Spanner reflects the logical deletes in its rows. You can then delete rows from BigQuery.

This example adds an `  is_deleted  ` column to the `  PersonOwnAccount  ` table in Spanner. It then adds an `  expired_ts_generated  ` column that depends on the `  is_deleted  ` value. The TTL policy schedules affected rows for deletion because the date in the generated column is earlier than the `  DELETION POLICY  ` threshold.

``` text
ALTER TABLE PersonOwnAccount
  ADD COLUMN is_deleted BOOL DEFAULT (FALSE);

ALTER TABLE PersonOwnAccount ADD COLUMN
  expired_ts_generated TIMESTAMP AS (IF(is_deleted,
    TIMESTAMP("1970-01-01 00:00:00+00"),
    TIMESTAMP("9999-01-01 00:00:00+00"))) STORED HIDDEN;

ALTER TABLE PersonOwnAccount
  ADD ROW DELETION POLICY (OLDER_THAN(expired_ts_generated, INTERVAL 0 DAY));
```

#### Use BigQuery change history for INSERT, UPDATE and logical deletes

**Note:** The BigQuery [`  CHANGES  `](/bigquery/docs/reference/standard-sql/time-series-functions#changes) function is in Preview and subject to the [Pre-GA Offerings Terms](https://cloud.google.com/terms/service-terms) in the Terms of Service. For more information, see the [launch stage](https://cloud.google.com/products?e=48754805#product-launch-stages) descriptions.

You can track changes to a BigQuery table using its change history. Use the GoogleSQL [`  CHANGES  `](/bigquery/docs/reference/standard-sql/time-series-functions#changes) function to find rows that changed within a specific time interval. Then, use the deleted row information with a reverse ETL pipeline. You can set up the pipeline to set an indicator, like a deleted flag or expiration date, in the Spanner table. This indicator marks rows for deletion in the Spanner tables.

Use the results from the `  CHANGES  ` time series function to decide which rows from the source table to include in the load of your reverse ETL pipeline.

The pipeline includes rows with `  _CHANGE_TYPE  ` as `  INSERT  ` or `  UPDATE  ` as upserts if the row exists in the source table. The current row from the source table provides the most recent data.

Use rows with `  _CHANGE_TYPE  ` as `  DELETE  ` that do *not* have existing rows in the source table to set an indicator in the Spanner table, such as a deleted flag or row expiration date.

Your export query must account for the order of insertions and deletions in BigQuery. For example, consider a row deleted at time T1 and a new row inserted at a later time T2. If both map to the same Spanner table row, the export must preserve the effects of these events in their original order.

If set, the *delete* indicator marks rows for deletion in the Spanner tables.

For example, you could add a column to a Spanner input table to store each row's expiration date. Then, create a deletion policy that uses these expiration dates.

The following example shows how to add a column to store the expiration dates of the table's rows.

``` text
ALTER TABLE PersonOwnAccount ADD COLUMN expired_ts TIMESTAMP;

ALTER TABLE PersonOwnAccount
  ADD ROW DELETION POLICY (OLDER_THAN(expired_ts, INTERVAL 1 DAY));
```

To use the `  CHANGES  ` function on a table in BigQuery, set the table's [`  enable_change_history  ` option](/bigquery/docs/reference/standard-sql/data-definition-language#table_option_list) to `  TRUE  ` :

``` text
ALTER TABLE `PROJECT_ID.DATASET_NAME.PersonOwnAccount`
  SET OPTIONS (enable_change_history=TRUE);
```

The following example shows how you can use reverse ETL to update new or changed rows and set the expiration date for rows marked for deletion. A left join with the `  PersonOwnAccount  ` table gives the query information about each row's current status.

``` text
EXPORT DATA OPTIONS (
  uri="https://spanner.googleapis.com/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID",
    format='CLOUD_SPANNER',
    spanner_options="""{
      "table": "PersonOwnAccount",
      "priority": "HIGH",
      "tag" : "graph_data_delete_via_reverse_etl"
    }"""
  ) AS
SELECT
  DISTINCT
   IF (changes._CHANGE_TYPE = 'DELETE', changes.id, poa.id) AS id,
   IF (changes._CHANGE_TYPE = 'DELETE', changes.account_id, poa.account_id) AS account_id,
   IF (changes._CHANGE_TYPE = 'DELETE', changes.create_time, poa.create_time) AS create_time,
   IF (changes._CHANGE_TYPE = 'DELETE', changes._CHANGE_TIMESTAMP, NULL) AS expired_ts
FROM
  CHANGES(TABLE `PROJECT_ID.DATASET_NAME.PersonOwnAccount`,
    TIMESTAMP_TRUNC(TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY), DAY),
    TIMESTAMP_TRUNC(CURRENT_TIMESTAMP(), DAY)) changes
LEFT JOIN `PROJECT_ID.DATASET_NAME.PersonOwnAccount` poa
  ON (poa.id = changes.id
  AND poa.account_id = changes.account_id)
WHERE (changes._CHANGE_TYPE = 'DELETE'
   AND poa.id IS NULL)
   OR (changes._CHANGE_TYPE IN ( 'UPDATE', 'INSERT')
   AND poa.id IS NOT NULL );
```

The example query uses a `  LEFT JOIN  ` with the source table to preserve order. This join ensures that `  DELETE  ` change records are ignored for rows deleted and then recreated within the query change history interval. The pipeline preserves the valid, new row.

When you delete rows, the pipeline populates the `  expired_ts  ` column in the corresponding Spanner Graph row using the `  DELETE  ` timestamp from the `  _CHANGE_TIMESTAMP  ` column. A row deletion policy (TTL policy) in Spanner deletes any row where the `  expired_ts  ` value is more than one day in the past.

To ensure system reliability, coordinate the pipeline's schedule, the change look-back window, and the Spanner TTL policy. Schedule the pipeline to run daily. The Spanner TTL policy must have a duration longer than this run interval. This prevents the pipeline from reprocessing a previous `  DELETE  ` event for a row already removed by the Spanner TTL policy.

This example shows the `  start_timestamp  ` and `  end_timestamp  ` interval for daily queries that capture all BigQuery table changes from the previous UTC day. Because this is a batch query and the `  CHANGES  ` function has limitations, the `  end_timestamp  ` must be at least 10 minutes before the current time. Therefore, schedule this query to run at least 10 minutes after midnight UTC. For more details, see the [`  CHANGES  `](/bigquery/docs/reference/standard-sql/time-series-functions#changes) documentation.

### Use TTL columns with last seen timestamp

A reverse ETL pipeline sets a `  last_seen_ts  ` column to the current timestamp for each row in the Spanner table. When you delete BigQuery rows, Spanner doesn't update corresponding rows, and the `  last_seen_ts  ` column doesn't change. Spanner then removes rows with an outdated `  last_seen_ts  ` by using a TTL policy or partitioned DML, based on a defined threshold. Before the scheduled deletion, Spanner queries can filter out rows with a `  last_seen_ts  ` older than this threshold. This approach works effectively when graph data updates routinely, and missing updates indicate stale data for deletion.

#### Perform a full refresh

Before loading from BigQuery, you can delete Spanner tables to reflect deletions in the source tables. This prevents the pipeline from loading rows deleted from the source BigQuery tables into Spanner during the next pipeline run. This might be the easiest option to implement. However, consider the time required to fully reload your graph data.

## Maintain a scheduled batch reverse ETL pipeline

After the initial run of your reverse ETL pipeline bulk-loads data from BigQuery into Spanner Graph, real-world data continues to change. Datasets change, and the pipeline adds or removes graph elements over time. The pipeline discovers new nodes and adds new edge relationships, or AI inference generates them.

To ensure the Spanner Graph database remains current, [schedule and sequence BigQuery](/bigquery/docs/orchestrate-workloads) pipeline orchestration using one of the following options:

[**BigQuery Pipelines**](/bigquery/docs/pipelines-introduction) let you develop, test, version-control, and deploy complex SQL data transformation workflows in BigQuery. It natively handles order dependencies by letting you define relationships between the queries in your pipeline. Dataform builds a dependency tree and runs your queries in the correct order. This ensures upstream dependencies complete before downstream tasks begin.

[**Workflows**](/workflows/docs) invoked by [**Cloud Scheduler**](/scheduler/docs) provide a useful and flexible solution to orchestrate sequences of Google Cloud services, including BigQuery queries. Define a workflow as a series of steps that each run a BigQuery job. You can use Cloud Scheduler to invoke these workflows on a defined schedule. Manage dependencies using the workflow definition to specify the order of execution, implement conditional logic, handle errors, and pass outputs from one query to another.

[**Scheduled queries**](/bigquery/docs/scheduling-queries) , also known as BigQuery transfer jobs, in BigQuery let you run SQL statements on a recurring basis. Scheduled queries don't offer robust error handling or dynamic dependency management.

## Reverse ETL with BigQuery continuous queries

The [BigQuery continuous queries](/bigquery/docs/continuous-queries-introduction) feature lets you run BigQuery operations in near real time. Combining `  EXPORT DATA  ` with continuous queries provides an alternative method for running reverse ETL pipelines that avoids scheduled batch jobs.

A continuous query is a long-running query that monitors a source BigQuery table for new rows. When BigQuery detects new rows appended to the table, it streams the query results to the `  EXPORT DATA  ` operation.

This approach offers the following advantages.

  - **Near real-time data synchronization** : New rows in BigQuery are reflected in Spanner with minimal delay.

  - **Reduced batch processing overhead** : A continuous query eliminates the need for periodic batch jobs, which reduces computational overhead.

  - **Event-driven updates** : Spanner data updates in response to actual changes in BigQuery.

A continuous query pipeline requires a slot reservation assignment with the `  job_type  ` of `  CONTINUOUS  ` . Assign this at the [project or folder](/bigquery/docs/reservations-assignments) level or at the [organization](/bigquery/docs/reservations-assignments#assign-organization) level.

### Create a continuous query with reverse ETL from BigQuery to Spanner

Configure the `  start_timestamp  ` parameter of the `  APPENDS  ` function to start processing data where the batch load left off. This function captures all rows created in the specific time window. In the following example, the pipeline arbitrarily sets the starting point to 10 minutes prior to the `  CURRENT_TIME  ` . This timestamp must be within the [BigQuery time travel window](/bigquery/docs/time-travel) .

There are [several methods](/bigquery/docs/continuous-queries#run_a_continuous_query_by_using_a_user_account) to start a continuous query pipeline, including the following:

1.  In BigQuery Studio, by selecting **More** and choosing **Continuous query** under **Choose query mode** .

2.  Use the *bq* CLI and provide the option `  --continuous=true  ` .

<!-- end list -->

``` text
EXPORT DATA OPTIONS ( uri="https://spanner.googleapis.com/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID",
  format="CLOUD_SPANNER",
  spanner_options="""{
      "table": "PersonOwnAccount",
      "priority": "HIGH",
      "tag": "reverse-etl-continuous",
      "change_timestamp_column": "create_time"
   }"""
)
AS SELECT id, account_id, _CHANGE_TIMESTAMP as create_time
  FROM
APPENDS(TABLE `PROJECT_ID.DATASET_NAME.PersonOwnAccount`,
  CURRENT_TIMESTAMP() - INTERVAL 10 MINUTE )
```

### Loading order not guaranteed

Spanner Graph data consists of multiple input tables. You must adhere to a strict loading order when tables have referential integrity constraints. However, concurrent continuous queries can't control the order in which Spanner adds rows. As a result, loading Spanner Graph data using continuous queries is only for graph schemas with relaxed referential integrity constraints.

### Integrate with existing pipelines

Continuous query complements existing scheduled batch jobs. For example, use continuous query for near real-time updates and scheduled jobs for full data synchronization or reconciliation.

Use BigQuery continuous query to build responsive and up-to-date reverse ETL pipelines for synchronizing data between BigQuery and Spanner Graph.

### Continuous queries considerations

  - **Cost** : Continuous queries incur costs for ongoing query execution and data streaming.

  - **Error handling** : A continuous query pipeline is canceled if it encounters any database errors, such as a duplicate primary key or a referential integrity violation. If a pipeline fails, you must manually correct the data in the source BigQuery table before you restart the query.

  - **Deletes and updates not handled** : The `  APPENDS  ` function only captures inserts. It doesn't capture deletes or updates.

## Follow reverse ETL best practices

For the best results, do the following.

  - Choose a strategy to prevent referential integrity errors when you load edge data.

  - Design your overall data pipeline to prevent dangling edges. Dangling edges can compromise Spanner Graph query efficiency and graph structure integrity. For more information, see [prevent dangling edges](/spanner/docs/graph/best-practices-designing-schema#prevent-dangling-edges) .

  - Follow Spanner [export optimization](/bigquery/docs/export-to-spanner#export_optimization) recommendations.

  - If you're loading a large amount of data, consider dividing the pipeline into multiple smaller pipelines to avoid hitting the default six-hour BigQuery query execution time quota. For more information, see [BigQuery query job limits](/bigquery/quotas#query_jobs) .

  - For large data loads, add indexes and [foreign key](/spanner/docs/foreign-keys/overview) constraints after the initial bulk data load is complete. This practice improves data loading performance because foreign key constraints require extra reads for validation and indexes require additional writes. These operations increase the number of transaction participants, which can slow down the data loading process.

  - Enable autoscaling in Spanner to speed up data load times into an instance. Then, configure the Spanner `  priority  ` parameter in the `  spanner_options  ` section of the BigQuery `  EXPORT DATA  ` command to `  HIGH  ` . For more information, see [Spanner autoscaling overview](/spanner/docs/autoscaling-overview) , [Configure exports with `  spanner_options  ` option](/bigquery/docs/export-to-spanner) , and [`  RequestOptions.priority  `](/spanner/docs/reference/rest/v1/RequestOptions#Priority) .

  - For large data loads, [create split points](/spanner/docs/create-manage-split-points) to pre-split your database. This prepares Spanner for increased throughput.

  - Configure Spanner [request priority](/spanner/docs/reference/rest/v1/RequestOptions#priority) for the data load in the pipeline definition.

## What's next

  - Review the [Spanner Graph overview](/spanner/docs/graph/overview) .
  - Learn how to [migrate to Spanner Graph](/spanner/docs/graph/migrate) .
  - Work with a [visualization of your graph](/spanner/docs/graph/overview) in Spanner.
  - Learn how to [use reverse ETL to export data from BigQuery to Spanner](/bigquery/docs/export-to-spanner) .
