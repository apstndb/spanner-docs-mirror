The distributed architecture of Spanner lets you design your schema to avoid *hotspots* - situations where too many requests are sent to the same server which saturates the resources of the server and potentially causes high latencies.

This page describes best practices for designing your schemas to avoid creating hotspots. One way to avoid hotspots is to adjust the schema design to allow Spanner to split and distribute the data across multiple servers. Distributing data across servers helps your Spanner database operate efficiently, particularly when performing bulk data insertions.

Spanner automatically detects opportunities to apply [schema design best practices](/spanner/docs/schema-design) . If recommendations are available for a database, you can view them on the **Spanner Studio** page for that database. For more information, see [View schema design best practice recommendations](/spanner/docs/manage-data-using-console#recommend-schema-best) .

## Choose a primary key to prevent hotspots

To avoid creating hotspots in your database, carefully [choose a primary key](/spanner/docs/schema-and-data-model#choosing_a_primary_key) during schema design.

A common cause of hotspots is using a key that [monotonically increases or decreases](https://en.wikipedia.org/wiki/Monotonic_function) , such as a timestamp. Monotonic keys cause all new entries to write to the same range of your key space. Because Spanner uses key ranges to distribute data across servers, a monotonic key directs all insert traffic to a single server, creating a bottleneck.

For example, suppose you want to maintain a last access timestamp column on rows of the `  UserAccessLogs  ` table. The following table definition uses a timestamp-based primary key as the first key part. We don't recommend this if the table sees a high rate of insertion:

### GoogleSQL

``` text
CREATE TABLE UserAccessLogs (
  LastAccess TIMESTAMP NOT NULL,
  UserId STRING(1024),
  ...
) PRIMARY KEY (LastAccess, UserId);
```

### PostgreSQL

``` text
CREATE TABLE useraccesslogs (
  lastaccess timestamptz NOT NULL,
  userid text,
  ...
PRIMARY KEY (lastaccess, userid)
);
```

The problem here is that rows are written to this table in order of last access timestamp, and because last access timestamps are always increasing, they're always written to the end of the table. The hotspot is created because a single Spanner server receives all of the writes, which overloads that one server.

The following diagram illustrates this pitfall:

The previous `  UserAccessLogs  ` table includes five example rows of data, which represent five different users taking some sort of user action about a millisecond apart from each other. The diagram also annotates the order in which Spanner inserts the rows (the labeled arrows indicate the order of writes for each row). Because inserts are ordered by timestamp, and the timestamp value is always increasing, Spanner always adds the inserts to the end of the table and directs them at the same split. (As discussed in [Schema and data model](/spanner/docs/schema-and-data-model#database-splits) , a split is a set of rows from one or more related tables that Spanner stores in order of row key.)

This is problematic because Spanner assigns work to different servers in units of splits, so the server assigned to this particular split ends up handling all the insert requests. As the frequency of user access events increases, the frequency of insert requests to the corresponding server also increases. The server then becomes prone to becoming a hotspot, and looks like the red border and background shown in the previous image. In this simplified illustration, each server handles at most one split but Spanner can assign each server more than one split.

When Spanner appends more rows to the table, the split grows, and when it reaches approximately 8 GiB, Spanner creates another split, as described in [Load-based splitting](/spanner/docs/schema-and-data-model#load-based_splitting) . Spanner appends subsequent new rows to this new split, and the server assigned to the split becomes the new potential hotspot.

When hotspots occur, you might observe that your inserts are slow and other work on the same server might slow down. Changing the order of the `  LastAccess  ` column to ascending order doesn't solve this problem because then all the writes are inserted at the top of the table instead, which still sends all the inserts to a single server.

**Schema design best practice \#1: Do not choose a column whose value monotonically increases or decreases as the first key part for a high write rate table.**

### Use a universally unique identifier (UUID)

You can use a universally unique identifier (UUID) as defined by [RFC 9562](https://datatracker.ietf.org/doc/html/rfc9562) as the primary key. We recommend using [UUID Version 4](https://datatracker.ietf.org/doc/html/rfc9562#name-uuid-version-4) , because it uses random values in the bit sequence. We don't recommend Version 1 UUIDs because they store the timestamp in the high order bits. You can store UUID Version 4 values in a `  UUID  ` column on Spanner.

Consider the following before deciding to use UUIDs:

  - They function independently of the record's content. Unlike semantic keys such as `  SingerId  ` and `  AlbumId  ` , a UUID is strictly a unique identifier unrelated to the data itself.
  - They don't keep locality between related records, which is why using a UUID eliminates hotspots.

For a `  UUID  ` column, you can use the Spanner [`  NEW_UUID()  `](/spanner/docs/reference/standard-sql/utility-functions#new_uuid) GoogleSQL function or the [`  gen_random_uuid()  `](/spanner/docs/reference/postgresql/functions#utility) PostgreSQL function to create UUID values.

For example, for the following table:

### GoogleSQL

``` text
  CREATE TABLE UserAccessLogs (
    LogEntryId UUID DEFAULT (NEW_UUID()),
    LastAccess TIMESTAMP NOT NULL,
    UserId STRING(1024)
  ) PRIMARY KEY (LogEntryId);
```

### PostgreSQL

``` text
  CREATE TABLE useraccesslogs (
    logentryid uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    lastaccess timestamptz NOT NULL,
    userid text);
```

You can use the generated UUID function to create new `  LogEntryId  ` values.

### GoogleSQL

``` text
INSERT INTO UserAccessLogs (LastAccess, UserId)
VALUES ('2016-01-25 10:10:10.555555-05:00', 'TomSmith');
```

### PostgreSQL

``` text
INSERT INTO UserAccessLogs (LastAccess, UserId)
VALUES ('2016-01-25 10:10:10.555555-05:00', 'TomSmith');
```

For a `  UUID  ` column, you can use the Spanner [`  NEW_UUID()  `](/spanner/docs/reference/standard-sql/utility-functions#new_uuid) GoogleSQL function or the [`  gen_random_uuid()  `](/spanner/docs/reference/postgresql/functions#utility) PostgreSQL function as the column default value so that Spanner automatically generates UUID values.

For example, for the following table:

### GoogleSQL

``` text
CREATE TABLE UserAccessLogs (
  LogEntryId UUID NOT NULL,
  LastAccess TIMESTAMP NOT NULL,
  UserId STRING(1024),
  ...
) PRIMARY KEY (LogEntryId);
```

### PostgreSQL

``` text
CREATE TABLE useraccesslogs (
  logentryid uuid PRIMARY KEY NOT NULL,
  lastaccess timestamptz NOT NULL,
  userid text);
```

You can insert GoogleSQL `  NEW_UUID()  ` or PostgreSQL `  gen_random_uuid()  ` to generate the `  LogEntryId  ` values. These functions produce a `  UUID  ` value, so the `  LogEntryId  ` column must use the `  UUID  ` type for GoogleSQL or PostgreSQL.

### GoogleSQL

``` text
INSERT INTO
  UserAccessLogs (LogEntryId, LastAccess, UserId)
VALUES
  (NEW_UUID(), '2016-01-25 10:10:10.555555-05:00', 'TomSmith');
```

### PostgreSQL

``` text
INSERT INTO
  useraccesslogs (logentryid, lastaccess, userid)
VALUES
  (gen_random_uuid(),'2016-01-25 10:10:10.555555-05:00', 'TomSmith');
```

You can also insert UUID values you've generated elsewhere, such as your backend application. This is because UUIDs are unique, regardless of where they are generated.

### GoogleSQL

``` text
INSERT INTO
  UserAccessLogs (LogEntryId, LastAccess, UserId)
VALUES
  ('4192bff0-e1e0-43ce-a4db-912808c32493', '2016-01-25 10:10:10.555555-05:00', 'TomSmith');
```

### PostgreSQL

``` text
INSERT INTO
  useraccesslogs (logentryid, lastaccess, userid)
VALUES
  ('4192bff0-e1e0-43ce-a4db-912808c32493','2016-01-25 10:10:10.555555-05:00', 'TomSmith');
```

### Bit-reverse sequential values

You should verify that numerical ( `  INT64  ` in GoogleSQL or `  bigint  ` in PostgreSQL) primary keys aren't sequentially increasing or decreasing. Sequential primary keys can cause hotspots at scale. One way to avoid this problem is to bit-reverse the sequential values, making sure to distribute primary key values evenly across the key space.

Spanner supports bit-reversed sequence, which generates unique integer bit-reversed values. You can use a sequence in the first (or only) component in a primary key to avoid hotspot issues. For more information, see [Bit-reversed sequence](/spanner/docs/primary-key-default-value#bit-reversed-sequence) .

### Swap the order of keys

One way to spread writes over the key space more uniformly is to swap the order of the keys so that the column that contains the monotonic value is not the first key part:

### GoogleSQL

``` text
CREATE TABLE UserAccessLogs (
UserId     INT64 NOT NULL,
LastAccess TIMESTAMP NOT NULL,
...
) PRIMARY KEY (UserId, LastAccess);
```

### PostgreSQL

``` text
CREATE TABLE useraccesslogs (
userid bigint NOT NULL,
lastaccess TIMESTAMPTZ NOT NULL,
...
PRIMARY KEY (UserId, LastAccess)
);
```

In this modified schema, inserts are now first ordered by `  UserId  ` , rather than by chronological last access timestamp. This schema spreads writes among different splits because it's unlikely that a single user produces thousands of events per second.

The following image shows the five rows from the `  UserAccessLogs  ` table that Spanner orders with `  UserId  ` instead of access timestamp:

Here Spanner might divide the `  UserAccessLogs  ` data into three splits, with each split containing approximately a thousand rows of ordered `  UserId  ` values. Even though the user events occurred about a millisecond apart, each event was raised by a different user, so the order of inserts is much less likely to create a hotspot compared with using the timestamp for ordering. To learn more about how splits are created, see [Load-based splitting](/spanner/docs/schema-and-data-model#load-based_splitting)

See also the related best practice for [ordering timestamp-based keys](#ordering_timestamp-based_keys) .

### Hash the unique key and spread the writes across logical shards

Another common technique for spreading the load across multiple servers is to create a column that contains the hash of the actual unique key, then use the hash column (or the hash column and the unique key columns together) as the primary key. This pattern helps avoid hotspots, because new rows are spread more evenly across the key space.

You can use the hash value to create logical shards, or partitions, in your database. In a physically sharded database, the rows are spread across several database servers. In a logically sharded database, the data in the table define the shards. For example, to spread writes to the `  UserAccessLogs  ` table across N logical shards, you could prepend a `  ShardId  ` key column to the table:

### GoogleSQL

``` text
CREATE TABLE UserAccessLogs (
ShardId     INT64 NOT NULL,
LastAccess  TIMESTAMP NOT NULL,
UserId      INT64 NOT NULL,
...
) PRIMARY KEY (ShardId, LastAccess, UserId);
```

### PostgreSQL

``` text
CREATE TABLE useraccesslogs (
shardid bigint NOT NULL,
lastaccess TIMESTAMPTZ NOT NULL,
userid bigint NOT NULL,
...
PRIMARY KEY (shardid, lastaccess, userid)
);
```

To compute the `  ShardId  ` , hash a combination of the primary key columns and then calculate modulo N of the hash. For example:

### GoogleSQL

``` text
ShardId = hash(LastAccess and UserId) % N
```

Your choice of hash function and combination of columns determines how the rows are spread across the key space. Spanner will then create splits across the rows to optimize performance.

The following diagram illustrates how using a hash to create three logical shards can spread write throughput more evenly across servers:

Here the `  UserAccessLogs  ` table is ordered by `  ShardId  ` , which is calculated as a hash function of key columns. The five `  UserAccessLogs  ` rows are chunked into three logical shards, each of which is coincidentally in a different split. The inserts are spread evenly among the splits, which balances write throughput to the three servers that handle the splits.

Spanner also lets you create a hash function in a [generated column](/spanner/docs/generated-column/how-to) .

To do this in GoogleSQL, use the [FARM\_FINGERPRINT](/spanner/docs/reference/standard-sql/hash_functions) function during write time, as shown in the following example:

### GoogleSQL

``` text
CREATE TABLE UserAccessLogs (
ShardId INT64 NOT NULL
AS (MOD(FARM_FINGERPRINT(CAST(LastAccess AS STRING)), 2048)) STORED,
LastAccess TIMESTAMP NOT NULL,
UserId    INT64 NOT NULL,
) PRIMARY KEY (ShardId, LastAccess, UserId);
```

Your choice of hash function determines how well your insertions are spread across the key range. You don't need a cryptographic hash, although a cryptographic hash might be a good choice. When picking a hash function, you need to consider the following factors:

  - Hotspot avoidance. A function that results in more hash values tends to reduce hotspots.
  - Read efficiency. Reads across all hash values are faster if there are fewer hash values to scan.
  - Node count.

When using a `  ShardId  ` to prevent hotspots, use the following guidelines to choose the value of N, the number of logical shards:

  - **Correlate N with the number of nodes:** set N to be equal to the number of nodes you expect your instance to have. For example, if you expect your instance to scale up to 10 nodes, a value of N=10 is an effective starting point. This helps Spanner distribute the write load evenly across the nodes.

  - **N is a static value:** changing N after initial setup requires a schema update and potentially a data backfill. Therefore, you must choose a value for N that can accommodate your scaling needs.

  - **Avoid excessively large values for N:** while it might be tempting to choose a very large value for N to prepare for growth, it isn't generally necessary. More shards than the physical servers won't improve the performance significantly compared to the additional Spanner cost. Aligning N with the number of nodes is an effective strategy for distributing the workload.

## Use descending order for timestamp-based keys

If you have a table for your history that uses the timestamp as a key, consider using descending order for the key column if any of the following apply:

  - **If you want to read the most recent history, you're using an interleaved table for the history, and you're reading the parent row** . In this case, with a `  DESC  ` timestamp column, the latest history entries are stored adjacent to the parent row. Otherwise, reading the parent row and its recent history will require a seek in the middle to skip over the older history.
  - **If you're reading sequential entries in reverse chronological order, and you don't know exactly how far back you're going** . For example, you might use a SQL query with a `  LIMIT  ` to get the most recent N events, or you might plan to cancel the read after you've read a certain number of rows. In these cases, you want to start with the most recent entries and read sequentially older entries until your condition has been met, which Spanner does more efficiently for timestamp keys that Spanner stores in descending order.

Add the `  DESC  ` keyword to make the timestamp key descending. For example:

### GoogleSQL

``` text
CREATE TABLE UserAccessLogs (
UserId     INT64 NOT NULL,
LastAccess TIMESTAMP NOT NULL,
...
) PRIMARY KEY (UserId, LastAccess DESC);
```

**Schema design best practice \#2: Descending order or ascending order depends on the user queries, for example, top being the newest, or top being the oldest.**

## When to use an interleaved index

Similar to the previous primary key example that you should avoid, it's also a bad idea to create non-interleaved indexes on columns whose values are monotonically increasing or decreasing, even if they aren't primary key columns.

For example, suppose you define the following table, in which `  LastAccess  ` is a non-primary-key column:

### GoogleSQL

``` text
CREATE TABLE Users (
UserId     INT64 NOT NULL,
LastAccess TIMESTAMP,
...
) PRIMARY KEY (UserId);
```

### PostgreSQL

``` text
CREATE TABLE Users (
userid     bigint NOT NULL,
lastaccess TIMESTAMPTZ,
...
PRIMARY KEY (userid)
);
```

It might seem convenient to define an index on the `  LastAccess  ` column for quickly querying the database for user accesses "since time X", like this:

### GoogleSQL

``` text
CREATE NULL_FILTERED INDEX UsersByLastAccess ON Users(LastAccess);
```

### PostgreSQL

``` text
CREATE INDEX usersbylastaccess ON users(lastaccess)
WHERE lastaccess IS NOT NULL;
```

However, this results in the same pitfall as described in the previous best practice, because Spanner implements indexes as tables under the hood, and the resulting index table uses a column whose value monotonically increases as its first key part.

It's okay to create an interleaved index where last access rows are interleaved under the corresponding user row. This is because it's unlikely for a single parent row to produce thousands of events per second.

### GoogleSQL

``` text
CREATE NULL_FILTERED INDEX UsersByLastAccess
ON Users(UserId, LastAccess),
INTERLEAVE IN Users;
```

### PostgreSQL

``` text
CREATE INDEX usersbylastaccess ON users(userid, lastaccess)
WHERE lastaccess IS NOT NULL,
INTERLEAVE IN Users;
```

**Schema design best practice \#3: Don't create a non-interleaved index on a high write rate column whose value monotonically increases or decreases. Use an interleaved index, or use techniques like those you would use for the base table primary key design when designing index columns—for example, add \`shardId\`.**

## What's next

  - Look through [examples of schema designs](https://cloudplatform.googleblog.com/2018/06/What-DBAs-need-to-know-about-Cloud-Spanner-part-1-Keys-and-indexes.html) .
  - Learn about [bulk loading data](/spanner/docs/bulk-loading) .
