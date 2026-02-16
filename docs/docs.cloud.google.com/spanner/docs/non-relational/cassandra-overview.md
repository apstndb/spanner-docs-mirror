This page compares [Apache Cassandra](https://cassandra.apache.org/) and Spanner architecture as well as helps you understand the capabilities and limitations of the Spanner Cassandra interface. It assumes you're familiar with Cassandra and want to migrate existing applications or design new applications while using Spanner as your database.

Cassandra and Spanner are both large-scale distributed databases built for applications requiring high scalability and low latency. While both databases can support demanding NoSQL workloads, Spanner provides advanced features for data modeling, querying, and transactional operations. For more information about how Spanner meets NoSQL database criteria, see [Spanner for non-relational workloads](/spanner/docs/non-relational/overview) .

## Core concepts

This section compares key Cassandra and Spanner concepts.

### Terminology

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Cassandra</strong></th>
<th><strong>Spanner</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Cluster</strong></td>
<td><strong>Instance</strong><br />
<br />
A Cassandra cluster is equivalent to a Spanner <a href="/spanner/docs/instances">instance</a> - a collection of servers and storage resources. Because Spanner is a managed service, you don't have to configure the underlying hardware or software. You only need to specify the amount of nodes you want to reserve for your instance or use <a href="/spanner/docs/autoscaling-overview">autoscaling</a> to automatically scale the instance. An instance acts like a container for your databases. You also choose the data replication topology ( <a href="/spanner/docs/instance-configurations">regional, dual-region, or multi-region</a> ) at the instance level.</td>
</tr>
<tr class="even">
<td><strong>Keyspace</strong></td>
<td><strong>Database</strong><br />
<br />
A Cassandra keyspace is equivalent to a Spanner <a href="/spanner/docs/databases">database</a> , which is a collection of tables and other schema elements (for example, indexes and roles). Unlike a keyspace, you don't need to configure the replication location. Spanner automatically replicates your data to the region designated in your instance.</td>
</tr>
<tr class="odd">
<td><strong>Table</strong></td>
<td><strong>Table</strong><br />
<br />
In both Cassandra and Spanner, tables are a collection of rows identified by a primary key specified in the table schema.</td>
</tr>
<tr class="even">
<td><strong>Partition</strong></td>
<td><strong>Split</strong><br />
<br />
Both Cassandra and Spanner scale by sharding data. In Cassandra, each shard is called a partition, while in Spanner, each shard is called a split. Cassandra uses hash-partitioning, which means that each row is independently assigned to a storage node based on a hash of the primary key. Spanner is range-sharded, which means that rows that are contiguous in the primary key keyspace are contiguous in storage as well (except at split boundaries). Spanner takes care of splitting and merging based on load and storage, and this is transparent to the application. The key implication is that unlike Cassandra, range scans over a prefix of the primary key is an efficient operation in Spanner.</td>
</tr>
<tr class="odd">
<td><strong>Row</strong></td>
<td><strong>Row</strong><br />
<br />
In both Cassandra and Spanner, a row is a collection of columns identified uniquely by a primary key. Like Cassandra, Spanner supports composite primary keys. Unlike Cassandra, Spanner doesn't make a distinction between the partition key and clustering columns, because data is range-sharded. You can think of Spanner as only having clustering columns, with partitioning managed behind the scenes.</td>
</tr>
<tr class="even">
<td><strong>Column</strong></td>
<td><strong>Column</strong><br />
<br />
In both Cassandra and Spanner, a column is a set of data values that have the same type. There is one value for each row of a table. For more information about comparing Cassandra column types to Spanner, see <a href="#data-types">Data types</a> .</td>
</tr>
</tbody>
</table>

## Architecture

A Cassandra cluster consists of a set of servers and storage colocated with those servers. A hash function maps rows from a partition keyspace to a virtual node (vnode). A set of vnodes is then randomly assigned to each server to serve a portion of the cluster keyspace. Storage for the vnodes is locally attached to the serving node. Client drivers connect directly to the serving nodes and handle load balancing and query routing.

A Spanner instance consists of a set of servers in a [replication topology](/spanner/docs/instance-configurations) . Spanner dynamically shards each table into row ranges based on CPU and disk usage. Shards are assigned to compute nodes for serving. Data is physically stored on Colossus, Google's distributed file system, separate from the compute nodes. Client drivers connect to Spanner's frontend servers which perform request routing and load balancing. To learn more, see the [Life of Spanner reads and writes](/spanner/docs/whitepapers/life-of-reads-and-writes) whitepaper.

At a high level, both architectures scale as resources are added to the underlying cluster. Spanner's compute and storage separation lets the load between compute nodes rebalance faster in response to workload changes. Unlike Cassandra, shard moves don't involve data moves as the data stays on Colossus. Moreover, Spanner's range-based partitioning might be more natural for applications that expect data to be sorted by partition key. The flip-side of range-based partitioning is that workloads that write to one end of the keyspace (for example, tables keyed by the current timestamp) might have hotspots if additional schema designs aren't considered. For more information about techniques for overcoming hotspots, see [Schema design best practices](/spanner/docs/schema-design) .

### Consistency

With Cassandra, you must specify a consistency level for each operation. If you use the quorum consistency level, a replica node majority must respond to the coordinator node for the operation to be considered successful. If you use a consistency level of one, Cassandra needs a [single replica node](https://cassandra.apache.org/doc/4.1/cassandra/architecture/dynamo.html#tunable-consistency) to respond for the operation to be considered successful.

Spanner provides strong consistency. The [Spanner API](/spanner/docs/reference/rest) doesn't expose replicas to the client. Spanner clients interact with Spanner as if it were a single machine database. A write is always written to a majority of replicas before Spanner reports its success to the user. Any subsequent reads reflects the newly written data. Applications can choose to read a snapshot of the database at a time in the past, which might have performance benefits over strong reads. For more information about the consistency properties of Spanner, see the [Transactions overview](/spanner/docs/transactions) .

Spanner was built to support the consistency and availability needed in large scale applications. Spanner provides strong consistency at scale and with high performance. For use cases that require it, Spanner supports [snapshot (stale) reads](/spanner/docs/reads#read_types) that relax freshness requirements.

## Cassandra interface

The Cassandra interface lets you take advantage of Spanner's fully managed, scalable, and highly available infrastructure using familiar Cassandra tools and syntax. This page helps you understand the capabilities and limitations of the Cassandra interface.

### Benefits of the Cassandra interface

  - **Portability** : the Cassandra interface provides access to the breadth of Spanner features, using schemas, queries, and clients that are compatible with Cassandra. This simplifies moving an application built on Spanner to another Cassandra environment or vice-versa. This portability provides deployment flexibility and supports disaster recovery scenarios, such as a stressed exit.
  - **Familiarity** : if you already use Cassandra, you can quickly get started with Spanner using many of the same CQL statements and types.
  - **Uncompromisingly Spanner** : because it's built on Spanner's existing foundation, the Cassandra interface provides all of Spanner's existing availability, consistency, and price-performance benefits without having to compromise on any of the capabilities available in the complementary GoogleSQL ecosystem.

### CQL compatibility

  - **CQL dialect support** : Spanner provides a subset of the CQL dialect, including Data Query Language (DQL), Data Manipulation Language (DML), lightweight transactions (LWT), aggregate and datetime functions.

  - **Supported Cassandra functionality** : the Cassandra interface supports many of the most commonly used features of Cassandra. This includes core parts of the schema and type system, many common query shapes, a variety of functions and operators, and the key aspects of Cassandra's system catalog. Applications can use many Cassandra clients or drivers by connecting over Spanner's implementation of the Cassandra wire protocol.

  - **Client and wire protocol support** : Spanner supports the core query capabilities of the [Cassandra wire protocol v4](https://github.com/apache/cassandra/blob/trunk/doc/native_protocol_v4.spec) using Cassandra Adapter, a lightweight client that runs alongside your application. This lets many Cassandra clients work as-is with a Spanner Cassandra interface database, while leveraging Spanner's global endpoint and connection management and IAM authentication.

### Supported Cassandra data types

The following table shows supported Cassandra data types and maps each data type to the equivalent Spanner GoogleSQL data type.

**Supported Cassandra data types**

**Spanner GoogleSQL data type**

**Numeric types**

`  tinyint  ` (8-bit signed integer)

`  INT64  ` (64-bit signed integer)

Spanner supports a single 64-bit wide data type for signed integers.

`  smallint  ` (16-bit signed integer)

`  int  ` (32-bit signed integer)

`  bigint  ` (64-bit signed integer)

`  float  ` (32-bit IEEE-754 floating point)

`  FLOAT32  ` (32-bit IEEE-754 floating point)

`  double  ` (64-bit IEEE-754 floating point)

`  FLOAT64  ` (64-bit IEEE-754 floating point)

`  decimal  `

For fixed precision decimal numbers, use the `  NUMERIC  ` data type (precision 38 scale 9).

`  varint  ` (variable precision integer)

**String types**

`  text  `

`  STRING(MAX)  `

Both `  text  ` and `  varchar  ` store and validate for UTF-8 strings. In Spanner, `  STRING  ` columns need to specify their maximum length. There is no impact on storage; this is for validation purposes.

`  varchar  `

`  ascii  `

`  STRING(MAX)  `

`  uuid  `

`  STRING(MAX)  `

`  timeuuid  `

`  STRING(MAX)  `

`  inet  `

`  STRING(MAX)  `

`  blob  `

`  BYTES(MAX)  `

To store binary data, use the `  BYTES  ` data type.

**Date and time types**

`  date  `

`  DATE  `

`  time  `

`  INT64  `

Spanner doesn't support a dedicated time data type. Use `  INT64  ` to store nanosecond duration.

`  timestamp  `

`  TIMESTAMP  `

`  duration  `

`  STRING(MAX)  `

Spanner doesn't support a stored duration type. Use `  STRING  ` to store duration.

**Container types**

`  set  `

`  ARRAY  `

Spanner doesn't support a dedicated `  set  ` data type. Use `  ARRAY  ` columns to represent a `  set  ` .

`  list  `

`  ARRAY  `

Use `  ARRAY  ` to store a list of typed objects.

`  map  `

`  JSON  `

Spanner doesn't support a dedicated map type. Use `  JSON  ` columns to represent maps.

**Other types**

`  boolean  `

`  BOOL  `

`  counter  `

`  INT64  `

#### Data type Annotations

The `  cassandra_type  ` column option lets you define mappings between the Cassandra and Spanner data types. When you create a table in Spanner that you intend to interact with it using Cassandra-compatible queries, you can use the `  cassandra_type  ` option to specify the corresponding Cassandra data type for each column. This mapping is then used by Spanner to correctly interpret and convert data when transferring it between the two database systems.

For example, if there's a table in Cassandra with the following schema:

``` text
CREATE TABLE Albums (
  albumId uuid,
  title varchar,
  artists set<varchar>,
  tags  map<varchar, varchar>,
  numberOfSongs tinyint,
  releaseDate date,
  copiesSold bigint,
  score frozen<set<int>>
  ....
  PRIMARY KEY(albumId)
)
```

In Spanner, you use type annotations to map to the Cassandra data types, as shown in the following:

``` text
CREATE TABLE Albums (
  albumId       STRING(MAX) OPTIONS (cassandra_type = 'uuid'),
  title         STRING(MAX) OPTIONS (cassandra_type = 'varchar'),
  artists       ARRAY<STRING(max)> OPTIONS (cassandra_type = 'set<varchar>'),
  tags          JSON OPTIONS (cassandra_type = 'map<varchar, varchar>'),
  numberOfSongs INT64 OPTIONS (cassandra_type = 'tinyint'),
  releaseDate   DATE OPTIONS (cassandra_type = 'date'),
  copiesSold    INT64 OPTIONS (cassandra_type = 'bigint'),
  score         ARRAY<INT64> OPTIONS (cassandra_type = 'frozen<set<int>>')
  ...
) PRIMARY KEY (albumId);
```

In the previous example, the `  OPTIONS  ` clause maps the column's Spanner data type to its corresponding Cassandra data type.

  - `  albumId  ` (Spanner `  STRING(MAX)  ` ) is mapped to `  uuid  ` in Cassandra.
  - `  title  ` (Spanner `  STRING(MAX)  ` ) is mapped to `  varchar  ` in Cassandra.
  - `  artists  ` (Spanner `  ARRAY<STRING(MAX)>  ` ) is mapped to `  set<varchar>  ` in Cassandra.
  - `  tags  ` (Spanner `  JSON  ` ) is mapped to `  map<varchar,varchar>  ` in Cassandra.
  - `  numberOfSongs  ` (Spanner `  INT64  ` ) is mapped to `  tinyint  ` in Cassandra.
  - `  releaseDate  ` (Spanner `  DATE  ` ) is mapped to `  date  ` in Cassandra.
  - `  copiesSold  ` (Spanner `  INT64  ` ) is mapped to `  bigint  ` in Cassandra.
  - `  score  ` (Spanner `  ARRAY<INT64>  ` ) is mapped to `  frozen<set<int>>  ` in Cassandra.

##### Modify the `     cassandra_type    ` option

You can use the `  ALTER TABLE  ` statement to add or modify the `  cassandra_type  ` option on existing columns.

To add a `  cassandra_type  ` option to a column that doesn't have it yet, use the following statement:

``` text
ALTER TABLE Albums ALTER COLUMN uuid SET OPTIONS (cassandra_type='uuid');
```

In this example, the `  uuid  ` column in the Albums table is updated with the `  cassandra_type  ` option set to `  uuid  ` .

To modify an existing `  cassandra_type  ` option, use the `  ALTER TABLE  ` statement with the new `  cassandra_type  ` value. For example, to change the `  cassandra_type  ` of the `  numberOfSongs  ` column in the Albums table from `  tinyint  ` to `  bigint  ` , use the following statement:

``` text
ALTER TABLE Albums ALTER COLUMN numberOfSongs SET OPTIONS (cassandra_type='bigint');
```

You are only permitted to modify the following types:

<table>
<thead>
<tr class="header">
<th>From</th>
<th>To</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>tinyint</td>
<td>smallint, int, bigint</td>
</tr>
<tr class="even">
<td>smallint</td>
<td>int, bigint</td>
</tr>
<tr class="odd">
<td>int</td>
<td>bigint</td>
</tr>
<tr class="even">
<td>float</td>
<td>double</td>
</tr>
<tr class="odd">
<td>text</td>
<td>varchar</td>
</tr>
<tr class="even">
<td>ascii</td>
<td>varchar, text</td>
</tr>
</tbody>
</table>

##### Direct and nuanced mappings

In many cases, the mapping between Spanner and Cassandra data types is straightforward. For example, a Spanner `  STRING(MAX)  ` maps to a Cassandra `  varchar  ` , and a Spanner `  INT64  ` maps to a Cassandra `  bigint  ` .

However, there are situations where the mapping requires more consideration and adjustment. For example, you might need to map a Cassandra `  smallint  ` to a Spanner `  INT64  ` .

### Supported Cassandra functions

This section lists the Cassandra functions supported in Spanner.

The following list shows Spanner support for Cassandra functions.

  - All [aggregate functions](https://cassandra.apache.org/doc/4.1/cassandra/cql/functions.html#aggregate-functions)
  - All [datetime functions](https://cassandra.apache.org/doc/4.1/cassandra/cql/functions.html#datetime-functions) except for `  currentTimeUUID  `
  - All [cast functions](https://cassandra.apache.org/doc/4.1/cassandra/cql/functions.html#cast) except for blob conversion functions
  - All [lightweight transaction functions](https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useInsertLWT.html) except for [`  BATCH  `](https://docs.datastax.com/en/cql-oss/3.x/cql/cql_reference/cqlBatch.html) conditions

### Time to live (TTL)

When migrating from Cassandra, add a row deletion policy to your Spanner table in order to use the `  USING TTL  ` option in `  INSERT  ` or `  UPDATE  ` statements or the Spanner TTL.

The Spanner TTL logic operates at the *row level* , in contrast to Cassandra, where TTL logic can be applied at the *cell level* . To use the Spanner TTL, you must include a timestamp column and a time interval in the row deletion policy. Spanner deletes the row after the row exceeds the specified duration relative to the timestamp.

[Spanner TTL](/spanner/docs/ttl) deletion isn't instantaneous. An asynchronous background process deletes expired rows, and deletions can take up to 72 hours.

For more information, see [Enable TTL on Cassandra data](/spanner/docs/non-relational/migrate-from-cassandra-to-spanner#enable-ttl-on-cassandra) .

### Unsupported Cassandra features on Spanner

It's important to understand that the Cassandra interface provides the capabilities of Spanner through schemas, types, queries, and clients that are compatible with Cassandra. It doesn't support all of the features of Cassandra. Migrating an existing Cassandra application to Spanner, even using the Cassandra interface, likely requires some rework to accommodate unsupported Cassandra capabilities or differences in behavior, like query optimization or primary key design. However, once it's migrated, your workloads can take advantage of Spanner's reliability and unique multi-model capabilities.

The following list provides more information on unsupported Cassandra features:

  - **Some CQL language features aren't supported** : user defined types and functions, `  token  ` function.
  - **Spanner and Google Cloud control plane** : databases with Cassandra interfaces use Spanner and Google Cloud tools to provision, secure, monitor, and optimize instances. Spanner doesn't support tools, such as `  nodetool  ` for administrative activities.

### DDL support

CQL DDL statements are not directly supported using Cassandra interface. For DDL changes, you must use the Spanner Google Cloud console, gcloud command, or client libraries.

### Connectivity

  - 
    
    <div id="connect-cassandra">
    
    **Cassandra client support**
    
    Spanner lets you connect to databases from a variety of clients:
    
      - Cassandra Adapter can be used as an in-process helper or as a sidecar proxy to connect your Cassandra applications to Cassandra interface. For more information, see [Connect to Spanner using the Cassandra Adapter](/spanner/docs/non-relational/connect-cassandra-adapter) .
      - Cassandra Adapter can be started as a standalone process locally and connected using `  CQLSH  ` . For more information, see [Connect the Cassandra interface to your application](/spanner/docs/non-relational/connect-cassandra-adapter#standalone) .
    
    </div>

### Access control with Identity and Access Management

You need to have the `  spanner.databases.adapt  ` , `  spanner.databases.select  ` , and `  spanner.databases.write  ` permissions to perform read and write operations against the Cassandra endpoint. For more information, see the [IAM overview](/spanner/docs/iam) .

For more information about how to grant Spanner IAM permissions, see [Apply IAM roles](/spanner/docs/grant-permissions) .

### Monitoring

Spanner provides the following metrics to help you monitor the Cassandra Adapter:

  - `  spanner.googleapis.com/api/adapter_request_count  ` : captures and exposes the number of adapter requests that Spanner performs per second, or the number of errors that occurs on the Spanner server per second.
  - `  spanner.googleapis.com/api/adapter_request_latencies  ` : captures and exposes the amount of time that Spanner takes to handle adapter requests.

You can create a custom Cloud Monitoring dashboard to display and monitor metrics for Cassandra Adapter. The custom dashboard contains the following charts:

  - **P99 Request Latencies** : The 99th percentile distribution of server request latencies per `  message_type  ` for your database.

  - **P50 Request Latencies** : The 50th percentile distribution of server request latencies per `  message_type  ` for your database.

  - **API Request Count by Message Type** : The API request count per `  message_type  ` for your database.

  - **API Request Count by Operation Type** : The API request count per `  op_type  ` for your database.

  - **Error Rates** : The API error rates for your database.

### Google Cloud console

1.  Download the [`  cassandra-adapter-dashboard.json  `](/static/spanner/docs/non-relational/cassandra-adapter-dashboard.json) file. This file has the information needed to populate a custom dashboard in Monitoring.

2.  In the Google Cloud console, go to the dashboard **Dashboards** page:
    
    If you use the search bar to find this page, then select the result whose subheading is **Monitoring** .

3.  In the **Dashboards Overview** page, click **Create Custom Dashboard** .

4.  In the dashboard toolbar, click the **Dashboard settings** icon. Then select **JSON** , followed by **JSON Editor** .

5.  In the **JSON Editor** pane, copy the contents of the `  cassandra-adapter-dashboard.json  ` file you downloaded and paste it in the editor.

6.  To apply your changes to the dashboard, click **Apply changes** . If you don't want to use this dashboard, navigate back to the Dashboards Overview page.

7.  After the dashboard is created, click **Add Filter** . Then select either `  project_id  ` or `  instance_id  ` to monitor the Cassandra Adapter.

### gcloud CLI

1.  Download the [`  cassandra-adapter-dashboard.json  `](/static/spanner/docs/non-relational/cassandra-adapter-dashboard.json) file. This file has the information needed to populate a custom dashboard in Monitoring.

2.  To create a dashboard in a project, use the `  gcloud monitoring dashboards create  ` command:
    
    ``` text
    gcloud monitoring dashboards create --config-from-file=cassandra-adapter-dashboard.json
    ```
    
    For more information, see the [`  gcloud monitoring dashboards create  `](/sdk/gcloud/reference/monitoring/dashboards/create) reference.

Additionally, the following Spanner metrics are helpful for monitoring the Cassandra Adapter:

  - [CPU utilization metrics](/spanner/docs/cpu-utilization) provide information about CPU usage for user and system tasks with breakdowns by priority and operation type.
  - [Storage utilization metrics](/spanner/docs/storage-utilization) provide information about database and backup storage.
  - [Spanner's built-in statistics tables](/spanner/docs/introspection) provide insights about queries, transactions, and reads to help you discover issues in your databases.

For a complete list of system insights, see [Monitor instances with system insights](/spanner/docs/monitoring-console) . To learn more about monitoring your Spanner resources, see [Monitor instances with Cloud Monitoring](/spanner/docs/monitoring-cloud) .

### Pricing

There is no additional charge for using the Cassandra endpoint. You are charged the standard Spanner pricing for the amount of compute capacity that your instance uses and the amount of storage that your database uses.

For more information, see [Spanner pricing](https://cloud.google.com/spanner/pricing) .

## What's next

  - Learn how to [Migrate from Cassandra to Spanner](/spanner/docs/non-relational/migrate-from-cassandra-to-spanner) .
  - Learn how to [Connect to Spanner using the Cassandra Adapter](/spanner/docs/non-relational/connect-cassandra-adapter) .
  - Try the [Spanner for Cassandra users codelab](https://codelabs.developers.google.com/codelabs/spanner-cassandra-adapter-getting-started#0) .
