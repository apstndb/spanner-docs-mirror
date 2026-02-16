A **change stream** watches and streams out a Spanner database's *data changes* —inserts, updates, and deletes—in near real-time.

This page offers a high-level overview of Spanner change streams: what they do, and how they work. To learn how to create and manage change streams in your database and connect them with other services, follow the links in [What's Next](#whats-next) .

## Purpose of change streams

Change streams provide a flexible, scalable way to stream data changes to other services. Common use cases include:

  - Replicating Spanner data changes to a data warehouse, such as [BigQuery](/bigquery) , for analytics.

  - Triggering application logic based on data changes sent to a message queue, such as [Pub/Sub](/pubsub) .

  - Storing data changes in [Cloud Storage](/storage) , for compliance or archival purposes.

## Change stream configuration

Spanner treats change streams as schema objects, much like tables and indexes. As such, you [create, modify, and delete change streams using DDL statements](/spanner/docs/change-streams/manage) , and you can [view a database's change streams](/spanner/docs/change-streams/manage#view) just like other DDL-managed schema objects.

You can configure a change stream to watch data changes across an entire database, or limit its scope to specific tables and columns. A database can have multiple change streams, and a particular table or column can have multiple streams watching it, [within limits](/spanner/quotas#change-streams) .

You can optionally configure a change stream with the following:

  - [Specify the data retention period](/spanner/docs/change-streams#data-retention) to override the default, one-day retention period.
  - [Specify the value capture type](/spanner/docs/change-streams#value-capture-type) to override the default value capture type `  OLD_AND_NEW_VALUES  ` .
  - [Apply a TTL-based deletes filter](/spanner/docs/change-streams#ttl-filter) to filter out TTL-based deletes from your change streams.
  - [Apply a table modifications filter](/spanner/docs/change-streams#mod-type-filter) to exclude all `  INSERT  ` , `  UPDATE  ` , or `  DELETE  ` table modifications.
  - [Enable transaction-level records exclusion](/spanner/docs/change-streams#transaction-exclusion) to exclude certain transactions from your change streams.

Issuing the DDL that creates a change stream starts a [long-running operation](/spanner/docs/manage-long-running-operations) . When it completes, the new change stream immediately begins to watch the tables and columns assigned to it.

### Implicitly watching tables and columns

Change streams that watch an entire table implicitly watch all the columns in that table, even when that table definition is updated. For example, when you add new columns to that table, the change stream automatically begins to watch those new columns, without requiring any modification to that change stream's configuration. Similarly, the change stream automatically stops watching any columns that are dropped from that table.

Whole-database change streams work the same way. They implicitly watch every column in every table, automatically watching any tables or columns added after the change stream's creation, and ceasing to watch any tables or columns dropped.

### Explicitly watching tables and columns

If you configure a change stream to watch only particular columns in a table, and you later add columns to that table, the change stream will not begin to watch those columns unless you reconfigure that change stream to do so.

The database's schema treats change streams as dependent objects of any columns or tables that they explicitly watch. Before you can drop any such column or table, you must manually remove it from the configuration of any change stream explicitly watching it.

### Types of data changes that change streams watch

The data changes that a change stream watches include all inserts, updates, and deletes made to the tables and columns that it watches. These changes can come from:

  - [DML statements](/spanner/docs/dml-tasks)

  - [Mutations](/spanner/docs/modify-mutation-api)

  - Cascading deletes on [interleaved child tables](/spanner/docs/schema-and-data-model#creating-interleaved-tables)

  - Deletes resulting from [time to live](/spanner/docs/ttl) rules

Change streams can watch data changes only in user-created columns and tables. They don't watch indexes, views, other change streams, or system tables such as the information schema or statistics tables. Change streams don't watch generated columns unless the column is part of the primary key. Primary key columns are always tracked.

Furthermore, change streams don't watch schema changes or any data changes that directly result from schema changes, other than backfills for [default values](/spanner/docs/primary-key-default-value) . For example, a change stream watching an entire database doesn't consider and record a table deletion as a data change, even though this action deletes all of that table's data from the database.

## How Spanner writes and stores change streams

Every time Spanner detects a data change in a column being watched by a change stream, it writes a *data change record* to [its internal storage](/spanner/docs/change-streams/details#data-change-records) . The data change write and the data change record are written within the same transaction. Spanner co-locates both of these writes so they are processed by the same server, minimizing write processing. The transaction is then replicated across the database's replicas, subjecting it to storage and replication costs. For more information, see [Spanner pricing](https://cloud.google.com/spanner/pricing) .

### Content of a data change record

Every data change record written by a change stream includes the following information about the data change:

  - The name of the affected table

  - The names, values, and data types of the primary keys identifying the changed row

  - The names and data types of the changed row's columns that were captured based on the change stream definition.

  - The old values of the row's columns. The availability of the old values and the content they track, which can be either the modified columns only or the entire tracked row, depends on the user-configured [value capture type](/spanner/docs/change-streams#value-capture-type) .

  - The new values of the row's columns. The availability of the new values and the content they track depends on the user-configured [value capture type](/spanner/docs/change-streams#value-capture-type) .

  - The modification type (insert, update, or delete)

  - The commit timestamp

  - The transaction ID

  - The record sequence number

  - The data change record's value capture type.

For a deeper look at the structure of data change records, see [Data change records](/spanner/docs/change-streams/details#data-change-records) .

**Note:** A change stream's *value capture type* configuration option controls the way that it records a changed row's values. `  OLD_AND_NEW_VALUES  ` is this option's default setting. For more information, see [value capture type](/spanner/docs/change-streams/details#value-capture-type) .

### Data retention

A change stream retains its data change records for a period of time between one and thirty days. [You can use DDL](/spanner/docs/change-streams/manage) to specify a data-retention limit other than the one-day default when initially creating a change stream, or adjust it at any future time. Note that reducing a change stream's data retention limit will make all historical change data older than the new limit immediately and permanently unavailable to that change stream's readers.

This data retention period presents a trade-off; a longer retention period carries greater storage demands on the stream's database.

### Value capture type

A change stream's *value capture type* configuration option controls the way that it stores a changed row's values. [You can use DDL](/spanner/docs/change-streams/manage) to specify one of the following value capture types for a change stream:

  - `  OLD_AND_NEW_VALUES  ` : Captures both old and new values of a row's modified columns.

  - `  NEW_VALUES  ` : Captures only the new values of the non-key columns, but no old values.

  - `  NEW_ROW  ` : Captures all new values of watched columns, both modified and unmodified, whenever any of those columns change. No old values are captured.

  - `  NEW_ROW_AND_OLD_VALUES  ` : Captures all new values for both modified and unmodified columns, and old values for modified columns.

### Exclude time-to-live based deletes

In Spanner, [time-to-live (TTL)](/spanner/docs/ttl) lets you set policies to periodically delete data from Spanner tables. By default, change streams include all TTL-based deletes. You can use `  exclude_ttl_deletes  ` to set your change stream to exclude TTL-based deletes. When you set this filter to exclude TTL-based deletes, only future TTL-based deletes are excluded from your change stream.

The default value for this filter is `  false  ` . To exclude TTL-based deletes, set the filter to `  true  ` . You can either [add the filter when you create a change stream](/spanner/docs/change-streams/manage#filter-ttl-deletes) or [modify an existing change stream to include the filter](/spanner/docs/change-streams/manage#modify-ttl-deletes) .

### Table modification type

By default, change streams include all table modifications, such as inserts, updates, and deletes. You can filter one or more of these table modifications from your change stream's scope using the following available filter options:

  - `  exclude_insert  ` : exclude all `  INSERT  ` table modifications
  - `  exclude_update  ` : exclude all `  UPDATE  ` table modifications
  - `  exclude_delete  ` : exclude all `  DELETE  ` table modifications

The default value for these filters is `  false  ` . To exclude a specific type of table modification, set the filter to `  true  ` . You can set one or more filters at the same time.

You can [add a filter for a table modification type](/spanner/docs/change-streams/manage#filter-mod-type) when you create a change stream or [modify the filter for a table modification type](/spanner/docs/change-streams/manage#modify-mod-type) for an existing change stream.

### Transaction-level records exclusion

By default, a change stream watches all write transactions in the database because the `  allow_txn_exclusion  ` DDL option is set to `  false  ` . You can set the `  allow_txn_exclusion  ` option to `  true  ` to enable your change stream to ignore records from specified write transactions. If you don't set this option to `  true  ` , then all write transactions are watched, even if you use the `  exclude_txn_from_change_streams  ` parameter in your write transaction.

You can either [enable this option when you create a change stream](/spanner/docs/change-streams/manage#enable-transaction-exclusion) or [modify an existing change stream](/spanner/docs/change-streams/manage#modify-allow-transaction-exclusion) .

#### Exclude write transaction from change streams

To exclude a write transaction from change streams, you must set the `  exclude_txn_from_change_streams  ` parameter to `  true  ` . This parameter is part of the [`  TransactionOptions  `](/spanner/docs/reference/rest/v1/TransactionOptions) and [`  BatchWriteRequest  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/batchWrite) methods. The default value for this parameter is `  false  ` . You can set this parameter with the RPC API, REST API, or using the client libraries. For more information, see [Specify a write transaction to be excluded from change streams](/spanner/docs/change-streams/manage#specify-write-transaction-exclusion) .

You can't set this parameter to `  true  ` for read-only transactions. If you do this, then the API returns an invalid argument error.

For change streams monitoring columns modified by transactions, when `  exclude_txn_from_change_streams  ` is set to `  true  ` , two scenarios are possible:

  - If the DDL option `  allow_txn_exclusion  ` is set to `  true  ` , then the updates made within this transaction aren't recorded in the change stream.
  - If you don't set the DDL option `  allow_txn_exclusion  ` or if it's set to `  false  ` , then the updates made within this transaction are recorded in the change stream.

If you don't set the `  exclude_txn_from_change_streams  ` option or if it's set to `  false  ` , then any change streams monitoring columns modified by transactions will capture the updates made within that transaction.

## Reading change streams

Spanner offers multiple ways to read a change stream's data:

  - Through Dataflow, using the Apache Beam SpannerIO connector. This is our recommended solution for most change stream applications. Google also provides Dataflow templates for common use cases.

  - Directly, using the Spanner API. This trades away the abstraction and capabilities of Dataflow pipelines for maximum speed and flexibility.

  - Through using the Debezium-based Kafka connector for Spanner change streams. This connector streams change records directly into Kafka topics.

  - Using Datastream to directly stream your changes to BigQuery, BigLake Iceberg tables, or Cloud Storage.

You can provide partial isolation for change streams reads by using directed reads. Directed reads can help to minimize impact on transactional workloads in your database. You can use the Spanner API to route change streams reads to a specific replica type or region within a multi-region instance configuration or a custom regional configuration with optional read-only region(s). For more information, see [directed reads](/spanner/docs/directed-reads) .

### Using Dataflow

Use the [Apache Beam SpannerIO connector](https://beam.apache.org/releases/javadoc/current/org/apache/beam/sdk/io/gcp/spanner/SpannerIO.html) to [build Dataflow pipelines](/spanner/docs/change-streams/use-dataflow) that read from change streams. After you configure the connector with details about a particular change stream, it automatically outputs new data change records into a single, unbounded [`  PCollection  `](/dataflow/docs/concepts/beam-programming-model#concepts) data set, ready for further processing by subsequent transforms in the Dataflow pipeline.

Dataflow uses windowing functions to divide unbounded collections into logical components, or windows. As a result, Dataflow provides near real-time streaming when reading from change streams.

Google provides templates that let you rapidly build Dataflow pipelines for common change stream use cases, including sending all of a stream's data changes [to a BigQuery](/dataflow/docs/guides/templates/provided-streaming#cloud-spanner-change-streams-to-bigquery) dataset, or copying them [to a Cloud Storage bucket](/dataflow/docs/guides/templates/provided-streaming#cloud-spanner-change-streams-to-cloud-storage) .

For a more detailed overview of how change streams and Dataflow work together, see [Build change streams connections with Dataflow](/spanner/docs/change-streams/use-dataflow) .

### Using the API

As an alternative to using Dataflow to build change stream pipelines, you can instead write code that uses the Spanner API to read a change stream's records directly. This lets you read data change records in the same way that the SpannerIO connector does, by providing the lowest possible latencies when reading change stream data instead of providing the flexibility of Dataflow.

To learn more, see [Query change streams](/spanner/docs/change-streams/details#query) . For a more detailed discussion on how to query change streams and interpret the records returned, see [Change streams partitions, records, and queries](/spanner/docs/change-streams/details) .

### Using the Kafka connector

The Kafka connector directly outputs change stream records into a Kafka topic. It abstracts away the details of querying change streams using the Spanner API.

To learn more about how change streams and the Kafka connector work together, see [Build change streams connections with the Kafka connector](/spanner/docs/change-streams/use-kafka) .

### Using Datastream

Use Datastream, a serverless and easy-to-use change data capture (CDC) and replication service available in Google Cloud. Datastream supports Spanner change streams, and lets you read from and stream your change data into various destinations.

To learn more about how Datastream supports Spanner for reading and streaming change data, see [Spanner as a source](/datastream/docs/sources-spanner) .

## Limits

There are several limits on change streams, including the maximum number of change streams a database can have, and the maximum number of streams that can watch a single column. For a full list, see [Change stream limits](/spanner/quotas#change-streams) .

## Permissions

Change streams uses the following:

  - Creating, updating, or dropping change streams requires `  spanner.databases.updateDdl  ` .

  - Reading a change stream's data requires `  spanner.databases.select  ` .

If using the SpannerIO connector, then the owner of the Dataflow job that reads change stream data requires additional Identity and Access Management (IAM) permissions, either on your application database or on a separate metadata database; see [Create a metadata database](/spanner/docs/change-streams/use-dataflow#metadata) .

## What's next

  - Learn the DDL syntax to [create and manage change streams](/spanner/docs/change-streams/manage) .

  - Use change streams and templates to replicate changes from Spanner [to BigQuery](/dataflow/docs/guides/templates/provided-streaming#cloud-spanner-change-streams-to-bigquery) or [to Cloud Storage](/dataflow/docs/guides/templates/provided-streaming#cloud-spanner-change-streams-to-cloud-storage) .

  - Learn more about [building Dataflow pipelines to process change stream data](/spanner/docs/change-streams/use-dataflow) .

  - Further explore [change streams details](/spanner/docs/change-streams/details) , including more details on the change stream architecture, how to query change streams using the API and interpret the records returned.

  - Learn more about [using the Kafka connector to process change stream data](/spanner/docs/change-streams/use-kafka) .
