Spanner provides built-in tables that records the read (or query), write, and delete operations statistics for your tables (including change streams tables) and indexes. With table operations statistics you can do the following:

  - Identify tables with increased write traffic corresponding to storage increase.

  - Identify tables with unexpected read, write, and delete traffic.

  - Identify heavily-used tables.

When you query or write to a table, the corresponding operation count for the table increments by 1, regardless of the number of rows accessed.

Overall operations-per-second metrics of a database can be monitored with `  Operations per second  ` , `  Operations per second by API method  ` , and other related metrics in your [System Insights](/spanner/docs/monitoring-console) charts.

**Note:** The sum of the operation counts on all tables and indexes might not be equal to the total operations on a database. For example, one write to a table increments the `  write_count  ` on the table and on all indexes on the table. However, the write only counts as one operation on the database. The operation counts don't depend on the number of rows read or written to. They track the number of operations only. When the PartitionRead or PartitionQuery API returns multiple partition tokens, each Read or ExecuteSql call with a different token counts as a separate table operation.

## Access table operations statistics

Spanner provides the table operations statistics in the `  SPANNER_SYS  ` schema.You can use the following ways to access `  SPANNER_SYS  ` data:

  - A database's Spanner Studio page in the Google Cloud console.

  - The `  gcloud spanner databases execute-sql  ` command.

  - The [`  executeSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql) or the [`  executeStreamingSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql) method.

The following single read methods that Spanner provides don't support `  SPANNER_SYS  ` :

  - Performing a strong read from a single row or multiple rows in a table.
  - Performing a stale read from a single row or multiple rows in a table.
  - Reading from a single row or multiple rows in a secondary index.

## Table operations statistics

The following tables track the read (or query), write, and delete statistics on your tables and indexes during a specific time period:

  - `  SPANNER_SYS.TABLE_OPERATIONS_STATS_MINUTE  ` : Operations during 1 minute intervals
  - `  SPANNER_SYS.TABLE_OPERATIONS_STATS_10MINUTE  ` : Operations during 10 minute intervals
  - `  SPANNER_SYS.TABLE_OPERATIONS_STATS_HOUR  ` : Operations during 1 hour intervals

These tables have the following properties:

  - Each table contains data for non-overlapping time intervals of the length that the table name specifies.

  - Intervals are based on clock times. 1 minute intervals start on the minute, 10 minute intervals start every 10 minutes starting on the hour, and 1 hour intervals start on the hour.
    
    For example, at 11:59:30 AM, the most recent intervals available to SQL queries are:
    
      - **1 minute** : 11:58:00–11:58:59 AM
      - **10 minute** : 11:40:00–11:49:59 AM
      - **1 hour** : 10:00:00–10:59:59 AM

### Schema for all table operations statistics tables

<table>
<colgroup>
<col style="width: 40%" />
<col style="width: 10%" />
<col style="width: 50%" />
</colgroup>
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
<td>End of time interval in which the table sizes were collected.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       TABLE_NAME      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>Name of the table or the index.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       READ_QUERY_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Number of queries or reads reading from the table.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       WRITE_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Number of queries writing to the table.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       DELETE_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Number of queries performing deletes on the table.</td>
</tr>
</tbody>
</table>

If you insert data into your database using mutations, the `  write_count  ` increments by 1 for each table accessed by the insert statement. In addition, a query that accesses an index, without scanning the underlying table, only increments the `  read_query_count  ` on the index.

## Data retention

At a minimum, Spanner keeps data for each table for the following time periods:

  - `  SPANNER_SYS.TABLE_OPERATIONS_STATS_MINUTE  ` : Intervals covering the previous 6 hours.

  - `  SPANNER_SYS.TABLE_OPERATIONS_STATS_10MINUTE  ` : Intervals covering the previous 4 days.

  - `  SPANNER_SYS.TABLE_OPERATIONS_STATS_HOUR  ` : Intervals covering the previous 30 days.

**Note:** You cannot prevent Spanner from collecting table operations statistics. To delete the data in these tables, you must delete the database associated with the tables or wait until Spanner removes the data automatically.

### Example queries

This section includes several example SQL statements that retrieve aggregate table operations statistics. You can run these SQL statements using the [client libraries](/spanner/docs/reference/libraries) , or the [gcloud spanner](/spanner/docs/gcloud-spanner#execute_sql_statements) .

#### Query the tables and indexes with the most write operations for the most recent interval

``` text
    SELECT interval_end,
          table_name,
          write_count
    FROM spanner_sys.table_operations_stats_minute
    WHERE interval_end = (
          SELECT MAX(interval_end)
          FROM spanner_sys.table_operations_stats_minute)
    ORDER BY write_count DESC;
  
```

#### Query the tables and indexes with the most delete operations for the most recent interval

``` text
    SELECT interval_end,
          table_name,
          delete_count
    FROM spanner_sys.table_operations_stats_minute
    WHERE interval_end = (
          SELECT MAX(interval_end)
          FROM spanner_sys.table_operations_stats_minute)
    ORDER BY delete_count DESC;
  
```

#### Query the tables and indexes with the most read and query operations for the most recent interval

``` text
    SELECT interval_end,
          table_name,
          read_query_count
    FROM spanner_sys.table_operations_stats_minute
    WHERE interval_end = (
          SELECT MAX(interval_end)
          FROM spanner_sys.table_operations_stats_minute)
    ORDER BY read_query_count DESC;
  
```

#### Query the usage of a table over the last 6 hours

### GoogleSQL

``` text
    SELECT interval_end,
          read_query_count,
          write_count,
          delete_count
    FROM spanner_sys.table_operations_stats_minute
    WHERE table_name = "table_name"
    ORDER BY interval_end DESC;
    
```

Where:

  - `  table_name  ` must be an existing table or index in the database.

### PostgreSQL

``` text
    SELECT interval_end,
          read_query_count,
          write_count,
          delete_count
    FROM spanner_sys.table_operations_stats_minute
    WHERE table_name = 'table_name'
    ORDER BY interval_end DESC;
    
```

Where:

  - `  table_name  ` must be an existing table or index in the database.

#### Query the usage of a table over the last 14 days

### GoogleSQL

``` text
SELECT interval_end,
       read_query_count,
       write_count,
       delete_count
FROM spanner_sys.table_operations_stats_hour
WHERE interval_end > TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL -14 DAY)
      AND table_name = "table_name"
ORDER BY interval_end DESC;
```

Where:

  - `  table_name  ` must be an existing table or index in the database.

### PostgreSQL

``` text
SELECT interval_end,
   read_query_count,
   write_count,
   delete_count
FROM spanner_sys.table_operations_stats_hour
WHERE interval_end > spanner.timestamptz_subtract(now(), '14 DAY')
  AND table_name = 'table_name'
ORDER BY interval_end DESC;
```

Where:

  - `  table_name  ` must be an existing table or index in the database.

#### Query the tables and indexes with no usage in the last 24 hours

### GoogleSQL

``` text
(SELECT t.table_name
 FROM  information_schema.tables AS t
 WHERE t.table_catalog = ""
   AND t.table_schema = ""
   AND t.table_type = "BASE TABLE"
 UNION ALL
 SELECT cs.change_stream_name
 FROM information_schema.change_streams cs
 WHERE cs.change_stream_catalog = ""
   AND cs.change_stream_schema = ""
 UNION ALL
 SELECT idx.index_name
 FROM information_schema.indexes idx
 WHERE idx.index_type = "INDEX"
   AND idx.table_catalog = ""
   AND idx.table_schema = "")
 EXCEPT ALL
(SELECT  DISTINCT(table_name)
 FROM spanner_sys.table_operations_stats_hour
 WHERE interval_end > TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL -24 HOUR));
```

## What's next

  - Use [Table sizes statistics](/spanner/docs/introspection/table-sizes-statistics) to determine the sizes of your tables and indexes.

  - Learn about other [Introspection tools](/spanner/docs/introspection) .

  - Learn more about [SQL best practices](/spanner/docs/sql-best-practices) for Spanner.
