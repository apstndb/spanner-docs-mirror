This page describes how to detect and debug hotspots in your database. You can access statistics about hotspots in splits with both GoogleSQL and PostgreSQL.

Spanner stores your data as a contiguous key space, ordered by the primary keys of your tables and indexes. A split is a range of rows from a set of tables or an index. The split's start is called the *split start* . The *split limit* sets the end of the split. The split includes the split start, but not the split limit.

In Spanner, *hotspots* are situations where too many requests are sent to the same server which saturates the resources of the server and potentially causes high latencies. The splits affected by hotspots are known as *hot* or *warm* splits.

A split's hotspot statistic (identified in the system as `  CPU_USAGE_SCORE  ` ) is a measurement of the load on a split that's constrained by the resources available on the server. This measurement is given as a percentage. If more than 50% of the load on a split is constrained by the available resources, then the split is considered warm. If 100% of the load on a split is constrained, then the split is considered hot. Such hot splits might also have an effect on the latency of the requests served by them.

The `  CPU_USAGE_SCORE  ` of a split can remain constant or vary over time based on the workload accessing the split and changes in the split boundaries.

Based on the warm and hot split resource constraints, Spanner might use [load-based splitting](/spanner/docs/schema-and-data-model#load-based_splitting) to evenly distribute the load across the key space. The warm and hot splits can be moved across the instance's servers for load balancing. Spanner performs load-based splitting in the background, minimizing the impact on the latencies. However, Spanner might not be able to balance the load, even after multiple attempts at splitting, due to [anti-patterns](/spanner/docs/whitepapers/optimizing-schema-design#anti-patterns) in the application. Hence, persistent warm or hot splits that last at least 10 minutes might need further troubleshooting and potential application changes.

The Spanner hot split statistics help you identify the splits where hotspots occur. You can then make changes to your application or schema, as needed. You can retrieve these statistics from the `  SPANNER_SYS.SPLIT_STATS_TOP_MINUTE  ` system tables using SQL statements.

## Access hot split statistics

Spanner provides the hot split statistics in the `  SPANNER_SYS  ` schema. `  SPANNER_SYS  ` data is available only through GoogleSQL and PostgreSQL interfaces. You can use the following ways to access this data:

  - A database's [Spanner Studio page](/spanner/docs/manage-data-using-console) in the Google Cloud console.
  - The [`  gcloud spanner databases execute-sql  `](/sdk/gcloud/reference/spanner/databases/execute-sql) command.
  - The [`  executeSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql) or the [`  executeStreamingSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql) method.

The following single read methods that Spanner provides don't support `  SPANNER_SYS  ` :

  - Performing a strong read from a single row or multiple rows in a table.
  - Performing a stale read from a single row or multiple rows in a table.
  - Reading from a single row or multiple rows in a secondary index.

## Hot split statistics

You use the following table to track hot splits:

  - `  SPANNER_SYS.SPLIT_STATS_TOP_MINUTE  ` : shows splits that are hot during 1-minute intervals.

These tables have the following properties:

  - Each table contains data for non-overlapping time intervals of the duration the table name specifies.

  - Intervals are based on clock times:
    
      - 1-minute intervals end on the minute.

  - After each interval, Spanner collects data from all servers and then makes the data available in the `  SPANNER_SYS  ` tables shortly thereafter.
    
    For example, at 11:59:30 AM, the most recent intervals available to SQL queries are:
    
      - 1 minute: 11:58:00-11:58:59 AM

  - Spanner groups the statistics by splits.

  - Each row contains a percentage that indicates how hot or warm a split is, for each split that Spanner captures statistics for during the specified interval.

  - If less than 50% of the load on a split is constrained by the available resources, then Spanner doesn't capture the statistic. If Spanner is unable to store all the hot splits during the interval, the system prioritizes the splits with the highest `  CPU_USAGE_SCORE  ` percentage during the specified interval. If there are no splits returned, it's an indication of the absence of any hotspots.

## Table schema

The following table shows the table schema for the following stats:

  - `  SPANNER_SYS.SPLIT_STATS_TOP_MINUTE  `

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       INTERVAL_END      </code></td>
<td><code dir="ltr" translate="no">       TIMESTAMP      </code></td>
<td>End of the time interval during which the split was hot</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       SPLIT_START      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The starting key of the range of rows in the split. The split start might also be <code dir="ltr" translate="no">       &lt;begin&gt;      </code> , indicating the beginning of the key space</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       SPLIT_LIMIT      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The limit key for the range of rows in the split. The limit: key might also be <code dir="ltr" translate="no">       &lt;end&gt;      </code> , indicating the end of the key space|</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       CPU_USAGE_SCORE      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The <code dir="ltr" translate="no">       CPU_USAGE_SCORE      </code> percentage of the splits. A <code dir="ltr" translate="no">       CPU_USAGE_SCORE      </code> percentage of 50% indicates the presence of warm or hot | splits |</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AFFECTED_TABLES      </code></td>
<td><code dir="ltr" translate="no">       STRING ARRAY      </code></td>
<td>The tables whose rows might be in the split</td>
</tr>
</tbody>
</table>

**Note:** Splits with `  CPU_USAGE_SCORE  ` percentages that are less than 100% can be used t0 track trends over time. Spanner uses load-based splitting to balance the load across servers and hence you might see the splits change or shrink over time. However, Spanner might not be able to balance the load due to problematic patterns in the application and hot splits might remain after several iterations of Spanner load-based splitting. Hence, the presence of hot splits for over 10 minutes is an indication that you might need to investigate further.

## Split start and split limit keys

A split is a contiguous row range of a database, and is defined by its *start* and *limit* keys. A split can be a single row, a narrow row range, or a wide row range, and the split can include multiple tables or indexes.

The `  SPLIT_START  ` and `  SPLIT_LIMIT  ` columns identify the primary keys of a warm or hot split.

### Example schema

The following schema is an example table for the topics in this page.

### GoogleSQL

``` text
CREATE TABLE Users (
  UserId INT64 NOT NULL,
  FirstName STRING(MAX),
  LastName STRING(MAX),
) PRIMARY KEY(UserId);

CREATE INDEX UsersByFirstName ON Users(FirstName DESC);

CREATE TABLE Threads (
  UserId INT64 NOT NULL,
  ThreadId INT64 NOT NULL,
  Starred BOOL,
) PRIMARY KEY(UserId, ThreadId),
  INTERLEAVE IN PARENT Users ON DELETE CASCADE;

CREATE TABLE Messages (
  UserId INT64 NOT NULL,
  ThreadId INT64 NOT NULL,
  MessageId INT64 NOT NULL,
  Subject STRING(MAX),
  Body STRING(MAX),
) PRIMARY KEY(UserId, ThreadId, MessageId),
  INTERLEAVE IN PARENT Threads ON DELETE CASCADE;

CREATE INDEX MessagesIdx ON Messages(UserId, ThreadId, Subject),
INTERLEAVE IN Threads;
```

### PostgreSQL

``` text
CREATE TABLE users
(
   userid    BIGINT NOT NULL PRIMARY KEY,-- INT64 to BIGINT
   firstname VARCHAR(max),-- STRING(MAX) to VARCHAR(MAX)
   lastname  VARCHAR(max)
);

CREATE INDEX usersbyfirstname
  ON users(firstname DESC);

CREATE TABLE threads
  (
    userid   BIGINT NOT NULL,
    threadid BIGINT NOT NULL,
    starred  BOOLEAN, -- BOOL to BOOLEAN
    PRIMARY KEY (userid, threadid),
    CONSTRAINT fk_threads_user FOREIGN KEY (userid) REFERENCES users(userid) ON
    DELETE CASCADE -- Interleave to Foreign Key constraint
  );

CREATE TABLE messages
  (
    userid    BIGINT NOT NULL,
    threadid  BIGINT NOT NULL,
    messageid BIGINT NOT NULL PRIMARY KEY,
    subject   VARCHAR(max),
    body      VARCHAR(max),
    CONSTRAINT fk_messages_thread FOREIGN KEY (userid, threadid) REFERENCES
    threads(userid, threadid) ON DELETE CASCADE
  -- Interleave to Foreign Key constraint
  );

CREATE INDEX messagesidx ON messages(userid, threadid, subject), REFERENCES
threads(userid, threadid);
```

Imagine your key space looks like this:

<table>
<thead>
<tr class="header">
<th>PRIMARY KEY</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       &lt;begin&gt;      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       Users()      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       Threads()      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       Users(2)      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       Users(3)      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       Threads(3)      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       Threads(3,"a")      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       Messages(3,"a",1)      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       Messages(3,"a",2)      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       Threads(3, "aa")      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       Users(9)      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       Users(10)      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       Threads(10)      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       UsersByFirstName("abc")      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       UsersByFirstName("abcd")      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       &lt;end&gt;      </code></td>
</tr>
</tbody>
</table>

### Example of splits

The following shows some example splits to help you understand what splits look like.

The `  SPLIT_START  ` and `  SPLIT_LIMIT  ` might indicate the row of a table or index, or they can be `  <begin>  ` and `  <end>  ` , representing the boundaries of the key space of the database. The `  SPLIT_START  ` and `  SPLIT_LIMIT  ` might also contain truncated keys, which are keys preceding any full key in the table. For example, `  Threads(10)  ` is a prefix for any `  Threads  ` row interleaved in `  Users(10)  ` .

<table>
<thead>
<tr class="header">
<th>SPLIT_START</th>
<th>SPLIT_LIMIT</th>
<th>AFFECTED_TABLES</th>
<th>EXPLANATION</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       Users(3)      </code></td>
<td><code dir="ltr" translate="no">       Users(10)      </code></td>
<td><code dir="ltr" translate="no">       UsersByFirstName      </code> , <code dir="ltr" translate="no">       Users      </code> , <code dir="ltr" translate="no">       Threads      </code> , <code dir="ltr" translate="no">       Messages      </code> , <code dir="ltr" translate="no">       MessagesIdx      </code></td>
<td>Split starts at row with <code dir="ltr" translate="no">       UserId=3      </code> and ends at the row before the row with <code dir="ltr" translate="no">       UserId = 10      </code> . The split contains the <code dir="ltr" translate="no">       Users      </code> table rows and all its interleaved tables rows for <code dir="ltr" translate="no">       UserId=3      </code> to 10.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       Messages(3,"a",1)      </code></td>
<td><code dir="ltr" translate="no">       Threads(3,"aa")      </code></td>
<td><code dir="ltr" translate="no">       Threads      </code> , <code dir="ltr" translate="no">       Messages      </code> , <code dir="ltr" translate="no">       MessagesIdx      </code></td>
<td>The split starts at the row with <code dir="ltr" translate="no">       UserId=3      </code> , <code dir="ltr" translate="no">       ThreadId="a"      </code> and <code dir="ltr" translate="no">       MessageId=1      </code> and ends at the row preceding the row with the key of <code dir="ltr" translate="no">       UserId=3      </code> and <code dir="ltr" translate="no">       ThreadsId = "aa"      </code> . The split contains all the tables between <code dir="ltr" translate="no">       Messages(3,"a",1)      </code> and <code dir="ltr" translate="no">       Threads(3,"aa")      </code> . As the <code dir="ltr" translate="no">       split_start      </code> and <code dir="ltr" translate="no">       split_limit      </code> are interleaved in the same top-level table row, the split contains the interleaved tables rows between the start and limit. See <a href="/spanner/docs/schema-and-data-model#create_a_hierarchy_of_interleaved_tables">schemas-overview</a> to understand how interleaved tables are co-located.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       Messages(3,"a",1)      </code></td>
<td><code dir="ltr" translate="no">       &lt;end&gt;      </code></td>
<td><code dir="ltr" translate="no">       UsersByFirstName      </code> , <code dir="ltr" translate="no">       Users      </code> , <code dir="ltr" translate="no">       Threads      </code> , <code dir="ltr" translate="no">       Messages      </code> , <code dir="ltr" translate="no">       MessagesIdx      </code></td>
<td>The split starts in the messages table at the row with key <code dir="ltr" translate="no">       UserId=3      </code> , <code dir="ltr" translate="no">       ThreadId="a"      </code> and <code dir="ltr" translate="no">       MessageId=1      </code> . The split hosts all the rows from the <code dir="ltr" translate="no">       split_start      </code> to <code dir="ltr" translate="no">       &lt;end&gt;      </code> , the end of the key space of the database. All the rows of the tables following the <code dir="ltr" translate="no">       split_start      </code> , like <code dir="ltr" translate="no">       Users(4)      </code> are included in the split.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       &lt;begin&gt;      </code></td>
<td><code dir="ltr" translate="no">       Users(9)      </code></td>
<td><code dir="ltr" translate="no">       UsersByFirstName      </code> , <code dir="ltr" translate="no">       Users      </code> , <code dir="ltr" translate="no">       Threads      </code> , <code dir="ltr" translate="no">       Messages      </code> , <code dir="ltr" translate="no">       MessagesIdx      </code></td>
<td>The split starts at <code dir="ltr" translate="no">       &lt;begin&gt;      </code> , the beginning of the key space of the database and ends at the row preceding the <code dir="ltr" translate="no">       Users      </code> row with <code dir="ltr" translate="no">       UserId=9      </code> . So the split has all the table rows preceding <code dir="ltr" translate="no">       Users      </code> and all the rows of <code dir="ltr" translate="no">       Users      </code> table preceding <code dir="ltr" translate="no">       UserId=9      </code> and the rows of its interleaved tables.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       Messages(3,"a",1)      </code></td>
<td><code dir="ltr" translate="no">       Threads(10)      </code></td>
<td><code dir="ltr" translate="no">       UsersByFirstName      </code> , <code dir="ltr" translate="no">       Users      </code> , <code dir="ltr" translate="no">       Threads      </code> , <code dir="ltr" translate="no">       Messages      </code> , <code dir="ltr" translate="no">       MessagesIdx      </code></td>
<td>Split starts at <code dir="ltr" translate="no">       Messages(3,"a", 1)      </code> interleaved in <code dir="ltr" translate="no">       Users(3)      </code> and ends at the row preceding <code dir="ltr" translate="no">       Threads(10)      </code> . <code dir="ltr" translate="no">       Threads(10)      </code> is a truncated split key that is a prefix of any key of the Threads table interleaved in <code dir="ltr" translate="no">       Users(10)      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       Users()      </code></td>
<td><code dir="ltr" translate="no">       &lt;end&gt;      </code></td>
<td><code dir="ltr" translate="no">       UsersByFirstName      </code> , <code dir="ltr" translate="no">       Users      </code> , <code dir="ltr" translate="no">       Threads      </code> , <code dir="ltr" translate="no">       Messages      </code> , <code dir="ltr" translate="no">       MessagesIdx      </code></td>
<td>The split starts at the truncated split key of <code dir="ltr" translate="no">       Users()      </code> which precedes any full key of the <code dir="ltr" translate="no">       Users      </code> table. The split extends until the end of the possible key space in the database. The affected_tables hence cover the <code dir="ltr" translate="no">       Users      </code> table, its interleaved tables and indexes and all the tables that might appear after users.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       Threads(10)      </code></td>
<td><code dir="ltr" translate="no">       UsersByFirstName("abc")      </code></td>
<td><code dir="ltr" translate="no">       UsersByFirstName      </code> , <code dir="ltr" translate="no">       Users      </code> , <code dir="ltr" translate="no">       Threads      </code> , <code dir="ltr" translate="no">       Messages      </code> , <code dir="ltr" translate="no">       MessagesIdx      </code></td>
<td>The split starts at the <code dir="ltr" translate="no">       Threads      </code> row with <code dir="ltr" translate="no">       UserId = 10      </code> and ends at the index, <code dir="ltr" translate="no">       UsersByFirstName      </code> at the key preceding <code dir="ltr" translate="no">       "abc"      </code> .</td>
</tr>
</tbody>
</table>

**Note:** `  SPLIT_LIMIT  ` is greater than the `  SPLIT_START  ` . However, lexicographical comparisons of the split start and limit might not work across splits.

### Example queries to find hot splits

The following example shows a SQL statement that you can use to retrieve the hot split statistics. You can run these SQL statements using the client libraries, gcloud, or the Google Cloud console.

### GoogleSQL

``` text
SELECT t.split_start,
       t.split_limit,
       t.cpu_usage_score,
       t.affected_tables,
FROM   SPANNER_SYS.SPLIT_STATS_TOP_MINUTE t
WHERE  t.interval_end =
  (SELECT MAX(interval_end)
  FROM    SPANNER_SYS.SPLIT_STATS_TOP_MINUTE)
ORDER BY  t.cpu_usage_score DESC;
```

### PostgreSQL

``` text
SELECT t.split_start,
       t.split_limit,
       t.cpu_usage_score,
       t.affected_tables
FROM   SPANNER_SYS.SPLIT_STATS_TOP_MINUTE t
WHERE  t.interval_end = (
  SELECT MAX(interval_end)
  FROM   SPANNER_SYS.SPLIT_STATS_TOP_MINUTE
)
ORDER BY t.cpu_usage_score DESC;
```

The query output looks like the following:

<table>
<thead>
<tr class="header">
<th><code dir="ltr" translate="no">       SPLIT_START      </code></th>
<th><code dir="ltr" translate="no">       SPLIT_LIMIT      </code></th>
<th><code dir="ltr" translate="no">       CPU_USAGE_SCORE      </code></th>
<th><code dir="ltr" translate="no">       AFFECTED_TABLES      </code></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       Users(13)      </code></td>
<td><code dir="ltr" translate="no">       Users(76)      </code></td>
<td><code dir="ltr" translate="no">       82      </code></td>
<td><code dir="ltr" translate="no">       Messages,Users,Threads      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       Users(101)      </code></td>
<td><code dir="ltr" translate="no">       Users(102)      </code></td>
<td><code dir="ltr" translate="no">       90      </code></td>
<td><code dir="ltr" translate="no">       Messages,Users,Threads      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       Threads(10, "a")      </code></td>
<td><code dir="ltr" translate="no">       Threads(10, "aa")      </code></td>
<td><code dir="ltr" translate="no">       100      </code></td>
<td><code dir="ltr" translate="no">       Messages,Threads      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       Messages(631, "abc", 1)      </code></td>
<td><code dir="ltr" translate="no">       Messages(631, "abc", 3)      </code></td>
<td><code dir="ltr" translate="no">       100      </code></td>
<td><code dir="ltr" translate="no">       Messages      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       Threads(12, "zebra")      </code></td>
<td><code dir="ltr" translate="no">       Users(14)      </code></td>
<td><code dir="ltr" translate="no">       76      </code></td>
<td><code dir="ltr" translate="no">       Messages,Users,Threads      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       Users(620)      </code></td>
<td><code dir="ltr" translate="no">       &lt;end&gt;      </code></td>
<td><code dir="ltr" translate="no">       100      </code></td>
<td><code dir="ltr" translate="no">       Messages,Users,Threads      </code></td>
</tr>
</tbody>
</table>

## Data retention for the hot split statistics

At a minimum, Spanner keeps data for each table for the following time period:

  - `  SPANNER_SYS.SPLIT_STATS_TOP_MINUTE  ` : Intervals that cover the previous 6 hours.

**Note:** You can't prevent Spanner from collecting the hot split statistics. To delete the data in these tables, you must delete the database associated with the tables or wait until Spanner removes the data automatically. The retention period for these tables is fixed. If you want to keep statistics for longer periods of time, we recommend that you periodically copy data out of these tables.

## Troubleshoot hotspots using hot split statistics

This section describes how to detect and troubleshoot hotspots.

### Select a time period to investigate

Check the [latency metrics](/spanner/docs/latency-metrics) for your Spanner database to find the time period when your application experienced high latency and CPU usage. For example, it might show you that an issue started around 10:50 PM on May 18, 2024.

### Find persistent hotspotting

As Spanner balances your load with [load-based splitting](/spanner/docs/schema-and-data-model#load-based_splitting) , we recommend that you investigate if hotspotting has continued for more than 10 minutes. You can do so by querying the `  SPANNER_SYS.SPLIT_STATS_TOP_MINUTE  ` table, as shown in the following example:

### GoogleSQL

``` text
SELECT Count(DISTINCT t.interval_end)
FROM   SPANNER_SYS.SPLIT_STATS_TOP_MINUTE t
WHERE  t.cpu_usage_score >= 50
  AND  t.interval_end >= "interval_end_date_time"
  AND  t.interval_end <= "interval_end_date_time";
```

Replace interval\_end\_date\_time with the date and time for the interval, using the format `  2024-05-18T17:40:00Z  ` .

### PostgreSQL

``` text
SELECT COUNT(DISTINCT t.interval_end)
FROM   SPLIT_STATS_TOP_MINUTE t
WHERE  t.cpu_usage_score >= 50
  AND  t.interval_end >= 'interval_end_date_time'::timestamptz
  AND  t.interval_end <= 'interval_end_date_time'::timestamptz;
```

Replace interval\_end\_date\_time with the date and time for the interval, using the format `  2024-05-18T17:40:00Z  ` .

If the previous query result is equal to 10, it means that your database is experiencing hotspotting that might need further debugging.

### Find the splits with the highest `     CPU_USAGE_SCORE    ` level

For this example, we run the following SQL to find the row ranges with the highest `  CPU_USAGE_SCORE  ` level:

### GoogleSQL

``` text
SELECT t.split_start,
       t.split_limit,
       t.affected_tables,
       t.cpu_usage_score
FROM   SPANNER_SYS.SPLIT_STATS_TOP_MINUTE t
WHERE  t.cpu_usage_score >= 50
  AND  t.interval_end = "interval_end_date_time";
```

Replace interval\_end\_date\_time with the date and time for the interval, using the format `  2024-05-18T17:40:00Z  ` .

### PostgreSQL

``` text
SELECT t.split_start,
       t.split_limit,
       t.affected_tables,
       t.cpu_usage_score
FROM   SPLIT_STATS_TOP_MINUTE t
WHERE  t.cpu_usage_score = 100
  AND  t.interval_end = 'interval_end_date_time'::timestamptz;
```

Replace interval\_end\_date\_time with the date and time for the interval, using the format `  2024-05-18T17:40:00Z  ` .

The previous SQL outputs the following:

<table>
<thead>
<tr class="header">
<th><code dir="ltr" translate="no">       SPLIT_START      </code></th>
<th><code dir="ltr" translate="no">       SPLIT_LIMIT      </code></th>
<th><code dir="ltr" translate="no">       CPU_USAGE_SCORE      </code></th>
<th><code dir="ltr" translate="no">       AFFECTED_TABLES      </code></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       Users(180)      </code></td>
<td><code dir="ltr" translate="no">       &lt;end&gt;      </code></td>
<td><code dir="ltr" translate="no">       85      </code></td>
<td><code dir="ltr" translate="no">       Messages,Users,Threads      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       Users(24)      </code></td>
<td><code dir="ltr" translate="no">       Users(76)      </code></td>
<td><code dir="ltr" translate="no">       76      </code></td>
<td><code dir="ltr" translate="no">       Messages,Users,Threads      </code></td>
</tr>
</tbody>
</table>

From this table of results, we can see that hotspots occurred on two splits. Spanner [load-based splitting](/spanner/docs/schema-and-data-model#load-based_splitting) might try to resolve hotspots on these splits. However, it might not be able to do so if there are problematic patterns in the schema or workload. To detect if there are splits that need your intervention, we recommend tracking the splits for at least 10 minutes. For example, the following SQL tracks the first split over the last ten minutes.

### GoogleSQL

``` text
SELECT t.interval_end,
       t.split_start,
       t.split_limit,
       t.cpu_usage_score
FROM   SPANNER_SYS.SPLIT_STATS_TOP_MINUTE t
WHERE  t.split_start = "users(180)"
  AND  t.split_limit = "<end>"
  AND  t.interval_end >= "interval_end_date_time"
  AND  t.interval_end <= "interval_end_date_time";
```

Replace interval\_end\_date\_time with the date and time for the interval, using the format `  2024-05-18T17:40:00Z  ` .

### PostgreSQL

``` text
SELECT t.interval_end,
       t.split_start,
       t.split_limit,
       t.cpu_usage_score
FROM   SPANNER_SYS.SPLIT_STATS_TOP_MINUTE t
WHERE  t.split_start = 'users(180)'
  AND  t.split_limit = ''
  AND  t.interval_end >= 'interval_end_date_time'::timestamptz
  AND  t.interval_end <= 'interval_end_date_time'::timestamptz;
```

Replace interval\_end\_date\_time with the date and time for the interval, using the format `  2024-05-18T17:40:00Z  ` .

The previous SQL outputs the following:

<table>
<thead>
<tr class="header">
<th><code dir="ltr" translate="no">       INTERVAL_END      </code></th>
<th><code dir="ltr" translate="no">       SPLIT_START      </code></th>
<th><code dir="ltr" translate="no">       SPLIT_LIMIT      </code></th>
<th><code dir="ltr" translate="no">       CPU_USAGE_SCORE      </code></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-18T17:46:00Z      </code></td>
<td><code dir="ltr" translate="no">       Users(180)      </code></td>
<td><code dir="ltr" translate="no">       &lt;end&gt;      </code></td>
<td><code dir="ltr" translate="no">       85      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       2024-05-18T17:47:00Z      </code></td>
<td><code dir="ltr" translate="no">       Users(180)      </code></td>
<td><code dir="ltr" translate="no">       &lt;end&gt;      </code></td>
<td><code dir="ltr" translate="no">       85      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-18T17:48:00Z      </code></td>
<td><code dir="ltr" translate="no">       Users(180)      </code></td>
<td><code dir="ltr" translate="no">       &lt;end&gt;      </code></td>
<td><code dir="ltr" translate="no">       85      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       2024-05-18T17:49:00Z      </code></td>
<td><code dir="ltr" translate="no">       Users(180)      </code></td>
<td><code dir="ltr" translate="no">       &lt;end&gt;      </code></td>
<td><code dir="ltr" translate="no">       85      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-18T17:50:00Z      </code></td>
<td><code dir="ltr" translate="no">       Users(180)      </code></td>
<td><code dir="ltr" translate="no">       &lt;end&gt;      </code></td>
<td><code dir="ltr" translate="no">       85      </code></td>
</tr>
</tbody>
</table>

The split seems to have been hot for the past few minutes. You might observe the split for longer to determine that the Spanner load-based splitting mitigates the hotspot. There might be cases wherein Spanner can't load balance any further.

For example, query the `  SPANNER_SYS.SPLIT_STATS_TOP_MINUTE  ` table. See the following example scenarios.

### GoogleSQL

``` text
SELECT t.interval_end,
      t.split_start,
      t.split_limit,
      t.cpu_usage_score
FROM  SPANNER_SYS.SPLIT_STATS_TOP_MINUTE t
WHERE t.interval_end >= "interval_end_date_time"
      AND t.interval_end <= "interval_end_date_time";
```

Replace interval\_end\_date\_time with the date and time for the interval, using the format `  2024-05-18T17:40:00Z  ` .

### PostgreSQL

``` text
SELECT t.interval_end,
       t.split_start,
       t.split_limit,
       t._cpu_usage
FROM   SPANNER_SYS.SPLIT_STATS_TOP_MINUTE t
WHERE  t.interval_end >= 'interval_end_date_time'::timestamptz
  AND  t.interval_end <= 'interval_end_date_time'::timestamptz;
```

Replace interval\_end\_date\_time with the date and time for the interval, using the format `  2024-05-18T17:40:00Z  ` .

### Single hot row

In the following example, it looks like `  Threads(10,"spanner")  ` is in a single row split which remained hot for over 10 minutes. This could happen when there's a persistent load on a popular row.

<table>
<thead>
<tr class="header">
<th><code dir="ltr" translate="no">       INTERVAL_END      </code></th>
<th><code dir="ltr" translate="no">       SPLIT_START      </code></th>
<th><code dir="ltr" translate="no">       SPLIT_LIMIT      </code></th>
<th><code dir="ltr" translate="no">       CPU_USAGE_SCORE      </code></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-16T20:40:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner")      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner1")      </code></td>
<td><code dir="ltr" translate="no">       62      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       2024-05-16T20:41:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner")      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner1")      </code></td>
<td><code dir="ltr" translate="no">       62      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-16T20:42:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner")      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner1")      </code></td>
<td><code dir="ltr" translate="no">       62      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       2024-05-16T20:43:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner")      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner1")      </code></td>
<td><code dir="ltr" translate="no">       62      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-16T20:44:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner")      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner1")      </code></td>
<td><code dir="ltr" translate="no">       62      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       2024-05-16T20:45:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner")      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner1")      </code></td>
<td><code dir="ltr" translate="no">       62      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-16T20:46:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner")      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner1")      </code></td>
<td><code dir="ltr" translate="no">       80      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       2024-05-16T20:47:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner")      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner1")      </code></td>
<td><code dir="ltr" translate="no">       80      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-16T20:48:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner")      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner1")      </code></td>
<td><code dir="ltr" translate="no">       80      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       2024-05-16T20:49:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner")      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner1")      </code></td>
<td><code dir="ltr" translate="no">       100      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-16T20:50:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner")      </code></td>
<td><code dir="ltr" translate="no">       Threads(10,"spanner1")      </code></td>
<td><code dir="ltr" translate="no">       100      </code></td>
</tr>
</tbody>
</table>

Spanner can't balance the load for this single key as it can't be split further.

### Moving hotspot

In the following example, the load moves through contiguous splits over time, moving to a new split across time intervals.

<table>
<thead>
<tr class="header">
<th><code dir="ltr" translate="no">       INTERVAL_END      </code></th>
<th><code dir="ltr" translate="no">       SPLIT_START      </code></th>
<th><code dir="ltr" translate="no">       SPLIT_LIMIT      </code></th>
<th><code dir="ltr" translate="no">       CPU_USAGE_SCORE      </code></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-16T20:40:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(1,"a")      </code></td>
<td><code dir="ltr" translate="no">       Threads(1,"aa")      </code></td>
<td><code dir="ltr" translate="no">       100      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       2024-05-16T20:41:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(1,"aa")      </code></td>
<td><code dir="ltr" translate="no">       Threads(1,"ab")      </code></td>
<td><code dir="ltr" translate="no">       100      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-16T20:42:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(1,"ab")      </code></td>
<td><code dir="ltr" translate="no">       Threads(1,"c")      </code></td>
<td><code dir="ltr" translate="no">       100      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       2024-05-16T20:43:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(1,"c")      </code></td>
<td><code dir="ltr" translate="no">       Threads(1,"ca")      </code></td>
<td><code dir="ltr" translate="no">       100      </code></td>
</tr>
</tbody>
</table>

This could occur, for example, due to a workload that reads or writes keys in monotonically increasing order. Spanner can't balance the load to mitigate the effects of this application behavior.

### Normal load balancing

Spanner tries to balance the load by adding more splits or moving splits around. The following example shows what that might look like.

<table>
<thead>
<tr class="header">
<th><code dir="ltr" translate="no">       INTERVAL_END      </code></th>
<th><code dir="ltr" translate="no">       SPLIT_START      </code></th>
<th><code dir="ltr" translate="no">       SPLIT_LIMIT      </code></th>
<th><code dir="ltr" translate="no">       CPU_USAGE_SCORE      </code></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-16T20:40:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(1000,"zebra")      </code></td>
<td><code dir="ltr" translate="no">       &lt;end&gt;      </code></td>
<td><code dir="ltr" translate="no">       82      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       2024-05-16T20:41:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(1000,"zebra")      </code></td>
<td><code dir="ltr" translate="no">       &lt;end&gt;      </code></td>
<td><code dir="ltr" translate="no">       90      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-16T20:42:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(1000,"zebra")      </code></td>
<td><code dir="ltr" translate="no">       &lt;end&gt;      </code></td>
<td><code dir="ltr" translate="no">       100      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       2024-05-16T20:43:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(1000,"zebra")      </code></td>
<td><code dir="ltr" translate="no">       Threads(2000,"spanner")      </code></td>
<td><code dir="ltr" translate="no">       100      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-16T20:44:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(1200,"c")      </code></td>
<td><code dir="ltr" translate="no">       Threads(2000)      </code></td>
<td><code dir="ltr" translate="no">       92      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       2024-05-16T20:45:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(1500,"c")      </code></td>
<td><code dir="ltr" translate="no">       Threads(1700,"zach")      </code></td>
<td><code dir="ltr" translate="no">       76      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-16T20:46:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(1700)      </code></td>
<td><code dir="ltr" translate="no">       Threads(1700,"c")      </code></td>
<td><code dir="ltr" translate="no">       76      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       2024-05-16T20:47:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(1700)      </code></td>
<td><code dir="ltr" translate="no">       Threads(1700,"c")      </code></td>
<td><code dir="ltr" translate="no">       50      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       2024-05-16T20:48:00Z      </code></td>
<td><code dir="ltr" translate="no">       Threads(1700)      </code></td>
<td><code dir="ltr" translate="no">       Threads(1700,"c")      </code></td>
<td><code dir="ltr" translate="no">       39      </code></td>
</tr>
</tbody>
</table>

Here, the larger split at 2024-05-16T17:40:00Z was split further into a smaller split and as a result, the `  CPU_USAGE_SCORE  ` statistic decreased. Spanner might not create splits into individual rows. The splits mirror the workload causing the high `  CPU_USAGE_SCORE  ` statistic.

If you have observed a persistent hot split for over 10 minutes, see [Best practices to mitigate hotspots](#best_practices_to_mitigate_hotspots) .

## Best practices to mitigate hotspots

**Note:** If you've had an increase in your load and have recently upscaled your instance, Spanner might take a few minutes to perform its load-balancing operations before your latency decreases.

If load-balancing doesn't decrease latency, the next step is to identify the cause of the hotspots. After that, options are to either reduce the hotspotting workload, or optimize the application schema and logic to avoid hotspots.

#### Identify the cause

  - Use [Lock & Transaction Insights](/spanner/docs/use-lock-and-transaction-insights) to look for transactions that have high lock wait time where the row range start key is within the hot split.

  - Use [Query Insights](/spanner/docs/using-query-insights) to look for queries that read from the table that contains the hot split, and have recently increased latency, or a higher ratio of latency to CPU.

  - Use [Oldest Active Queries](/spanner/docs/introspection/oldest-active-queries) to look for queries that read from the table that contains the hot split, and that have higher than expected latency.

Some special cases to watch for:

  - Check to see if [time to live (TTL)](/spanner/docs/ttl/monitoring-and-metrics) was enabled recently. If there are a lot of splits from old data, then TTL can raise `  CPU_USAGE_SCORE  ` levels during mass deletes. In this case, the issue should self-resolve once the initial deletions complete.

#### Optimize the workload

  - Follow [SQL best practices](/spanner/docs/sql-best-practices) . Consider stale reads, writes that don't perform reads first, or adding indexes.
  - Follow [Schema best practices](/spanner/docs/schema-design) . Ensure your schema is designed to handle load balancing and avoid hotspots.

## What's next

  - Learn about [schema design best practices](/spanner/docs/schema-design) .
  - Learn about [Key Visualizer](/spanner/docs/key-visualizer) .
  - Look through [examples of schema designs](https://cloudplatform.googleblog.com/2018/06/What-DBAs-need-to-know-about-Cloud-Spanner-part-1-Keys-and-indexes.html) .
  - Learn how to use the [split insights dashboard](/spanner/docs/find-hotspots-in-database) to detect hotspots.
