This page explains how to use the Kafka connector to consume and forward Spanner [change streams](/spanner/docs/change-streams) data.

## Core concepts

The following describes core concepts of the Kafka connector.

### Debezium

[Debezium](https://github.com/debezium/debezium) is an open source project that provides a low latency data streaming platform for change data capture.

### Kafka connector

The Kafka connector provides an abstraction over the Spanner API to publish Spanner change streams to Kafka. With this connector, you don't have to manage the change streams partition lifecycle, which is necessary when you [use the Spanner API directly](/spanner/docs/change-streams/details#query) .

The Kafka connector produces a change event for every data change record mod and sends change event records downstream into a separate Kafka topic for each change stream-tracked table. A [data change record mod](/spanner/docs/change-streams/details#data-change-records) represents a single modification (insert, update, or delete) that was captured. A single data change record can contain more than one mod.

### Kafka connector output

The Kafka connector forwards change streams records directly into a separate Kafka topic. The output topic name should be `  connector_name  ` . `  table_name  ` . If the topic doesn't exist, the Kafka connector automatically creates a topic under that name.

You can also configure [topic routing transformations](https://debezium.io/documentation/reference/stable/transformations/topic-routing.html) to re-route records into topics that you specify. If you want to use topic routing, disable the [low watermark](/spanner/docs/change-streams/use-kafka#low-watermark) functionality.

#### Record ordering

Records are ordered by commit timestamp per primary key in the Kafka topics. Records belonging to different primary keys do not have ordering guarantees. Records with the same primary key are stored in the same Kafka topic partition. If you want to process whole transactions, you can also use the [data change record's](/spanner/docs/change-streams/details#data-change-records) `  server_transaction_id  ` and `  number_of_records_in_transaction  ` fields to assemble a Spanner transaction.

#### Change events

The Kafka connector generates a data change event for each `  INSERT  ` , `  UPDATE  ` , and `  DELETE  ` operation. Each event contains a key and values for the changed row.

You can use [Kafka Connect converters](https://kafka.apache.org/24/javadoc/org/apache/kafka/connect/storage/Converter.html) to produce data change events in `  Protobuf  ` , `  AVRO  ` , `  JSON  ` , or `  JSON Schemaless  ` formats. If you use a Kafka Connect converter that produces schemas, the event contains separate schemas for the key and values. Otherwise, the event only contains the key and values.

The schema for the key never changes. The schema for the values is an amalgamation of all the columns the change stream has tracked since the connector start time.

If you configure the connector to produce JSON events, the output change event contains five fields:

  - The first `  schema  ` field specifies a Kafka Connect schema that describes the Spanner key schema.

  - The first `  payload  ` field has the structure described by the previous `  schema  ` field and contains the key for the row that was changed.

  - The second `  schema  ` field specifies the Kafka Connect schema that describes the schema for the changed row.

  - The second `  payload  ` field has the structure described by the previous `  schema  ` field and it contains the actual data for the row that was changed.

  - The `  source  ` field is a mandatory field that describes the source metadata for the event.

The following is an example of a data change event:

``` text
{
  // The schema for the Spanner key.
  "schema": {
    "type": "struct",
    "name": "customers.Key",
    "optional": false,
    "fields": [
      {
        "type": "int64",
        "optional": "false"
        "field": "false"
      }
    ]
  },
  // The value of the Spanner key.
  "payload": {
      "id": "1"
  },
  // The schema for the payload, which contains the before and after values
  // of the changed row. The schema for the payload contains all the
  // columns that the change stream has tracked since the connector start
  // time.
  "schema": { 
    "type": "struct",
    "fields": [
      {
        // The schema for the before values of the changed row.
        "type": "struct",
        "fields": [
            {
                "type": "int32",
                "optional": false,
                "field": "id"
            },
            {
                "type": "string",
                "optional": true,
                "field": "first_name"
            }
        ],
        "optional": true,
        "name": "customers.Value",
        "field": "before"
      },
      {
        // The schema for the after values of the changed row.
        "type": "struct",
        "fields": [
          {
            "type": "int32",
            "optional": false,
            "field": "id"
          },
          {
            "type": "string",
            "optional": false,
            "field": "first_name"
          }
        ],
          "optional": true,
          "name": "customers.Value",
          "field": "after"
        },
        {
          // The schema for the source metadata for the event.
          "type": "struct",
          "fields": [
            {
                "type": "string",
                "optional": false,
                "field": "version"
            },
            {
                "type": "string",
                "optional": false,
                "field": "connector"
            },
            {
                "type": "string",
                "optional": false,
                "field": "name"
            },
            {
                "type": "int64",
                "optional": false,
                "field": "ts_ms"
            },
            {
                "type": "boolean",
                "optional": true,
                "default": false,
                "field": "snapshot"
            },
            {
                "type": "string",
                "optional": false,
                "field": "db"
            },
            {
                "type": "string",
                "optional": false,
                "field": "sequence"
            },
            {
                "type": "string",
                "optional": false,
                "field": "project_id"
            },
            {
                "type": "string",
                "optional": false,
                "field": "instance_id"
            },
            {
                "type": "string",
                "optional": false,
                "field": "database_id"
            },
            {
                "type": "string",
                "optional": false,
                "field": "change_stream_name"
            },
            {
                "type": "string",
                "optional": true,
                "field": "table"
            }
            {
                "type": "string",
                "optional": true,
                "field": "server_transaction_id"
            }
            {
                "type": "int64",
                "optional": true,
                "field": "low_watermark"
            }
            {
                "type": "int64",
                "optional": true,
                "field": "read_at_timestamp"
            }
            {
                "type": "int64",
                "optional": true,
                "field": "number_of_records_in_transaction"
            }
            {
                "type": "string",
                "optional": true,
                "field": "transaction_tag"
            }
            {
                "type": "boolean",
                "optional": true,
                "field": "system_transaction"
            }
            {
                "type": "string",
                "optional": true,
                "field": "value_capture_type"
            }
            {
                "type": "string",
                "optional": true,
                "field": "partition_token"
            }
            {
                "type": "int32",
                "optional": true,
                "field": "mod_number"
            }
            {
                "type": "boolean",
                "optional": true,
                "field": "is_last_record_in_transaction_in_partition"
            }
            {
                "type": "int64",
                "optional": true,
                "field": "number_of_partitions_in_transaction"
            }
          ],
          "optional": false,
          "name": "io.debezium.connector.spanner.Source",
          "field": "source"
        },
      ]
      {
        "type": "string",
        "optional": false,
        "field": "op"
      },
      {
        "type": "int64",
        "optional": true,
        "field": "ts_ms"
      }
    ],
    "optional": false,
    "name": "connector_name.customers.Envelope"
  },
  "payload": {
    // The values of the row before the event.
    "before": null,
    // The values of the row after the event.
    "after": { 
        "id": 1,
        "first_name": "Anne",
    }
  },
  // The source metadata.
  "source": {
    "version": "{debezium-version}",
    "connector": "spanner",
    "name": "spanner_connector",
    "ts_ms": 1670955531785,
    "snapshot": "false",
    "db": "database",
    "sequence": "1",
    "project_id": "project",
    "instance_id": "instance",
    "database_id": "database",
    "change_stream_name": "change_stream",
    "table": "customers",
    "server_transaction_id": "transaction_id",
    "low_watermark": 1670955471635,
    "read_at_timestamp": 1670955531791,
    "number_records_in_transaction": 2,
    "transaction_tag": "",
    "system_transaction": false,
    "value_capture_type": "OLD_AND_NEW_VALUES",
    "partition_token": "partition_token",
    "mod_number": 0,
    "is_last_record_in_transaction_in_partition": true,
    "number_of_partitions_in_transaction": 1
  },
  "op": "c", 
  "ts_ms": 1559033904863 //
}
```

#### Low watermark

The low watermark describes the time T at which the Kafka connector is guaranteed to have streamed out and published to a Kafka topic all events with timestamp \< T.

You can enable the low watermark in the Kafka connector using the `  gcp.spanner.low-watermark.enabled  ` parameter. This parameter is disabled by default. If the low watermark is enabled, the `  low_watermark  ` field in the change stream data change record is populated with the Kafka connector's current low watermark timestamp.

If there are no records being produced, the Kafka connector sends periodic watermark "heartbeats" to the Kafka output topics detected by the connector.

These watermark heartbeats are records that are empty except for the `  low_watermark  ` field. You can then use the low watermark to perform time-based aggregations. For example, you can use the low watermark to order events by commit timestamp across primary keys.

#### Metadata topics

The Kafka connector, as well as the [Kafka Connect](https://kafka.apache.org/documentation/#connect) framework, creates several metadata topics to store connector-related information. It is not advisable to modify either the configuration or the content of these metadata topics.

The following are the metadata topics:

  - `  _consumer_offsets  ` : A topic automatically created by Kafka. Stores consumer offsets for consumers created in the Kafka connector.
  - `  _kafka-connect-offsets  ` : A topic automatically created by Kafka Connect. Stores the connector offsets.
  - `  _sync_topic_spanner_connector_connectorname  ` : A topic automatically created by the connector. Stores metadata regarding change stream partitions.
  - `  _rebalancing_topic_spanner_connector_connectorname  ` : A topic automatically created by the connector. Used to determine connector task aliveness.
  - `  _debezium-heartbeat.connectorname  ` : A topic used to process Spanner change stream heartbeats.

### Kafka connector runtime

The following describes the Kafka connector runtime.

#### Scalability

The Kafka connector is horizontally scalable and runs on one or more tasks spread out among multiple Kafka Connect workers.

#### Message delivery guarantees

The Kafka connector supports at-least-once delivery guarantee.

#### Fault tolerance

The Kafka connector is tolerant of failures. As the Kafka connector reads changes and produces events, it records the last commit timestamp processed for each change stream partition. If the Kafka connector stops for any reason (including communication failures, network problems, or software failures), upon restart the Kafka connector continues streaming records where it last left off.

The Kafka connector reads the information schema at the Kafka connector's start timestamp to retrieve schema information. By default, Spanner cannot read the information schema at read timestamps before the [version retention period](/spanner/docs/reference/rest/v1/projects.instances.databases#Database.FIELDS.version_retention_period) , which defaults to one hour. If you want to start the connector from earlier than one hour into the past, you must increase the database's version retention period.

## Set up the Kafka connector

### Create a change stream

For details on how to create a change stream, see [Create a change stream](/spanner/docs/change-streams/manage#create) . To continue with the next steps, a Spanner instance with a change stream configured is required.

Note that if you want both changed and unchanged columns to be returned on each data change event, use the value capture type `  NEW_ROW  ` . For more information, see [value capture type](/spanner/docs/change-streams#value-capture-type) .

### Install the Kafka connector JAR

With [Zookeeper](https://zookeeper.apache.org) , [Kafka](http://kafka.apache.org/) , and [Kafka Connect](https://kafka.apache.org/documentation/#connect) installed, the remaining tasks to deploy a Kafka connector are to download the [connector's plug-in archive](https://repo1.maven.org/maven2/io/debezium/debezium-connector-spanner) , extract the JAR files into your Kafka Connect environment, and add the directory with the JAR files to [Kafka Connect's `  plugin.path  `](https://kafka.apache.org/documentation/#connectconfigs) . You then need to restart your Kafka Connect process to pick up the new JAR files.

If you are working with immutable containers, you can pull images from [Debezium's Container images](https://quay.io/organization/debezium) for Zookeeper, Kafka and Kafka Connect. The Kafka Connect image has the Spanner connector pre-installed.

For more information on how to install Debezium-based Kafka connector JARs, see [Installing Debezium](https://debezium.io/documentation/reference/stable/install.html) .

### Configure the Kafka connector

The following is an example of the configuration for a Kafka connector that connects to a change stream called `  changeStreamAll  ` in the database `  users  ` in instance `  test-instance  ` and project `  test-project  ` .

``` json
"name": "spanner-connector",
"config": {
    "connector.class": "io.debezium.connector.spanner.SpannerConnector",
    "gcp.spanner.project.id": "test-project",
    "gcp.spanner.instance.id": "test-instance",
    "gcp.spanner.database.id": "users",
    "gcp.spanner.change.stream": "changeStreamAll",
    "gcp.spanner.credentials.json": "{"client_id": user@example.com}",
    "gcp.spanner.database.role": "cdc-role",
    "tasks.max": "10"
}
```

This configuration contains the following:

  - The name of the connector when registered with a Kafka Connect service.

  - The name of this Spanner connector class.

  - The Project ID.

  - The Spanner Instance ID.

  - The Spanner Database ID.

  - The change stream name.

  - The JSON object for the service account key.

  - (Optional) The Spanner database role to use.

  - The maximum number of tasks.

For a complete list of connector properties, see [Kafka connector configuration properties](/spanner/docs/change-streams/use-kafka#kafka-connector-configuration-properties) .

### Add the connector configuration to Kafka Connect

To start running a Spanner connector:

1.  Create a configuration for the Spanner connector.

2.  Use the [Kafka Connect REST API](https://kafka.apache.org/documentation/#connect_rest) to add that connector configuration to your Kafka Connect cluster.

You can send this configuration with a `  POST  ` command to a running Kafka Connect service. By default, the Kafka Connect service runs on port `  8083  ` . The service records the configuration and start the connector task that connects to the Spanner database and streams change event records to Kafka topics.

The following is an example `  POST  ` command:

``` text
POST /connectors HTTP/1.1
Host: http://localhost:8083
Accept: application/json
{
  "name": "spanner-connector"
  "config": {
      "connector.class": "io.debezium.connector.spanner.SpannerConnector",
      "gcp.spanner.project.id": "test-project",
      "gcp.spanner.instance.id": "test-instance",
      "gcp.spanner.database.id": "users",
      "gcp.spanner.change.stream": "changeStreamAll",
      "gcp.spanner.credentials.json": "{\"client_id\": \"XXXX\".... }",
      "heartbeat.interval.ms": "100",
      "tasks.max": "10"
  }
}
```

Example successful response:

``` text
HTTP/1.1 201 Created
Content-Type: application/json
{
    "name": "spanner-connector",
    "config": {
        "connector.class": "io.debezium.connector.spanner.SpannerConnector",
        "gcp.spanner.project.id": "test-project",
        "gcp.spanner.instance.id": "test-instance",
        "gcp.spanner.database.id": "users",
        "gcp.spanner.change.stream": "changeStreamAll",
        "gcp.spanner.credentials.json": "{\"client_id\": \"XXXX\".... }",
        "heartbeat.interval.ms": "100",
        "tasks.max": "10"
    },
    "tasks": [
        { "connector": "spanner-connector", "task": 1 },
        { "connector": "spanner-connector", "task": 2 },
        { "connector": "spanner-connector", "task": 3 }
    ]
}
```

### Update the Kafka connector configuration

To update the connector configuration, send a `  PUT  ` command to the running Kafka Connect service with the same connector name.

Assume that we have a connector running with the configuration from the previous section. The following is an example `  PUT  ` command:

``` text
PUT /connectors/spanner-connector/config HTTP/1.1
Host: http://localhost:8083
Accept: application/json
{
    "connector.class": "io.debezium.connector.spanner.SpannerConnector",
    "gcp.spanner.project.id": "test-project",
    "gcp.spanner.instance.id": "test-instance",
    "gcp.spanner.database.id": "users",
    "gcp.spanner.change.stream": "changeStreamAll",
    "gcp.spanner.credentials.json": "{\"client_id\": \"XXXX\".... }",
    "heartbeat.interval.ms": "100",
    "tasks.max": "10"
}
```

Example successful response:

``` text
HTTP/1.1 200 OK
Content-Type: application/json
{
    "connector.class": "io.debezium.connector.spanner.SpannerConnector",
    "tasks.max": "10",
    "gcp.spanner.project.id": "test-project",
    "gcp.spanner.instance.id": "test-instance",
    "gcp.spanner.database.id": "users",
    "gcp.spanner.change.stream": "changeStreamAll",
    "gcp.spanner.credentials.json": "{\"client_id\": \"XXXX\".... }",
    "heartbeat.interval.ms": "100",
    "tasks.max": "10"
}
```

### Stop the Kafka connector

To stop the connector, send a `  DELETE  ` command to the running Kafka Connect service with the same connector name.

Assume that we have a connector running with the configuration from the previous section. The following is an example `  DELETE  ` command:

``` text
DELETE /connectors/spanner-connector HTTP/1.1
Host: http://localhost:8083
```

Example successful response:

``` text
HTTP/1.1 204 No Content
```

### Monitor the Kafka connector

In addition to standard Kafka Connect and [Debezium](https://debezium.io/documentation/reference/stable/operations/monitoring.html) metrics, the Kafka connector exports its own metrics:

  - `  MilliSecondsLowWatermark  ` : The current low watermark of the connector task in milliseconds. The low watermark describes the time T at which the connector is guaranteed to have streamed out all events with timestamp \< T

  - `  MilliSecondsLowWatermarkLag  ` : The lag of the low watermark behind the current time in milliseconds. streamed out all events with timestamp \< T

  - `  LatencyLowWatermark<Variant>MilliSeconds  ` : The lag of the low watermark behind the current time in milliseconds. P50, P95, P99, Average, Min, and Max variants are provided.

  - `  LatencySpanner<Variant>MilliSeconds  ` : The Spanner-commit-timestamp-to-connector-read latency. P50, P95, P99, Average, Min, Max variants are provided.

  - `  LatencyReadToEmit<Variant>MilliSeconds  ` : The Spanner-read-timestamp-to-connector-emit latency. P50, P95, P99, Average, Min, and Max variants are provided.

  - `  LatencyCommitToEmit<Variant>tMilliSeconds  ` : The Spanner-commit-timestamp-to-connector-emit latency. P50, P95, P99, Average, Min, and Max variants are provided.

  - `  LatencyCommitToPublish<Variant>MilliSeconds  ` : The Spanner-commit-timestamp-to Kafka-publish-timestamp latency. P50, P95, P99, Average, Min, Max variants are provided.

  - `  NumberOfChangeStreamPartitionsDetected  ` : The total number of partitions detected by the current connector task.

  - `  NumberOfChangeStreamQueriesIssued  ` : The total number of change stream queries issued by the current task.

  - `  NumberOfActiveChangeStreamQueries  ` : The active number of change stream queries detected by the current connector task.

  - `  SpannerEventQueueCapacity  ` : The total capacity of [`  StreamEventQueue  `](https://github.com/debezium/debezium-connector-spanner/blob/main/src/main/java/io/debezium/connector/spanner/StreamEventQueue.java) , a queue that stores elements received from change stream queries.

  - `  SpannerEventQueueCapacity  ` : The remaining `  StreamEventQueue  ` capacity.

  - `  TaskStateChangeEventQueueCapacity  ` : The total capacity of [`  TaskStateChangeEventQueue  `](https://github.com/debezium/debezium-connector-spanner/blob/main/src/main/java/io/debezium/connector/spanner/task/TaskStateChangeEventProcessor.java#L34) , a queue that stores events happening in the connector.

  - `  RemainingTaskStateChangeEventQueueCapacity  ` : The remaining `  TaskStateChangeEventQueue  ` capacity.

  - `  NumberOfActiveChangeStreamQueries  ` : The active number of change stream queries detected by the current connector task.

### Kafka connector configuration properties

The following are required configuration properties for the connector:

  - `  name  ` : Unique name for the connector. Attempting to register again with the same name causes failure. This property is required by all Kafka Connect connectors.

  - `  connector.class  ` : The name of the Java class for the connector. Always use a value of `  io.debezium.connector.spanner.SpannerConnector  ` for the Kafka connector.

  - `  tasks.max  ` : The maximum number of tasks that should be created for this connector.

  - `  gcp.spanner.project.id  ` : The project ID

  - `  gcp.spanner.instance.id  ` : The Spanner instance ID

  - `  gcp.spanner.database.id  ` : The Spanner database ID

  - `  gcp.spanner.change.stream  ` : The Spanner change stream name

  - `  gcp.spanner.credentials.json  ` : The service account key JSON object.

  - `  gcp.spanner.credentials.path  ` : The file path to the service account key JSON object. Required if the above field isn't provided.

  - `  gcp.spanner.database.role  ` : The Spanner database role to use. This is required only when the change stream is secured with fine-grained access control. The database role must have the `  SELECT  ` privilege on the change stream and the `  EXECUTE  ` privilege on the change stream's read function. For more information, see [Fine-grained access control for change streams](../fgac-change-streams) .

The following advanced configuration properties have defaults that work in most situations and therefore rarely need to be specified in the connector's configuration:

  - `  gcp.spanner.low-watermark.enabled  ` : Indicates whether the low watermark is enabled for the connector. Default is false.

  - `  gcp.spanner.low-watermark.update-period.ms  ` : The interval at which the low watermark is updated. Default is 1000 ms.

  - `  heartbeat.interval.ms  ` : The Spanner heartbeat interval. The default is 300000 (five minutes).

  - `  gcp.spanner.start.time  ` : The connector start time. Defaults to the current time.

  - `  gcp.spanner.end.time  ` : The connector end time. Defaults to infinity.

  - `  tables.exclude.list  ` : The tables to exclude change events for. Defaults to empty.

  - `  tables.include.list  ` : The tables to include change events for. If not populated, all tables are included. Defaults to empty.

  - `  gcp.spanner.stream.event.queue.capacity  ` : The Spanner event queue capacity. Defaults to 10000.

  - `  connector.spanner.task.state.change.event.queue.capacity  ` : The task state change event queue capacity. Defaults to 1000.

  - `  connector.spanner.max.missed.heartbeats  ` : The maximum number of missed heartbeats for a change stream query before an exception is thrown. Defaults to 10.

  - `  scaler.monitor.enabled  ` : Indicates whether task autoscaling is enabled. Defaults to false.

  - `  tasks.desired.partitions  ` : The preferred number of change streams partitions per task. This parameter is needed for task autoscaling. Defaults to 2.

  - `  tasks.min  ` : The minimum number of tasks. This parameter is needed for task autoscaling. Defaults to 1.

  - `  connector.spanner.sync.topic  ` : The name for the sync topic, an internal connector topic used to store communication between tasks. Defaults to `  _sync_topic_spanner_connector_connectorname  ` if the user did not provide a name.

  - `  connector.spanner.sync.poll.duration  ` : The poll duration for the sync topic. Defaults to 500 ms.

  - `  connector.spanner.sync.request.timeout.ms  ` : The timeout for requests to the sync topic. Defaults to 5000 ms.

  - `  connector.spanner.sync.delivery.timeout.ms  ` : The timeout for publishing to the sync topic. Defaults to 15000 ms.

  - `  connector.spanner.sync.commit.offsets.interval.ms  ` : The interval at which offsets are committed for the sync topic. Defaults to 60000 ms.

  - `  connector.spanner.sync.publisher.wait.timeout  ` : The interval at which messages are published to the sync topic. Defaults to 5 ms.

  - `  connector.spanner.rebalancing.topic  ` : The name of the rebalancing topic. The rebalancing topic is an internal connector topic used to determine task aliveness. Defaults to `  _rebalancing_topic_spanner_connector_connectorname  ` if the user did not provide a name.

  - `  connector.spanner.rebalancing.poll.duration  ` : The poll duration for the rebalancing topic. Defaults to 5000 ms.

  - `  connector.spanner.rebalancing.commit.offsets.timeout  ` : The timeout for committing offsets for the rebalancing topic. Defaults to 5000 ms.

  - `  connector.spanner.rebalancing.commit.offsets.interval.ms  ` : The interval at which offsets are committed for the sync topic. Defaults to 60000 ms.

  - `  connector.spanner.rebalancing.task.waiting.timeout  ` : The duration of time a task waits before processing a rebalancing event. Defaults to 1000 ms.

For an even more detailed list of the configurable connector properties, see the [GitHub repository](https://github.com/debezium/debezium-connector-spanner/blob/main/src/main/java/io/debezium/connector/spanner/config/BaseSpannerConnectorConfig.java) .

## Limitations

  - The connector does not support streaming [snapshot events](https://debezium.io/blog/2021/10/07/incremental-snapshots/) .

  - If watermarking is enabled in the connector, you cannot configure [Debezium topic routing transformations](https://debezium.io/documentation/reference/stable/transformations/topic-routing.html) .
