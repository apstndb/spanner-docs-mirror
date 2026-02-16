This page explains how to set up fallback for MySQL using reverse replication. Fallback refers to a contingency plan to revert to your source MySQL database if you encounter issues with Spanner.

Reverse replication is useful when you encounter unexpected issues and need to fallback to the original MySQL database with minimum disruption to the service. Reverse replication lets you fallback by replicating data written to Spanner to your source MySQL database. This ensures both databases are eventually consistent.

The reverse replication flow involves the following steps, performed by the [`  Spanner_to_SourceDb  ` Dataflow template](https://github.com/GoogleCloudPlatform/DataflowTemplates/tree/main/v2/spanner-to-sourcedb) :

1.  Read changes from Spanner using [Spanner change streams](/spanner/docs/change-streams) .

2.  Ensure the mode of filtration is `  forward_migration  ` .

3.  Transform Spanner data to be compatible with your source database schema using the Spanner migration tool. For more information, see [Custom transformation](https://googlecloudplatform.github.io/spanner-migration-tool/custom-transformation) .

4.  Verify whether the source database already contains more recent data for the specified primary key.

5.  Write the data to your source database.

## Use the `     Spanner_to_SourceDb    ` Dataflow template

The Dataflow template ensures consistency at a primary key level. The template creates metadata tables, known as *shadow tables* , in Spanner that contain the commit timestamp of the last write transaction on the shard for that particular table.

The write is consistent up to the primary key's commit timestamp.

You can configure the Dataflow job that performs the reverse replication to run in one of the following modes:

  - **Regular** : This is the default mode. The Dataflow job reads events from Spanner change streams, converts them to data types compatible with the source database schema, and applies them to the source database. The job automatically retries errors. After exhausting retries, it moves the errors to the `  severe  ` folder of the dead letter queue (DLQ) directory in the Cloud Storage bucket. The job also moves all permanant errors to the `  severe  ` folder.

  - **RetryDLQ** : In this mode, the Dataflow job reads events from the `  severe  ` folder of the DLQ and retries them. Run this mode after you fix all the permanant errors. This mode reads only from the DLQ, and not from the Spanner change streams. If records processed from the `  severe  ` folder are moved to the `  retry  ` folder, the job retries them.

## Before you begin

  - Ensure network connectivity between your source MySQL database and your Google Cloud project, where your Dataflow jobs will run.

  - Allowlist Dataflow worker IP addresses on your destination MySQL instance.

  - Check that the MySQL credentials are correctly specified in the [`  source shards file  `](https://github.com/GoogleCloudPlatform/DataflowTemplates/blob/main/v2/spanner-to-sourcedb/README.md#sample-source-shards-file-for-MySQL) .

  - Check that your MySQL instance is online and running.

  - Verify that the MySQL user has `  INSERT  ` , `  UPDATE  ` , and `  DELETE  ` privileges on the MySQL database.

  - Verify that you have the required IAM permissions to run a Dataflow flex template. For more information, see [Build and run a flex template](/dataflow/docs/guides/templates/using-flex-templates#before-you-begin) .

  - Verify that the port `  12345  ` is open for communication between the Dataflow worker VMs.

### Required roles

  - To get the permissions that you need to launch the reverse replication, ask your administrator to grant you the following IAM roles on the instance:
    
      - [Cloud Spanner Database User](/iam/docs/roles-permissions/spanner#spanner.databaseUser) ( `  roles/spanner.databaseUser  ` )
      - [Dataflow Developer](/iam/docs/roles-permissions/dataflow#dataflow.developer) ( `  roles/dataflow.developer  ` )

<!-- end list -->

  - To ensure that the compute engine service account has the necessary permissions to launch the reverse replication, ask your administrator to grant the compute engine service account the following IAM roles on the instance:
    
    **Important:** You must grant these roles to the compute engine service account, *not* to your user account. Failure to grant the roles to the correct principal might result in permission errors.
    
      - [Cloud Spanner Database User](/iam/docs/roles-permissions/spanner#spanner.databaseUser) ( `  roles/spanner.databaseUser  ` )
      - [Secret Manager Secret Accessor](/iam/docs/roles-permissions/secretmanager#secretmanager.secretAccessor) ( `  roles/secretmanager.secretAccessor  ` )
      - [Secret Manager Viewer](/iam/docs/roles-permissions/secretmanager#secretmanager.viewer) ( `  roles/secretmanager.viewer  ` )

## Run reverse replication

To run the reverse replication, use the following steps:

1.  Upload the [session file](https://googlecloudplatform.github.io/spanner-migration-tool/reports.html#session-file-ending-in-sessionjson) to the Cloud Storage bucket.

2.  Create a Pub/Sub notification for the `  retry  ` folder of the DLQ directory. You can do this by creating a [Pub/Sub topic](/pubsub/docs/create-topic) and a [Pub/Sub subscription](/pubsub/docs/create-subscription) for that topic.

3.  Build and stage the Dataflow template. For more information, see [Building template](https://github.com/GoogleCloudPlatform/DataflowTemplates/blob/main/v2/spanner-to-sourcedb/README_Spanner_to_SourceDb.md#building-template) .

4.  Run the reverse replication Dataflow template using the following Google Cloud CLI command:
    
    ``` text
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
      --parameters "startTimestamp=START_TIMESTAMP" \
      --parameters "endTimestamp=END_TIMESTAMP" \
      --parameters "shadowTablePrefix=SHADOW_TABLE_PREFIX" \
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
    
      - `  project  ` : the Google Cloud project ID.
      - `  region  ` : the Google Cloud region.
      - `  template-file-gcs-location  ` : the path to the Cloud Storage file where you staged the Dataflow template.
      - `  changeStreamName  ` : the name of the Spanner change stream that the job reads from.
      - `  instanceId  ` : the Spanner instance ID.
      - `  databaseId  ` : the Spanner database ID.
      - `  spannerProjectId  ` : the project ID where your Spanner instances reside.
      - `  metadataInstance  ` : the instance that stores the metadata that the connector uses to control the consumption of change stream API data.
      - `  metadataDatabase  ` : the database that stores the metadata that the connector uses to control the consumption of change stream API data.
      - `  sourceShardsFilePath  ` : the path to the Cloud Storage file that contains the connection profile information for the source shards.
    
    The optional variables are described in the following list:
    
      - `  startTimestamp  ` : the timestamp from which to start reading changes. Defaults to empty.
      - `  endTimestamp  ` : the timestamp until which to read changes. If you don't provide a timestamp, the process reads changes indefinitely. Defaults to empty.
      - `  shadowTablePrefix  ` : the prefix for naming shadow tables. Defaults to `  shadow_  ` .
      - `  sessionFilePath  ` : the path to the session file in Cloud Storage that contains mapping information from the Spanner migration tool.
      - `  filtrationMode  ` : the mode that determines how to filter records based on criteria. Supported modes are `  none  ` or `  forward_migration  ` .
      - `  shardingCustomJarPath  ` : the location (path) in Cloud Storage of the custom JAR file that contains the logic for fetching the shard ID. Defaults to empty.
      - `  shardingCustomClassName  ` : the fully qualified class name that has the custom shard ID implementation. This field is mandatory if `  shardingCustomJarPath  ` is specified. Defaults to empty.
      - `  shardingCustomParameters  ` : a string containing any custom parameters to pass to the custom sharding class. Defaults to empty.
      - `  sourceDbTimezoneOffset  ` : the time zone offset from UTC for the source database. Example: +10:00. Default: +00:00.
      - `  dlqGcsPubSubSubscription  ` : the Pub/Sub subscription used in a Cloud Storage notification policy for the DLQ retry directory when running in `  regular  ` mode. Specify the name in the format `  projects/<PROJECT_ID>/subscriptions/<SUBSCRIPTION_ID>  ` . When you set this parameter, the system ignores `  deadLetterQueueDirectory  ` and `  dlqRetryMinutes  ` .
      - `  skipDirectoryName  ` : the directory where the system writes records that it skips during reverse replication. Default: `  skip  ` .
      - `  maxShardConnections  ` : the maximum number of connections allowed per shard. Default: 10000.
      - `  deadLetterQueueDirectory  ` : the path for storing the DLQ output. The default is a directory under the Dataflow job's temporary location.
      - `  dlqMaxRetryCount  ` : the maximum number of times the system retries temporary errors through the DLQ. Default: 500.
      - `  runMode  ` : the run mode type. Options are `  regular  ` or `  retryDLQ  ` . Use `  retryDLQ  ` to retry only the severe DLQ records. Default: `  regular  ` .
      - `  dlqRetryMinutes  ` : the number of minutes between DLQ retries. Default: 10.
      - `  sourceType  ` : the type of source database to reverse replicate to. Default: `  mysql  ` .
      - `  transformationJarPath  ` : the location (path) in Cloud Storage of the custom JAR file that contains the custom transformation logic for processing records in reverse replication. Defaults to empty.
      - `  transformationClassName  ` : the fully qualified class name that has the custom transformation logic. This field is mandatory if `  transformationJarPath  ` is specified. Defaults to empty.
      - `  transformationCustomParameters  ` : a string containing any custom parameters to pass to the custom transformation class. Defaults to empty.
      - `  filterEventsDirectoryName  ` : the directory where the system writes records that it skips during reverse replication. Default: `  skip  ` .

## What's next?

  - [`  Spanner_to_SourceDb  ` Dataflow template](https://github.com/GoogleCloudPlatform/DataflowTemplates/blob/main/v2/spanner-to-sourcedb/README_Spanner_to_SourceDb.md#building-template) .
