---
name: documents/docs.cloud.google.com/spanner/docs/mysql-perform-cutover
uri: https://docs.cloud.google.com/spanner/docs/mysql-perform-cutover
title: Perform cutover from MySQL
description: Learn how to perform a live migration cutover from MySQL to Spanner by using Datastream and Dataflow.
data_source: docs.cloud.google.com
---

This guide describes how to perform the cutover of a live database migration from a MySQL source instance to a Spanner database by using Datastream and Dataflow.

> **Note:** The cutover procedure described in this guide requires a planned application downtime window. During this window, pause all application database write operations to allow in-flight changes to drain and converge completely before switching application connections to Spanner.

## Before you begin

Before you perform the cutover, verify that you have completed the following tasks as part of your live data migration:

1.  **Schema migration** : Deploy your schema to Spanner. For more information, see [Migrate a schema from MySQL](https://docs.cloud.google.com/spanner/docs/migrate-mysql-schema) .
2.  **Backfill migration** : Complete the initial data migration by using the backfill migration Dataflow job. For more information, see [Perform the backfill migration](https://docs.cloud.google.com/spanner/docs/mysql-live-data-migration#perform-backfill-migration) .
3.  **Active CDC replication** : Verify that continuous change data capture (CDC) replication is active and healthy:
    - The Datastream stream is actively reading change events from your source MySQL database and writing them to the Cloud Storage bucket.
    - Cloud Storage bucket notifications are publishing object creation events to the Pub/Sub topic.
    - The streaming Dataflow job ( `Cloud_Datastream_to_Spanner` ) is consuming the Pub/Sub subscription and writing changes to Spanner with sub-second or negligible replication lag. For more information, see [Migrate your MySQL data](https://docs.cloud.google.com/spanner/docs/mysql-live-data-migration) .
4.  **Initial data validation** : Run data validation checks to verify data consistency between the source MySQL instance and the Spanner target. For more information, see [Validate your data migration](https://docs.cloud.google.com/spanner/docs/data-validation) .
5.  **Clean dead-letter queue (DLQ) state** : Verify that the forward replication severe DLQ directory in Cloud Storage (for example, `gs://MIGRATION_BUCKET/ENV/SERVICE/cdc/DB_SCHEMA/severe/` ) is completely empty. If severe errors exist, resolve the underlying issues and replay all failed change events by using a `retryDLQ` batch job before starting cutover. For more information, see [Monitor and resolve CDC DLQ errors](https://docs.cloud.google.com/spanner/docs/mysql-live-data-migration#resolve-cdc-dlq-errors) .
6.  **Reverse replication setup (Fallback readiness)** : If you require a contingency fallback pathway, set up an active reverse replication pipeline from Spanner back to your source MySQL database. For more information, see [Configure fallback to MySQL](https://docs.cloud.google.com/spanner/docs/mysql-configure-fallback) .

### Required roles

- To get the permissions that you need to perform the cutover, ask your administrator to grant you the following IAM roles on the project:

  - [Cloud Spanner Database User](https://docs.cloud.google.com/iam/docs/roles-permissions/spanner#spanner.databaseUser) ( `roles/spanner.databaseUser` )
  - [Dataflow Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/dataflow#dataflow.admin) ( `roles/dataflow.admin` )
  - [Datastream Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/datastream#datastream.admin) ( `roles/datastream.admin` )
  - [Storage Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/storage#storage.admin) ( `roles/storage.admin` )
  - [Monitoring Viewer](https://docs.cloud.google.com/iam/docs/roles-permissions/monitoring#monitoring.viewer) ( `roles/monitoring.viewer` )
  - [Pub/Sub Viewer](https://docs.cloud.google.com/iam/docs/roles-permissions/pubsub#pubsub.viewer) ( `roles/pubsub.viewer` )
  - [Viewer](https://docs.cloud.google.com/iam/docs/roles-overview#basic) ( `roles/viewer` )

## Perform cutover

The following sections describe how to drain the write pipeline and verify that all source changes replicate to Spanner before switching application writes.

### Verify pipeline health and DLQ state

Before pausing application write traffic or initiating downtime, verify that your forward replication pipeline has no stranded change events in its DLQ:

1.  In the Google Cloud console, go to the **Job details** page for your streaming Dataflow pipeline.

2.  On the **Custom metrics** tab, inspect the `Total permanent errors` custom counter.

3.  If this counter is greater than `0` , check your Cloud Storage severe DLQ directory (for example, `gs://MIGRATION_BUCKET/ENV/SERVICE/cdc/DB_SCHEMA/severe/` ) and verify that the directory is empty.

    > **Note:** The `Total permanent errors` counter is cumulative and doesn't decrement when records are retried. If any error files exist in your DLQ directory, don't proceed with cutover. For more information about resolving errors and replaying records by using a `retryDLQ` batch job, see \[Monitor and resolve CDC DLQ errors\](/spanner/docs/mysql-live-data-migration#resolve-cdc-dlq-errors).

### Pause application writes

> **Warning:** Application downtime starts at this step.

1.  Pause all database write traffic on your application.
2.  Allow active transactions on the source MySQL primary instances to fully commit and drain.

### Track and pause Datastream

1.  In the Google Cloud console, go to the **Stream details** page for your stream.

2.  On the **Monitoring** tab, inspect the **Data freshness** chart (Cloud Monitoring metric type: `datastream.googleapis.com/stream/freshness` ). Wait for the metric value to decrease to `0` . This indicates that Datastream captured and processed all source database changes up to the moment writes were paused. For more information, see [Monitor a stream](https://docs.cloud.google.com/datastream/docs/monitor-a-stream) .

3.  Verify that no replication events pass through the stream. On the **Monitoring** tab, inspect the **Throughput** chart (Monitoring metric type: `datastream.googleapis.com/stream/event_count` ) and verify that the event rate is `0` .

    Alternatively, to verify programmatically by using Metrics Explorer, query the event count by using Monitoring Query Language (MQL):

    ```
    fetch datastream.googleapis.com/Stream
    | metric 'datastream.googleapis.com/stream/event_count'
    | filter (resource.stream_id == 'STREAM_ID')
    | align rate(1m)
    | group_by [], [value_event_count_sum: sum(value.event_count)]
    | every 1m
    ```

    Replace `STREAM_ID` with the ID of your stream.

    Verify that this metric reports `0` for at least two consecutive aggregation periods.

4.  Pause your Datastream stream by using the Google Cloud console or the API, and wait for the stream status to change to `PAUSED` .

### Wait for in-flight change data to drain

1.  In the Google Cloud console, go to the **Stream details** page for your stream.
2.  On the **Monitoring** tab, inspect the **System latency** chart (Monitoring metric type: `datastream.googleapis.com/stream/system_latencies` ).
3.  Note the 99th percentile (P99) latency value and wait for at least two times the P99 system latency plus 30 seconds after pausing your stream. This waiting period helps ensure that all change data files generated before pausing are safely written to Cloud Storage and that their object creation events are published to Pub/Sub.
4.  Verify that your streaming Dataflow job consumed all pending change events. In the Google Cloud console, go to the Pub/Sub **Subscriptions** page, and then select the subscription created during your CDC migration setup.
5.  On the **Monitoring** tab, inspect the **Unacknowledged message count** chart (Monitoring metric type: `pubsub.googleapis.com/subscription/num_undelivered_messages` ) and verify that the count is `0` .

### Verify replication is complete

1.  Verify that your streaming Dataflow pipeline finished writing all replicated changes. On the **Job details** page for your streaming job, select the **Custom metrics** tab, and then inspect the `Replication lag total latency_MAX` user counter. Wait for the counter value to reach `0` .

    Alternatively, in Metrics Explorer, query the user counter by using MQL:

    ```
    fetch dataflow_job
    | metric 'dataflow.googleapis.com/job/user_counter'
    | filter (metric.job_id == 'JOB_ID' && metric.metric_name == 'Replication lag total latency_MAX')
    | group_by 1m, [value_p99_latency: percentile(value.user_counter, 99)]
    | every 1m
    ```

    Replace `JOB_ID` with the ID of your Dataflow job.

2.  Verify that write activity stopped on the target Spanner database. On the **Database details** page for your target database, select the **Monitoring** tab and inspect the **Write throughput** chart (Monitoring metric type: `spanner.googleapis.com/transaction_stat/total/bytes_written_count` ). Verify that write throughput is `0` for at least two consecutive 1-minute aggregation periods.

3.  Perform a final check of your forward replication DLQ. Inspect severe and retry DLQ directories in Cloud Storage (for example, `gs://MIGRATION_BUCKET/ENV/SERVICE/cdc/DB_SCHEMA/severe/` and `gs://MIGRATION_BUCKET/ENV/SERVICE/cdc/DB_SCHEMA/retry/` ) to verify that they remain empty.

4.  Drain your streaming Dataflow job:

    1.  In the Google Cloud console, go to the **Job details** page for your streaming job.
    2.  Click **Stop job** , select **Drain** , and then click **Stop job** .
    3.  Wait until the job status changes to `Drained` . For more information, see [Stop a running pipeline](https://docs.cloud.google.com/dataflow/docs/guides/stopping-a-pipeline) .

5.  Perform post-drain DLQ verification: After the job status changes to `Drained` , inspect severe and retry DLQ directories in Cloud Storage one final time. Verify that no new error files appeared while in-flight data drained. If any files exist, resolve and replay them. For more information, see [Monitor and resolve CDC DLQ errors](https://docs.cloud.google.com/spanner/docs/mysql-live-data-migration#resolve-cdc-dlq-errors) .

## Perform application cutover

> **Note:** Application downtime ends at this step.

Switch application database connections to Spanner and resume live read and write traffic:

1.  Update application configuration or database connection strings to target your Spanner database.
2.  Start application services and verify that live read and write traffic runs successfully against Spanner.
3.  Monitor your reverse replication pipeline for errors. If a Spanner write fails to replicate back to MySQL, Dataflow saves the failed record to a DLQ directory in Cloud Storage (for example, `gs://MIGRATION_BUCKET/ENV/SERVICE/reverse/DB_SCHEMA/severe/` ). Regularly check this path to verify that it remains empty. If failed records appear, see [Monitor and resolve reverse replication errors](https://docs.cloud.google.com/spanner/docs/mysql-configure-fallback#resolve-reverse-replication-errors) .

## What's next

- [Configure fallback to MySQL](https://docs.cloud.google.com/spanner/docs/mysql-configure-fallback)
- [Perform fallback to MySQL](https://docs.cloud.google.com/spanner/docs/mysql-fallback)
- [Migrate your MySQL data](https://docs.cloud.google.com/spanner/docs/mysql-live-data-migration)
- [Migrate from MySQL to Spanner](https://docs.cloud.google.com/spanner/docs/migrating-mysql-to-spanner)
