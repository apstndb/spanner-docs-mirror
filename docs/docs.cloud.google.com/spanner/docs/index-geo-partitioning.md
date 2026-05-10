---
name: documents/docs.cloud.google.com/spanner/docs/index-geo-partitioning
uri: https://docs.cloud.google.com/spanner/docs/index-geo-partitioning
title: Indexes and geo-partitioning
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
update_time: "2026-05-08T21:32:28Z"
---

Spanner offers several types of indexes for improving query performance. Choosing the correct index type for your schema and query patterns is critical, especially for databases that use geo-partitioning. This page describes the benefits of different index types and best practices for selecting and using Spanner indexes with geo-partitioning.

## Index types

Spanner supports global, local, and remote indexes. Each type has different performance characteristics and use cases. For geo-partitioned databases, it's important to understand these index types. Choosing the right index helps you optimize your database schema and queries, which can significantly improve latency for your geo-partitioned database. For databases that don't use geo-partitioning, it's less important to understand these index types because they are all stored in the default placement and have similar performance characteristics.

### Global indexes

A global index is the default index type in Spanner. The index data is stored in the default partition, which might not be colocated with the table's data. Creating global indexes on geo-partitioned tables might result in significantly higher write latencies for writes involving the indexed columns, especially if the default partition [write quorum](https://docs.cloud.google.com/spanner/docs/instance-configurations#regional-replication) for the index is far from the write quorum of the table rows that are being written to. You can alleviate the read latencies of global indexes by using nearby [read-only replicas](https://docs.cloud.google.com/spanner/docs/replication#read-only) alongside [read lease](https://docs.cloud.google.com/spanner/docs/read-lease) regions or [stale reads](https://docs.cloud.google.com/spanner/docs/reads#perform-stale-read) .

Global indexes have the following characteristics:

  - They speed up queries that would otherwise require a full table scan, and enforce uniqueness across all table rows regardless of location.
  - They are suitable for columns that require unique values across a database.
  - They speed up queries that filter or sort by indexed columns.

The following is an example of a global unique index:

    CREATE UNIQUE INDEX idx_customer_email ON customer(email);

### Local indexes

A local index is interleaved in the parent hierarchy of the indexed table. The primary key column names and types must match the indexed table.

Local indexes have the following characteristics:

  - They store index data in the same partition as the data that is indexed. Write latency is subject to the particular placement's write quorum, not the default placement write quorum.

  - They provide the lowest latency for queries that target a specific key prefix, because both the index and table data are colocated in the same placement.

To create a local index, interleave the index in the parent table. If you use a `UNIQUE` local index, uniqueness is enforced only within a particular parent row, not across the entire table.

The following is an example of creating a local index on the table `customer` , interleaved in parent table `locations` :

### GoogleSQL

    -- Create locations placement table
    CREATE TABLE locations (
      location STRING(MAX) NOT NULL PLACEMENT KEY,
    ) PRIMARY KEY(location);
    
    -- Create customer table interleaved in the locations table
    CREATE TABLE customer (
      location STRING(MAX) NOT NULL,
      customerId  INT64 NOT NULL,
      email STRING(MAX),
      webcookie STRING(64),
    ) PRIMARY KEY(location, customerId), INTERLEAVE IN PARENT locations;
    
    -- Create a local index on the interleaved customer table
    CREATE INDEX idx_customer_email_local ON customer(location, email),
    INTERLEAVE IN locations;

### PostgreSQL

    -- Create locations placement table
    CREATE TABLE locations (
      location varchar NOT NULL PLACEMENT KEY PRIMARY KEY
    );
    
    -- Create customer table interleaved in the locations table
    CREATE TABLE customer (
      location varchar NOT NULL,
      customerId  BIGINT NOT NULL,
      email varchar(1024),
      webcookie varchar(64),
      PRIMARY KEY(location, customerId)
    ) INTERLEAVE IN PARENT locations;
    
    -- Create a local index on the interleaved customer table
    CREATE INDEX idx_customer_email_local ON customer(location, email)
    INTERLEAVE IN locations;

When you query data in a read-write transaction, you must specify the primary key prefix of the indexed table in the query. Otherwise, the query might require a full table scan. For example:

### GoogleSQL

    -- The location (the index key prefix) must be specified
    SELECT *
    FROM customer
    WHERE location= @location AND email= @email;

### PostgreSQL

    -- The location (the index key prefix) must be specified
    SELECT *
    FROM customer
    WHERE location= @location AND email= @email;

For information about optimization that doesn't require you to specify the location, see [Use global unique index with local or remote index](https://docs.cloud.google.com/spanner/docs/index-geo-partitioning#use-global-unique) .

A local index is also useful when the placement table is entity based and you want to index data within a particular entity or sub-entity. For example:

### GoogleSQL

    -- Create entity based customer placement table
    CREATE TABLE customer (
      customerId INT64 NOT NULL,
      email STRING(MAX),
      webcookie STRING(64),
      location STRING(MAX) NOT NULL PLACEMENT KEY
    ) PRIMARY KEY(customerId);
    
    -- Create customerOrders child table
    CREATE TABLE customerOrders (
      customerId INT64 NOT NULL,
      orderId INT64 NOT NULL,
      orderName STRING(MAX) NOT NULL
    ) PRIMARY KEY(customerId, orderId), INTERLEAVE IN PARENT customer;
    
    -- Create a local index on the interleaved child table
    CREATE INDEX idx_order_local ON customerOrders(customerId, orderName),
    INTERLEAVE IN customer;

### PostgreSQL

    -- Create entity based customer placement table
    CREATE TABLE customer (
      customerId BIGINT NOT NULL PRIMARY KEY,
      email varchar(1024),
      webcookie varchar(64),
      location varchar NOT NULL PLACEMENT KEY
    );
    
    -- Create customerOrders child table
    CREATE TABLE customerOrders (
      customerId BIGINT NOT NULL,
      orderId BIGINT NOT NULL,
      orderName varchar(1024) NOT NULL,
      PRIMARY KEY(customerId, orderId)
    ) INTERLEAVE IN PARENT customer;
    
    -- Create a local index on the interleaved child table
    CREATE INDEX idx_order_local ON customerOrders(customerId, orderName)
    INTERLEAVE IN customer;

### Remote indexes

A remote index interleaves index data under a table that isn't an ancestor in the indexed table's interleaving hierarchy. The primary key types must match the types of corresponding index columns, but the names can be different.

Using geo-partitioning, you can only interleave a remote index under the automatically-managed root placement table. This table contains one row for each placement in your database.

Remote indexes have the following characteristics:

  - They are particularly useful when your table's primary key isn't prefixed using a placement key but you want to geographically colocate partitions of the index according to the placement key.
  - They only support indexing columns in the placement table, and not in any columns in the interleaved child tables.

To use remote indexes with geo-partitioning, create the root placement table by setting the `auto_managed_root_placement_table_name` option in the `ALTER DATABASE` DDL statement.

1.  Use the `ALTER DATABASE DDL` statement to create a root placement table.
    
    ### GoogleSQL
    
        ALTER DATABASE DATABASE_NAME SET OPTIONS
          (auto_managed_root_placement_table_name="TABLE_NAME");
    
    Replace the following:
    
      - DATABASE\_NAME : The name of your database.
      - TABLE\_NAME : The name of the table to create. We recommend using the name `root_placement_table` .
    
    For example, the following command creates a table called `root_placement_table` .
    
        ALTER DATABASE example_db SET OPTIONS
          (auto_managed_root_placement_table_name='root_placement_table');
    
    After you create the root placement table, Spanner creates an internal table and automatically inserts and deletes rows when you create or drop placement. The following is an example of a system-defined placement table that is created by Spanner, where the example table name is set to `root_placement_table` . (Don't run this example.)
    
        // Automatically generated after you run the previous example.
        // Don't put this in your schema explicitly.
        CREATE TABLE root_placement_table (
        location STRING(MAX) NOT NULL PLACEMENT KEY
        ) PRIMARY KEY(location);
    
    ### PostgreSQL
    
        ALTER DATABASE DATABASE_NAME SET
          spanner.auto_managed_root_placement_table_name='TABLE_NAME';
    
    Replace the following:
    
      - DATABASE\_NAME : The name of your database.
      - TABLE\_NAME : The name of the table to create.
    
    For example, to create a `root_placement_table` table that is used as an interleave root, run:
    
        ALTER DATABASE example_db SET
          spanner.auto_managed_root_placement_table_name='root_placement_table';
    
    After you create the root placement table, Spanner creates an internal table and automatically inserts and deletes rows when you create or drop placement. The following is an example of a system-defined placement table that is created by Spanner, where the example table name is set to `root_placement_table` . (Don't run this example.)
    
        // Automatically generated after you run the previous example.
        // Don't put this in your schema explicitly.
        CREATE TABLE root_placement_table (
          location varchar NOT NULL PLACEMENT KEY,
          PRIMARY KEY (location)
        );

2.  Create a remote index interleaved under the automatically-managed `root_placement_table` table.
    
    ### GoogleSQL
    
        -- Create a customer table with a primary key that is not the location
        CREATE TABLE customer (
          customerId INT64 NOT NULL ,
          email STRING(MAX),
          webcookie STRING(64),
          location STRING(MAX) NOT NULL PLACEMENT KEY,
        ) PRIMARY KEY(customerId);
        
        -- Create a remote index on the customer table
        CREATE INDEX idx_customer_email_remote ON customer(location, email),
        INTERLEAVE IN root_placement_table;
    
    ### PostgreSQL
    
        -- Create a customer table with a primary key that is not the location
        CREATE TABLE customer (
          customerId BIGINT NOT NULL PRIMARY KEY,
          email varchar(1024),
          webcookie varchar(64),
          location varchar NOT NULL PLACEMENT KEY
        );
        
        -- Creates a remote index on the customer table
        CREATE INDEX idx_customer_email_remote ON customer(location, email)
        INTERLEAVE IN root_placement_table;

3.  When you query data in a read-write transaction, specify the key prefix of the index in the query predicate so that a full table scan isn't required. For example:
    
    ### GoogleSQL
    
        -- Specify the location (the index key prefix) in query
        SELECT *
        FROM customer
        WHERE location= @location AND email= @email;
    
    For information about optimization that doesn't require you to specify the location, see the section [Global unique index with local or remote index](https://docs.cloud.google.com/spanner/docs/index-geo-partitioning#use-global-unique)
    
    ### PostgreSQL
    
        -- Specify the location (the index key prefix) in query
        SELECT *
        FROM customer
        WHERE location= @location AND email= @email;
    
    For information about optimization that doesn't require you to specify the location, see the section [Global unique index with local or remote index](https://docs.cloud.google.com/spanner/docs/index-geo-partitioning#use-global-unique) .

## Optimizations for global unique indexes

When you use a global unique index, Spanner may trigger query latency improvements with heuristics-based optimizations in the following use cases:

  - When you use a global unique index with a local or remote index
  - When you use a global unique index with partial primary keys

The following sections describe how Spanner may apply optimizations in each use case.

### Global unique index with a local or remote index

To improve local query latency, Spanner may initiate a heuristics-based optimization when a global unique index is combined with a local or remote index.

This optimization minimizes latency for intra-region queries — even when the location of geo-partitioned data is unspecified — by guessing that the placement is the same as where the client is located and bypassing the global index's default partition, or eliminating the necessity for filtered full table scans. Such an approach is particularly beneficial when clients predominantly access data that is stored within their own region.

Using a mixture of different index types is helpful if intra-region query latency is the primary concern, and you can tolerate some increased write latency. Combining different index types also improves the performance of intra-region queries, even when you don't specify the location in your query.

This optimization requires that you create a global unique index and a corresponding local or remote index on the same column. The indexed data must be globally unique. Spanner applies this optimization to your query if the following are true:

  - You don't know the primary key prefix and don't specify the location of the data.
  - Your request originates from the same region as the default leader of the data's placement, which contains the local or remote index shard.

Spanner applies the optimization in the following ways:

  - If the optimization is triggered and the row is found in the local placement: Given the global unique index, Spanner doesn't need to search other locations. Your query has intra-region latency.
  - If the initial location search doesn't return a row: This indicates that it isn't an intra-region query. Spanner falls back to using the global index.

The following example creates a global unique index and local index:

### GoogleSQL

    CREATE UNIQUE INDEX idx_customer_email ON customer(email);
    CREATE INDEX idx_customer_email_local ON customer(location, email), INTERLEAVE IN locations;

### PostgreSQL

    CREATE UNIQUE INDEX idx_customer_email ON customer(email);
    CREATE INDEX idx_customer_email_local ON customer(location, email) INTERLEAVE IN locations;

The following example creates a global unique index and a remote index:

### GoogleSQL

    CREATE UNIQUE INDEX idx_customer_email ON customer(email);
    CREATE INDEX idx_customer_email_remote ON customer(location, email), INTERLEAVE IN root_placement_table;

### PostgreSQL

    CREATE UNIQUE INDEX idx_customer_email ON customer(email);
    CREATE INDEX idx_customer_email_remote ON customer(location, email) INTERLEAVE IN root_placement_table;

Based on the previous example indexes, the following example query has intra-region latency:

### GoogleSQL

    SELECT *
    FROM customer
    WHERE email= @email;

### PostgreSQL

    SELECT *
    FROM customer
    WHERE email= @email;

### Global unique index on partial primary keys

Spanner can apply an optimization that is similar to the one that is detailed in [Use a global unique index with a local or remote index](https://docs.cloud.google.com/spanner/docs/index-geo-partitioning#use-global-unique) when employing a global unique index on partial primary keys.

The following example creates a `customer` interleaved in parent table `locations` , then creates a global unique index on the `customerId` column.

### GoogleSQL

    -- Create locations placement table
    CREATE TABLE locations (
    location STRING(MAX) NOT NULL PLACEMENT KEY,
    ) PRIMARY KEY(location);
    
    -- Create customer table interleaved in the locations table
    CREATE TABLE customer (
      location STRING(MAX) NOT NULL,
      customerId  INT64 NOT NULL,
      email STRING(MAX),
      webcookie STRING(64),
    ) PRIMARY KEY(location, customerId), INTERLEAVE IN PARENT locations;
    
    -- Create global unique index on customerId column
    CREATE UNIQUE INDEX idx_customer_customerid ON customer(customerId);

### PostgreSQL

    -- Create locations placement table
    CREATE TABLE locations (
      location varchar NOT NULL PLACEMENT KEY PRIMARY KEY
    );
    
    -- Create customer table interleaved in the locations table
    CREATE TABLE customer (
      location varchar NOT NULL,
      customerId  BIGINT NOT NULL,
      email varchar(1024),
      webcookie varchar(64),
      PRIMARY KEY(location, customerId)
    ) INTERLEAVE IN PARENT locations;
    
    -- Create global unique index on customerId column
    CREATE UNIQUE INDEX idx_customer_customerid ON customer(customerId);

The optimization applies to queries like the following:

### GoogleSQL

    SELECT * FROM customer WHERE customerId= @customerId;

### PostgreSQL

    SELECT * FROM customer WHERE customerId= @customerId;

If you don't create the global unique index, then this query might require a full table scan. If you're not using the global unique index, then you would need to add the location filter in your query to achieve good query latency:

### GoogleSQL

    SELECT * FROM customer WHERE location = @location AND customerId= @customerId;

### PostgreSQL

    SELECT * FROM customer WHERE location = @location AND customerId= @customerId;

## General guidelines for choosing an index type for optimal latency

The type of index that you choose directly impacts query latency. The location of index data relative to table data is a primary factor in performance for geo-partitioned workloads.

This section describes how to choose between global, local, and remote indexes.

### When to choose global indexes

Use global indexes if your workload can tolerate the associated read and write latency, or if you require enforced global uniqueness on indexed columns.

Consider the following when choosing global indexes:

  - The distance between the client and the leader of the default write quorum, along with the quorum's default latency, determines the increase in write latency. This effect is specifically limited to operations involving indexed columns, such as row insertions or updates to indexed columns.
  - Adding read-only replicas or using read leases can alleviate increased read latency:
      - Adding a read-only replica in geographic proximity can reduce stale read latency.
      - Adding a read-only replica and using read lease regions can reduce strong read latency. If you add a read-only replica without using a read lease region, strong read latency isn't reduced but read throughput can increase.
      - Spanner always serves pessimistic transactional reads from the leader. Adding replicas to the default placement doesn't help with pessimistic transactional reads of data in the default placements.
  - Global indexes (including keys and storing values) are placed in the default placement, which doesn't provide placement-level data residency. For more information, see the Spanner [Data residency overview](https://docs.cloud.google.com/spanner/docs/data-residency) .

### When to choose local and remote indexes

Consider the following when choosing local and remote indexes:

  - Local and remote indexes provide placement-local read and write performance, but sacrifice the global uniqueness and ordering properties of unique index columns. Instead, local and remote indexes provide ordering and uniqueness of indexed columns within the parent row they are interleaved in.
  - When using local or remote indexes, you must include the placement location in the query predicate, except in cases where there is also a global unique index that lets Spanner guess the local placement location. Otherwise, the query plan and performance isn't deterministic. Spanner might perform base table scan or scatter and gather from the index across placement locations based on query statistics, increasing latency.

### When to choose global unique indexes with local or remote indexes

Consider the following when choosing a combination of global unique indexes together with local or remote indexes:

  - When the specific placement location is unknown, use a combination of global unique indexes with local or remote indexes. This approach is ideal when most queries originate from services geographically situated within the same region as the requested data's placement.
  - When writing global indexes, write latency is subject to additional default write quorum latency.
  - With the heuristic-based optimization, queries are served by local index shards and demonstrate intra-region latency most of the time.

## Detailed guidelines for choosing an index for specific schema designs

The optimal index strategy depends on your table's primary key structure and your application's query patterns. This section provides guidance for selecting the appropriate index type for three common schema designs:

  - Schemas that use an entity as the primary key
  - Schemas that use location as the primary key
  - Schemas that use location-related values as the primary key

### Schema design: Entity as the primary key

If your schema uses an entity as the primary key, then choose your indexing strategy based on whether or not location is specified in your queries.

In cases where an entity like `customerID` is the primary key and a separate non-key column like `location` is the placement key, determine your indexing strategy for the placement table based on your query patterns. (Don't use an entity as primary key of the placement table if insert latency is a concern for the entities.)

If you want to index data under a particular entity, such as `customerID` , then use a local index. Data is indexed and sorted within the entity but not across it. For example, if you wanted to index each customer's orders by date, you could create a local index interleaved under the `customerID` identity.

When the location is always known in your queries, use one of the following strategies:

  - If location is consistently known in your queries and enforcing global uniqueness isn't required, then use remote indexes. These indexes provide intra-region latency for both read and write operations.
    
    Remote indexes only support indexing columns in the placement table, not in columns in interleaved tables. The remote index must be interleaved under the root placement table. The remote index indexes data across all the placement rows for the placement.

In cases where the location *isn't* always known in your queries, use one of the following strategies:

  - If the indexed column is globally unique, create a global unique index to enforce that uniqueness.
    
    To achieve low-latency strong reads, create a remote index in addition to a global unique index.
    
    With this combination, writes might incur cross-region latency, while queries with a specified location (with `WHERE location= @location` ) benefit from intra-region latency by using the remote index. For queries without a specified location, Spanner uses a heuristics-based optimization by first searching locally. If the data isn't found, it falls back to the global index.
    
    If you are using read lease regions and have a read-only replica of the default partition in the same region as your data, a remote index is unnecessary because the read lease regions already provide low strong read latency for reads of a global index.

  - If your query doesn't specify a location and the indexed columns aren't globally unique, then create only a global (non-unique) index. Adding a local or remote index doesn't improve read latency in this case because Spanner can't determine whether there is matching data in another placement, even if Spanner finds data in the local placement.

### Schema design: Location as the primary key

When a `location` column serves as both the primary key and the placement key, your index selection is informed by cross-latency concerns and whether or not your queries always specify the location.

  - If cross-region latency isn't a concern, or you need global uniqueness, use global indexes.

  - If cross-region latency is a concern, your queries always include the location, and you don't need Spanner to enforce global uniqueness, use only local indexes. This ensures local latency for both reads and writes.

  - If cross-region query latency is a concern, cross-region write latency is acceptable, and the location isn't always known, then the following strategies apply:
    
    <table>
    <colgroup>
    <col style="width: 50%" />
    <col style="width: 50%" />
    </colgroup>
    <thead>
    <tr class="header">
    <th>Condition</th>
    <th>Recommendation</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td>Query by partial primary key that is globally unique</td>
    <td><p>Create a unique global index to enforce uniqueness. A local index is not needed because the primary key performs a similar function. The heuristics-based optimization applies. First, Spanner checks the full primary key with the local location before falling back to the global index.</p>
    <p>For an example, see <a href="https://docs.cloud.google.com/spanner/docs/index-geo-partitioning#unique-index-partial-primary-key">Global unique index on partial primary keys</a> .</p></td>
    </tr>
    <tr class="even">
    <td>Query by non-key globally unique column</td>
    <td><p>Create a unique global index to enforce uniqueness.</p>
    <p>For intra-region latency, the following scenarios are possible:</p>
    <ul>
    <li>Create a local index on the same column as the global index. The heuristics-based optimization applies. The combination of global and local indexes provides low latency for intra-region strong and stale queries, while writes and cross-region queries and writes have cross-region latency.</li>
    <li>If there is a read-write or read-only replica of the default partition in the same region as your data, then:
    <ul>
    <li>If you only need intra-region latency for stale reads, but not for strong reads, then you don't need a local index. The local replica provides intra-region latency.</li>
    <li>If you need intra-region latency for strong reads, you can create a local index on the same column as the global index, or use read lease regions. Read lease regions provide low strong read latency at the expense of write latency.</li>
    </ul></li>
    </ul></td>
    </tr>
    <tr class="odd">
    <td>Indexed column isn't globally unique</td>
    <td>Create only a global index. A local index doesn't improve read latency because Spanner might need to check all locations.</td>
    </tr>
    </tbody>
    </table>

If these three scenarios don't apply to your use case, you likely have to sacrifice application simplicity or write latency by consistently providing the location.

### Schema design: Location-related values as the primary key

If your table's primary key is based on location-related values, but it isn't directly the placement key column (for example, using `country` as the primary key when you have fewer placements than countries), you can use either a global index or a local index (interleaved under the `country` column). However, remote indexes aren't supported for any tables interleaved under this type of placement table.

The heuristics-based optimization isn't supported for local indexes in this scenario. Therefore, you only achieve local latency when your query explicitly specifies the primary key prefix.
