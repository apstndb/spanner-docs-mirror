Spanner provides built-in tables that record read, query, and write operation statistics for your table columns. With column operations statistics you can do the following:

  - Identify columns with unexpected read, query, and write traffic.

  - Identify heavily-used columns.

When you query or write to a column, Spanner increments the corresponding operation count for that column increments by one, regardless of the number of rows accessed.

You can monitor a database's overall using metrics that measure operations-per-second, operations per second by API method, and other related metrics within your [System Insights](https://docs.cloud.google.com/spanner/docs/monitoring-console) charts.

## Access column operations statistics

Spanner provides the column operations statistics in the `  SPANNER_SYS  ` schema. You can use the following to access `  SPANNER_SYS  ` data:

  - A database's Spanner Studio page in the Google Cloud console

  - The [`  gcloud spanner databases execute-sql  `](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/execute-sql) command

  - The [`  executeSql  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql) or the [`  executeStreamingSql  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql) method.

The following single read methods that Spanner provides don't support `  SPANNER_SYS  ` :

  - Performing a strong read from a single row or multiple rows in a table.
  - Performing a stale read from a single row or multiple rows in a table.
  - Reading from a single row or multiple rows in a secondary index.

For more information, see [Single read methods](https://docs.cloud.google.com/spanner/docs/reads#single_read_methods) .

## Column operations statistics

The following tables track the read, query, and write statistics on your columns during a specific time period:

  - `  SPANNER_SYS.COLUMN_OPERATIONS_STATS_MINUTE  ` : Operations during 1-minute intervals
  - `  SPANNER_SYS.COLUMN_OPERATIONS_STATS_10MINUTE  ` : Operations during 10-minute intervals
  - `  SPANNER_SYS.COLUMN_OPERATIONS_STATS_HOUR  ` : Operations during 1-hour intervals

These tables have the following properties:

  - Each table contains data for non-overlapping time intervals of the length that the table name specifies.

  - 1-minute intervals start on the minute, 10-minute intervals start every 10 minutes starting on the hour, and 1-hour intervals start on the hour.
    
    For example, at 11:59:30 AM, SQL queries can access the following most recent intervals:
    
      - **1 minute** : 11:58:00–11:58:59 AM
      - **10 minute** : 11:40:00–11:49:59 AM
      - **1 hour** : 10:00:00–10:59:59 AM

### Schema for all column operations statistics tables

| Column name                                   | Type                       | Description                                                               |
| --------------------------------------------- | -------------------------- | ------------------------------------------------------------------------- |
| `        INTERVAL_END       `                 | `        TIMESTAMP       ` | End of time interval in which the column usage statistics were collected. |
| `        TABLE_NAME       `                   | `        STRING       `    | Name of the table or the index.                                           |
| `        COLUMN_NAME       `                  | `        STRING       `    | Name of the column.                                                       |
| `        READ_COUNT       `                   | `        INT64       `     | Number of reads from the column.                                          |
| `        QUERY_COUNT       `                  | `        INT64       `     | Number of queries reading from the column.                                |
| `        WRITE_COUNT       `                  | `        INT64       `     | Number of queries writing to the table.                                   |
| `        IS_QUERY_CACHE_MEMORY_CAPPED       ` | `        BOOL       `      | Whether the statistics collection was capped due to memory pressure.      |

If you insert data into your database using mutations, Spanner increments the `  WRITE_COUNT  ` by 1 for each table that the insert statement accesses. In addition, a query that accesses an index without scanning the underlying table only increments the `  QUERY_COUNT  ` on the index.

## Data retention

At a minimum, Spanner keeps data for each table for the following time periods:

  - `  SPANNER_SYS.COLUMN_OPERATIONS_STATS_MINUTE  ` : Intervals covering the previous six hours.

  - `  SPANNER_SYS.COLUMN_OPERATIONS_STATS_10MINUTE  ` : Intervals covering the previous four days.

  - `  SPANNER_SYS.COLUMN_OPERATIONS_STATS_HOUR  ` : Intervals covering the previous 30 days.

**Note:** You can't prevent Spanner from collecting column operations statistics. To delete the data in these tables, you must delete the database associated with them or wait until Spanner removes the data after the data retention period ends.

### Example queries

This section includes several example SQL statements that retrieve aggregate column operations statistics. You can run these SQL statements using the [client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) or the [Google Cloud CLI](https://docs.cloud.google.com/spanner/docs/gcloud-spanner#execute_sql_statements) .

#### Query the table columns with the most write operations for the most recent interval

### GoogleSQL

``` 
    SELECT interval_end,
          table_name,
          column_name,
          write_count
    FROM spanner_sys.column_operations_stats_minute
    WHERE interval_end = (
          SELECT MAX(interval_end)
          FROM spanner_sys.column_operations_stats_minute)
    ORDER BY write_count DESC;
```

### PostgreSQL

``` 
    SELECT interval_end,
          table_name,
          column_name,
          write_count
    FROM spanner_sys.column_operations_stats_minute
    WHERE interval_end = (
          SELECT MAX(interval_end)
          FROM spanner_sys.column_operations_stats_minute)
    ORDER BY write_count DESC;
```

#### Query the columns with the most query operations for the most recent interval

### GoogleSQL

``` 
    SELECT interval_end,
          table_name,
          column_name,
          query_count
    FROM spanner_sys.column_operations_stats_minute
    WHERE interval_end = (
          SELECT MAX(interval_end)
          FROM spanner_sys.column_operations_stats_minute)
    ORDER BY query_count DESC;
```

### PostgreSQL

``` 
    SELECT interval_end,
          table_name,
          column_name,
          query_count
    FROM spanner_sys.column_operations_stats_minute
    WHERE interval_end = (
          SELECT MAX(interval_end)
          FROM spanner_sys.column_operations_stats_minute)
    ORDER BY query_count DESC;
```

#### Query the usage of a column over the last 6 hours

### GoogleSQL

``` 
    SELECT interval_end,
          read_count,
          query_count,
          write_count
    FROM spanner_sys.column_operations_stats_minute
    WHERE table_name = "table_name"
          AND column_name = "column_name"
    ORDER BY interval_end DESC;
    
```

Where:

  - `  table_name  ` must be an existing table or index in the database.
  - `  column_name  ` must be an existing column in the table.

### PostgreSQL

``` 
    SELECT interval_end,
          read_count,
          query_count,
          write_count
    FROM spanner_sys.column_operations_stats_minute
    WHERE table_name = 'table_name'
          AND column_name = 'column_name'
    ORDER BY interval_end DESC;
    
```

Where:

  - `  table_name  ` must be an existing table or index in the database.
  - `  column_name  ` must be an existing column in the table.

#### Query the usage of a column over the last 14 days

### GoogleSQL

    SELECT interval_end,
           read_count,
           query_count,
           write_count
    FROM spanner_sys.column_operations_stats_hour
    WHERE interval_end > TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL -14 DAY)
          AND table_name = "table_name"
          AND column_name = "column_name"
    ORDER BY interval_end DESC;

Make the following replacements:

  - `  table_name  ` : table or index name in the database.
  - `  column_name  ` : column name in the table.

### PostgreSQL

    SELECT interval_end,
       read_count,
       query_count,
       write_count
    FROM spanner_sys.column_operations_stats_hour
    WHERE interval_end > spanner.timestamptz_subtract(now(), '14 DAY')
      AND table_name = 'table_name'
      AND column_name = 'column_name'
    ORDER BY interval_end DESC;

Make the following replacements:

  - `  table_name  ` : table or index name in the database.
  - `  column_name  ` : column name in the table.

## What's next

  - Learn about other [Built-in statistics tables](https://docs.cloud.google.com/spanner/docs/introspection) .

  - Learn more about [SQL best practices](https://docs.cloud.google.com/spanner/docs/sql-best-practices) for Spanner.
