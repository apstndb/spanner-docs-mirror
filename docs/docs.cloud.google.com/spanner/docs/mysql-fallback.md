---
name: documents/docs.cloud.google.com/spanner/docs/mysql-fallback
uri: https://docs.cloud.google.com/spanner/docs/mysql-fallback
title: Perform fallback to MySQL
description: Perform fallback and disaster recovery rollback from Spanner back to your source MySQL database.
data_source: docs.cloud.google.com
---

This guide describes how to perform fallback and disaster recovery rollback from Spanner back to your source MySQL instances.

If you encounter unexpected performance degradation, unrecoverable application errors, or critical functional blockers after cutting over to Spanner, use the procedures in the following sections to roll back to your MySQL primary instances:

## Before you begin

Before initiating fallback procedures, verify the following prerequisites:

1.  **Active reverse replication** : Verify that the reverse replication Dataflow pipeline ( `Spanner_to_SourceDb` ) is running and actively consuming Spanner change streams in `forward_migration` filtering mode. For more information, see [Configure fallback to MySQL](https://docs.cloud.google.com/spanner/docs/mysql-configure-fallback) .
2.  **Source MySQL availability** : Verify that your source MySQL database is online, reachable from Dataflow, and able to accept write connections.
3.  **Maintenance window** : Alert stakeholders that a planned downtime window is required while Spanner writes are paused and reverse replication drains.

### Required roles

- To get the permissions that you need to perform fallback to MySQL, ask your administrator to grant you the following IAM roles on the project:

  - [Cloud Spanner Database User](https://docs.cloud.google.com/iam/docs/roles-permissions/spanner#spanner.databaseUser) ( `roles/spanner.databaseUser` )
  - [Dataflow Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/dataflow#dataflow.admin) ( `roles/dataflow.admin` )
  - [Datastream Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/datastream#datastream.admin) ( `roles/datastream.admin` )
  - [Storage Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/storage#storage.admin) ( `roles/storage.admin` )
  - [Monitoring Viewer](https://docs.cloud.google.com/iam/docs/roles-permissions/monitoring#monitoring.viewer) ( `roles/monitoring.viewer` )
  - [Viewer](https://docs.cloud.google.com/iam/docs/roles-overview#basic) ( `roles/viewer` )

## Pause Spanner writes

> **Warning:** Application downtime starts at this step.

1.  Pause all database write traffic to Spanner on your application.
2.  Allow active transactions on Spanner to fully commit and drain.

## Allow reverse replication to drain

The following sections describe how to verify that all changes written to Spanner post-cutover replicate back to MySQL before switching application writes.

### Monitor replication lag

Track replication lag to verify that all in-flight changes are written to MySQL:

1.  In the Google Cloud console, go to the **Job details** page for your reverse replication Dataflow job.
2.  On the **Custom metrics** tab, inspect the `replication_lag_in_seconds_LOGICAL_SHARD_NAME_MAX` user counter. For more information, see [Configure fallback to MySQL](https://docs.cloud.google.com/spanner/docs/mysql-configure-fallback) .
3.  Wait for the counter value to reach `0` .

Alternatively, in Metrics Explorer, query the user counter by using Monitoring Query Language (MQL):

```
fetch dataflow_job
| metric 'dataflow.googleapis.com/job/user_counter'
| filter (metric.job_id == 'JOB_ID' && metric.metric_name == 'replication_lag_insecondsLOGICAL_SHARD_NAME_MAX')
| group_by 1m, [value_p99_latency: percentile(value.user_counter, 99)]
| every 1m
```

Replace the following: \* `JOB_ID` : the ID of your reverse replication Dataflow job. \* `LOGICAL_SHARD_NAME` : the logical shard name configured for your source database.

### Verify MySQL write activity

Verify that reverse replication write operations have completed on your source database:

1.  In your source MySQL primary database, inspect active queries and database connection metrics.
2.  Verify that write traffic from the reverse replication pipeline has stopped.

### Verify clean DLQ state

Inspect the dead-letter queues (DLQ) to verify that no unhandled errors occurred during reverse replication:

1.  In the Google Cloud console, go to the **Job details** page for your reverse replication Dataflow job. On the **Custom metrics** tab, inspect the `severe_error_count` user counter.

2.  Inspect both severe and retry reverse replication DLQ directories in Cloud Storage (for example, `gs://MIGRATION_BUCKET/ENV/SERVICE/reverse/DB_SCHEMA/severe/` and `gs://MIGRATION_BUCKET/ENV/SERVICE/reverse/DB_SCHEMA/retry/` ).

    You can list the contents of the severe DLQ directory by using the gcloud CLI CLI:

    ```
    gcloud storage ls gs://MIGRATION_BUCKET/ENV/SERVICE/reverse/DB_SCHEMA/severe/
    ```

    Replace the following:

    - `MIGRATION_BUCKET` : the name of your Cloud Storage bucket.
    - `ENV` : your deployment environment name (for example, `prod` ).
    - `SERVICE` : your application or service name.
    - `DB_SCHEMA` : your database schema name.

3.  Verify that the DLQ directories remain completely empty.

4.  If error files exist, resolve underlying issues and replay failed records before proceeding. For more information, see [Monitor and resolve reverse replication errors](https://docs.cloud.google.com/spanner/docs/mysql-configure-fallback#resolve-reverse-replication-errors) .

## Resume forward CDC streaming

If you plan to resume forward migration to Spanner in the future, resume the forward Datastream stream:

1.  In the Google Cloud console, go to the **Stream details** page for your stream.
2.  Click **Resume** , and wait for the stream status to change to `RUNNING` . For more information, see [Migrate your MySQL data](https://docs.cloud.google.com/spanner/docs/mysql-live-data-migration) .

Alternatively, to resume the stream by using the gcloud CLI CLI, run the `gcloud datastream streams resume` command:

```
gcloud datastream streams resume STREAM_ID \
    --location=LOCATION
```

Replace the following:

- `STREAM_ID` : the ID of your Datastream stream.
- `LOCATION` : the Google Cloud region where your stream is located.

## Reroute writes to MySQL

> **Note:** Application downtime ends at this step.

1.  Update application configuration and database connection strings to target your source MySQL primary database instances.
2.  Start application services and resume live read and write traffic against MySQL.
3.  Verify that application read and write operations run successfully against the MySQL database.

## What's next

- [Perform cutover from MySQL](https://docs.cloud.google.com/spanner/docs/mysql-perform-cutover)
- [Configure fallback to MySQL](https://docs.cloud.google.com/spanner/docs/mysql-configure-fallback)
- [Migrate from MySQL to Spanner](https://docs.cloud.google.com/spanner/docs/migrating-mysql-to-spanner)
