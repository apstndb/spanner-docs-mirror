---
name: documents/docs.cloud.google.com/spanner/docs/mysql-configure-fallback
uri: https://docs.cloud.google.com/spanner/docs/mysql-configure-fallback
title: Configure fallback to MySQL
description: Configure fallback for MySQL by using reverse replication. Replicate data from Spanner to your source MySQL database by using Dataflow for minimal disruption.
data_source: docs.cloud.google.com
---

This document describes how to configure fallback for MySQL by using reverse replication. Fallback refers to a contingency plan to revert to your source MySQL database if you encounter issues with Spanner.

Reverse replication is useful when you encounter unexpected issues and need to fall back to the original MySQL database with minimal disruption to the service. Reverse replication lets you fall back by replicating data written to Spanner on your source MySQL database.

The reverse replication flow involves the following steps, performed by the [`Spanner_to_SourceDb` Dataflow template](https://github.com/GoogleCloudPlatform/DataflowTemplates/tree/main/v2/spanner-to-sourcedb) :

1.  Read changes from Spanner by using [Spanner change streams](https://docs.cloud.google.com/spanner/docs/change-streams) .

2.  Filter records by using the `forward_migration` filtration mode.

3.  Transform Spanner data to be compatible with your source database schema by using the Spanner migration tool. For more information, see [Custom transformation](https://googlecloudplatform.github.io/spanner-migration-tool/custom-transformation) .

4.  Verify whether the source MySQL database already contains more recent data for the specified primary key. If more recent data already exists, the pipeline skips the change event to avoid overwriting newer data.

5.  Write the data to your source database.

## Use the `Spanner_to_SourceDb` Dataflow template

The Dataflow template helps ensure consistency at a primary key level. The template creates metadata tables, known as *shadow tables* , in Spanner that contain the commit timestamp of the last write transaction on the shard for that particular table.

The write is consistent up to the primary key's commit timestamp.

Reverse replication runs as a continuous streaming Dataflow job. The job reads change events from Spanner change streams, converts them to data types compatible with the source database schema, and writes them to your MySQL database. If an event encounters a transient error, the job automatically retries it through a `retry/` dead-letter queue (DLQ) directory in Cloud Storage. If an event encounters a permanent error (or exceeds retry attempts), the job routes it to a `severe/` DLQ directory for manual resolution. For more information about resolving and replaying failed records, see [Monitor and resolve reverse replication errors](https://docs.cloud.google.com/spanner/docs/mysql-configure-fallback#resolve-reverse-replication-errors) .

## Before you begin

- Verify network connectivity between your source MySQL database and your Google Cloud project, where your Dataflow jobs run.

- Allowlist Dataflow worker IP addresses on your source MySQL instance.

- Verify that the MySQL credentials are correctly specified in the [`source shards file`](https://github.com/GoogleCloudPlatform/DataflowTemplates/blob/main/v2/spanner-to-sourcedb/README.md#sample-source-shards-file-for-MySQL) .

- Verify that your MySQL instance is online and running.

- Verify that the MySQL user has `INSERT` , `UPDATE` , and `DELETE` privileges on the MySQL database.

- Verify that you have the required IAM permissions to run a Dataflow flex template. For more information, see [Build and run a flex template](https://docs.cloud.google.com/dataflow/docs/guides/templates/using-flex-templates#before-you-begin) .

- Verify that the port `12345` is open for communication between the Dataflow worker VMs.

### Required roles

- To get the permissions that you need to launch the reverse replication, ask your administrator to grant you the following IAM roles on the instance:

  - [Cloud Spanner Database User](https://docs.cloud.google.com/iam/docs/roles-permissions/spanner#spanner.databaseUser) ( `roles/spanner.databaseUser` )
  - [Dataflow Developer](https://docs.cloud.google.com/iam/docs/roles-permissions/dataflow#dataflow.developer) ( `roles/dataflow.developer` )

<!-- -->

- To ensure that the Compute Engine service account has the necessary permissions to launch the reverse replication, ask your administrator to grant the following IAM roles to the Compute Engine service account on the instance:

  > **Important:** You must grant these roles to the Compute Engine service account, *not* to your user account. Failure to grant the roles to the correct principal might result in permission errors.

  - [Cloud Spanner Database User](https://docs.cloud.google.com/iam/docs/roles-permissions/spanner#spanner.databaseUser) ( `roles/spanner.databaseUser` )
  - [Secret Manager Secret Accessor](https://docs.cloud.google.com/iam/docs/roles-permissions/secretmanager#secretmanager.secretAccessor) ( `roles/secretmanager.secretAccessor` )
  - [Secret Manager Viewer](https://docs.cloud.google.com/iam/docs/roles-permissions/secretmanager#secretmanager.viewer) ( `roles/secretmanager.viewer` )

## Run reverse replication

To run reverse replication, follow these steps:

1.  Upload the [session file](https://googlecloudplatform.github.io/spanner-migration-tool/reports.html#session-file-ending-in-sessionjson) to the Cloud Storage bucket.

2.  Create a Pub/Sub notification for the `retry` folder of the DLQ directory. You can do this by creating a [Pub/Sub topic](https://docs.cloud.google.com/pubsub/docs/create-topic) and a [Pub/Sub subscription](https://docs.cloud.google.com/pubsub/docs/create-subscription) for that topic.

3.  Build and stage the Dataflow template. For more information, see [Building template](https://github.com/GoogleCloudPlatform/DataflowTemplates/blob/main/v2/spanner-to-sourcedb/README_Spanner_to_SourceDb.md#building-template) .

4.  To start the reverse replication streaming pipeline, run the following `gcloud dataflow flex-template run` command:

    ```
      gcloud dataflow flex-template run "spanner-to-sourcedb-job" \
      --project "PROJECT" \
      --region "REGION" \
      --template-file-gcs-location "TEMPLATE_SPEC_GCSPATH" \
      --parameters "changeStreamName=CHANGE_STREAM_NAME" \
      --parameters "instanceId=INSTANCE_ID" \
      --parameters "databaseId=DATABASE_ID" \
      --parameters "spannerProjectId=SPANNER_PROJECT_ID" \
      --parameters "metadataInstance=METADATA_INSTANCE" \
      --parameters "metadataDatabase=METADATA_DATABASE" \
      --parameters "sourceShardsFilePath=SOURCE_SHARDS_FILE_PATH" \
      # All flags after this point are optional
      [--parameters "startTimestamp=START_TIMESTAMP"] \
      [--parameters "endTimestamp=END_TIMESTAMP"] \
      [--parameters "shadowTablePrefix=SHADOW_TABLE_PREFIX"] \
      [--parameters "sessionFilePath=SESSION_FILE_PATH"] \
      [--parameters "filtrationMode=FILTRATION_MODE"] \
      [--parameters "shardingCustomJarPath=SHARDING_CUSTOM_JAR_PATH"] \
      [--parameters "shardingCustomClassName=SHARDING_CUSTOM_CLASS_NAME"] \
      [--parameters "shardingCustomParameters=SHARDING_CUSTOM_PARAMETERS"] \
      [--parameters "sourceDbTimezoneOffset=SOURCE_DB_TIMEZONE_OFFSET"] \
      [--parameters "dlqGcsPubSubSubscription=DLQ_GCS_PUB_SUB_SUBSCRIPTION"] \
      [--parameters "skipDirectoryName=SKIP_DIRECTORY_NAME"] \
      [--parameters "maxShardConnections=MAX_SHARD_CONNECTIONS"] \
      [--parameters "deadLetterQueueDirectory=DEAD_LETTER_QUEUE_DIRECTORY"] \
      [--parameters "dlqMaxRetryCount=DLQ_MAX_RETRY_COUNT"] \
      [--parameters "runMode=RUN_MODE"] \
      [--parameters "dlqRetryMinutes=DLQ_RETRY_MINUTES"] \
      [--parameters "sourceType=SOURCE_TYPE"] \
      [--parameters "transformationJarPath=TRANSFORMATION_JAR_PATH"] \
      [--parameters "transformationClassName=TRANSFORMATION_CLASS_NAME"] \
      [--parameters "transformationCustomParameters=TRANSFORMATION_CUSTOM_PARAMETERS"] \
      [--parameters "filterEventsDirectoryName=FILTER_EVENTS_DIRECTORY_NAME"]
    ```

    The mandatory variables are described in the following list:

    - `PROJECT` : the Google Cloud project ID.
    - `REGION` : the Google Cloud region where you want to run the Dataflow job.
    - `TEMPLATE_SPEC_GCSPATH` : the Cloud Storage path to the staged Dataflow template file.
    - `CHANGE_STREAM_NAME` : the name of the Spanner change stream that the job reads from.
    - `INSTANCE_ID` : the Spanner instance ID.
    - `DATABASE_ID` : the Spanner database ID.
    - `SPANNER_PROJECT_ID` : the project ID where your Spanner instances reside.
    - `METADATA_INSTANCE` : the Spanner instance that stores the metadata that the connector uses to control the consumption of change stream API data.
    - `METADATA_DATABASE` : the Spanner database that stores the metadata that the connector uses to control the consumption of change stream API data.
    - `SOURCE_SHARDS_FILE_PATH` : the Cloud Storage path that contains the connection profile information for the source shards.

    The optional variables are described in the following list:

    - `START_TIMESTAMP` : the timestamp from which to start reading changes. Defaults to empty.
    - `END_TIMESTAMP` : the timestamp until which to read changes. If not specified, the process reads changes indefinitely. Defaults to empty.
    - `SHADOW_TABLE_PREFIX` : the prefix for naming shadow tables. Default: `shadow_` .
    - `SESSION_FILE_PATH` : the path to the session file in Cloud Storage that contains mapping information from the Spanner migration tool.
    - `FILTRATION_MODE` : the mode that determines how to filter records based on criteria. Supported modes are `none` and `forward_migration` .
    - `SHARDING_CUSTOM_JAR_PATH` : the location (path) in Cloud Storage of the custom JAR file that contains the logic for fetching the shard ID. Defaults to empty.
    - `SHARDING_CUSTOM_CLASS_NAME` : the fully qualified class name that has the custom shard ID implementation. This parameter is required if `SHARDING_CUSTOM_JAR_PATH` is specified. Defaults to empty.
    - `SHARDING_CUSTOM_PARAMETERS` : a string containing any custom parameters to pass to the custom sharding class. Defaults to empty.
    - `SOURCE_DB_TIMEZONE_OFFSET` : the time zone offset from UTC for the source database. For example, `+10:00` . Default: `+00:00` .
    - `DLQ_GCS_PUB_SUB_SUBSCRIPTION` : the Pub/Sub subscription used in a Cloud Storage notification policy for the DLQ retry directory when running in `regular` mode. Specify the name in the format `projects/<PROJECT_ID>/subscriptions/<SUBSCRIPTION_ID>` . When you set this parameter, the system ignores `deadLetterQueueDirectory` and `dlqRetryMinutes` .
    - `SKIP_DIRECTORY_NAME` : the directory where the system writes records that it skips during reverse replication. Default: `skip` .
    - `MAX_SHARD_CONNECTIONS` : the maximum number of connections allowed per shard. Default: `10000` .
    - `DEAD_LETTER_QUEUE_DIRECTORY` : the path for storing the DLQ output. The default is a directory under the Dataflow job's temporary location.
    - `DLQ_MAX_RETRY_COUNT` : the maximum number of times the system retries temporary errors through the DLQ. Default: `500` .
    - `RUN_MODE` : the run mode type. Supported options are `regular` , `retryDLQ` , and `retryAllDLQ` . Use `retryDLQ` to retry error records from the `severe/` directory alongside an active pipeline, or `retryAllDLQ` to process both `retry/` and `severe/` directories when the regular pipeline is stopped. Default: `regular` .
    - `DLQ_RETRY_MINUTES` : the number of minutes between DLQ retries. Default: `10` .
    - `SOURCE_TYPE` : the type of source database to reverse replicate to. Default: `mysql` .
    - `TRANSFORMATION_JAR_PATH` : the location (path) in Cloud Storage of the custom JAR file that contains the custom transformation logic for processing records in reverse replication. Defaults to empty.
    - `TRANSFORMATION_CLASS_NAME` : the fully qualified class name that has the custom transformation logic. This parameter is required if `TRANSFORMATION_JAR_PATH` is specified. Defaults to empty.
    - `TRANSFORMATION_CUSTOM_PARAMETERS` : a string containing any custom parameters to pass to the custom transformation class. Defaults to empty.
    - `FILTER_EVENTS_DIRECTORY_NAME` : the directory in Cloud Storage where the system writes records that are filtered out by custom reverse transformation logic. Default: `filteredEvents` .

## Monitor and resolve reverse replication errors

When running reverse replication from Spanner back to MySQL, if your streaming Dataflow job can't write a change event to your MySQL primary database, it classifies the failure into one of two categories and writes the record to a dead-letter queue directory in your Cloud Storage bucket:

- **Retryable errors** : Temporary failures, such as MySQL database lock contention or network connection timeouts. Dataflow writes these events to a `retry/` dead-letter queue directory and automatically retries them within the same pipeline until they succeed or exceed `dlqMaxRetryCount` .
- **Severe errors** : Permanent failures that cannot be resolved by automatic retries, such as data type conversion errors, syntax errors, or exceptions inside custom transformation JARs. Dataflow writes these events to a `severe/` dead-letter queue directory (for example, `gs://MIGRATION_BUCKET/ENV/SERVICE/reverse/DB_SCHEMA/severe/` ) and increments the custom user counter `severe_error_count` .

Because severe errors cannot be resolved by automatic retries, Dataflow removes those failed events from the active replication stream and saves them in the `severe/` directory. If left unhandled, your rollback MySQL database will be missing those records. Regularly monitor the `severe_error_count` counter and the `severe/` directory during reverse replication so you can resolve and replay failed change events before initiating a rollback.

The Dataflow custom counter `severe_error_count` is a cumulative metric that records the total number of severe errors encountered since the streaming job started. When you replay and resolve failed records by using a DLQ retry job, this counter on your original streaming pipeline doesn't decrement. Therefore, your Cloud Storage severe DLQ directory ( `gs://MIGRATION_BUCKET/ENV/SERVICE/reverse/DB_SCHEMA/severe/` ) is the authoritative source of truth for whether unhandled errors exist.

### Replay failed records

When failed records appear in your severe DLQ directory, inspect the JSON error files in Cloud Storage, identify the root cause, and resolve the underlying issue.

To replay corrected DLQ records into MySQL, launch a new Dataflow job by using one of the following run modes:

- **`retryDLQ` mode** : Run this mode as a batch job alongside your active streaming pipeline to process error files in the `severe/` directory without halting live reverse replication. Run this mode after resolving the root causes of your severe errors.
- **`retryAllDLQ` mode** : If your regular streaming pipeline is stopped, use this mode to process both the `retry/` and `severe/` directories. Don't run `retryAllDLQ` concurrently with an active pipeline, as it will cause file contention and processing conflicts.

To launch a reverse replication DLQ retry job by using the gcloud CLI CLI, run the following command:

```
gcloud dataflow flex-template run "spanner-to-sourcedb-retry-job" \
    --project "PROJECT" \
    --region "REGION" \
    --template-file-gcs-location "TEMPLATE_SPEC_GCSPATH" \
    --parameters "changeStreamName=CHANGE_STREAM_NAME" \
    --parameters "instanceId=INSTANCE_ID" \
    --parameters "databaseId=DATABASE_ID" \
    --parameters "spannerProjectId=SPANNER_PROJECT_ID" \
    --parameters "metadataInstance=METADATA_INSTANCE" \
    --parameters "metadataDatabase=METADATA_DATABASE" \
    --parameters "sourceShardsFilePath=SOURCE_SHARDS_FILE_PATH" \
    --parameters "deadLetterQueueDirectory=DLQ_DIRECTORY" \
    --parameters "runMode=retryDLQ"
```

Replace the following:

- `PROJECT` : the Google Cloud project ID.
- `REGION` : the Google Cloud region where you want to run the Dataflow job.
- `TEMPLATE_SPEC_GCSPATH` : the Cloud Storage path to the staged Dataflow template file.
- `CHANGE_STREAM_NAME` : the name of the Spanner change stream that the job reads from.
- `INSTANCE_ID` : the Spanner instance ID.
- `DATABASE_ID` : the Spanner database ID.
- `SPANNER_PROJECT_ID` : the project ID where your Spanner instances reside.
- `METADATA_INSTANCE` : the Spanner instance that stores the metadata that the connector uses to control the consumption of change stream API data.
- `METADATA_DATABASE` : the Spanner database that stores the metadata that the connector uses to control the consumption of change stream API data.
- `SOURCE_SHARDS_FILE_PATH` : the Cloud Storage path that contains the connection profile information for the source shards.
- `DLQ_DIRECTORY` : the Cloud Storage path to the dead-letter queue directory containing the severe error files to retry.

## What's next

- [Perform cutover from MySQL](https://docs.cloud.google.com/spanner/docs/mysql-perform-cutover)
- [Perform fallback to MySQL](https://docs.cloud.google.com/spanner/docs/mysql-fallback)
