Spanner provides built-in tables that store statistics about reads. You can retrieve statistics from these `  SPANNER_SYS.READ_STATS*  ` tables using SQL statements.

**Note:** We will use the term **read shape** in this document to refer to the set of columns read in a read request. Read statistics track properties of these read shapes.

## When to use read statistics

Read statistics provide insight into how an application is using the database, and are useful when investigating performance issues. For example, you can check what read shapes are running against a database, how frequently they run and explain the performance characteristics of these read shapes. You can use the read statistics for your database to identify read shapes that result in high CPU usage. At a high level, read statistics will help you to understand the behavior of the traffic going into a database in terms of resource usage.

### Limitations

  - This tool is best suited for analyzing streams of similar reads that account for most of the CPU usage. It is not good for searching for reads that were run only one time.

  - The CPU usage tracked in these statistics represents Spanner server side CPU usage, excluding prefetch CPU usage and some other overheads.

  - Statistics are collected on a best-effort basis. As a result, it is possible for statistics to be missed if there are issues with underlying systems. For example, if there is internal networking issues, it is possible for some statistics to be missed.

## Access read statistics

**Note:** Spanner Studio (formerly labeled **Query** in the Google Cloud console) supports SQL, DML, and DDL operations in a single editor. For more information, see [Manage your data using the Google Cloud console](https://docs.cloud.google.com/spanner/docs/manage-data-using-console) .

Spanner provides the read statistics in the `  SPANNER_SYS  ` schema. You can use the following ways to access `  SPANNER_SYS  ` data:

  - A database's Spanner Studio page in the Google Cloud console.

  - The `  gcloud spanner databases execute-sql  ` command.

  - The [`  executeSql  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql) or the [`  executeStreamingSql  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql) method.

The following single read methods that Spanner provides don't support `  SPANNER_SYS  ` :

  - Performing a strong read from a single row or multiple rows in a table.
  - Performing a stale read from a single row or multiple rows in a table.
  - Reading from a single row or multiple rows in a secondary index.

## CPU usage grouped by read shape

The following tables track the read shapes with the highest CPU usage during a specific time period:

  - `  SPANNER_SYS.READ_STATS_TOP_MINUTE  ` : Read shape statistics aggregated across 1 minute intervals.
  - `  SPANNER_SYS.READ_STATS_TOP_10MINUTE  ` : Read shape statistics aggregated across 10 minute intervals.
  - `  SPANNER_SYS.READ_STATS_TOP_HOUR  ` : Read shape statistics aggregated across 1 hour intervals.

These tables have the following properties:

  - Each table contains data for non-overlapping time intervals of the length the table name specifies.

  - Intervals are based on clock times. 1 minute intervals end on the minute, 10 minute intervals end every 10 minutes starting on the hour, and 1 hour intervals end on the hour. After each interval, Spanner collects data from all servers and then makes the data available in the SPANNER\_SYS tables shortly thereafter.
    
    For example, at 11:59:30 AM, the most recent intervals available to SQL queries are:
    
      - **1 minute** : 11:58:00–11:58:59 AM
      - **10 minute** : 11:40:00–11:49:59 AM
      - **1 hour** : 10:00:00–10:59:59 AM

  - Spanner groups the statistics by read shape. If a tag is present, **FPRINT** is the hash of the tag. Otherwise, it is the hash of the `  READ_COLUMNS  ` value.

  - Each row contains statistics for all executions of a particular read shape that Spanner captures statistics for during the specified interval.

  - If Spanner is unable to store information about every distinct read shape run during the interval, the system prioritizes read shapes with the highest CPU usage during the specified interval.

### Table schema

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
<td>End of the time interval that the included read executions occurred in.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       REQUEST_TAG      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The optional request tag for this read operation. For more information about using tags, see <a href="https://docs.cloud.google.com/spanner/docs/introspection/troubleshooting-with-tags#request_tags">Troubleshooting with request tags</a> . Statistics for multiple reads that have the same tag string are grouped in a single row with the `REQUEST_TAG` matching that tag string.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       READ_TYPE      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>Indicates if a read is a <code dir="ltr" translate="no">       PARTITIONED_READ      </code> or <code dir="ltr" translate="no">       READ      </code> . A read with a partitionToken obtained from the PartitionRead API is represented by the <code dir="ltr" translate="no">       PARTITIONED_READ      </code> read type and the other read APIs by <code dir="ltr" translate="no">       READ      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       READ_COLUMNS      </code></td>
<td><code dir="ltr" translate="no">       ARRAY&lt;STRING&gt;      </code></td>
<td>The set of columns that were read. These are in alphabetical order.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       FPRINT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The hash of the <code dir="ltr" translate="no">       REQUEST_TAG      </code> value if present; Otherwise, the hash of the <code dir="ltr" translate="no">       READ_COLUMNS      </code> value.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       EXECUTION_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>Number of times Spanner executed the read shape during the interval.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_ROWS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of rows that the read returned.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_BYTES      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of data bytes that the read returned, excluding transmission encoding overhead.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_CPU_SECONDS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of Spanner server side CPU seconds executing the read, excluding prefetch CPU and other overhead.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_LOCKING_DELAY_SECONDS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of seconds spent waiting due to locking.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AVG_CLIENT_WAIT_SECONDS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of seconds spent waiting due to the client not consuming data as fast as Spanner could generate it.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_LEADER_REFRESH_DELAY_SECONDS      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td>Average number of seconds spent waiting to confirm with the Paxos leader that all writes have been observed.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       RUN_IN_RW_TRANSACTION_EXECUTION_COUNT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The number of times the read was run as part of a read-write transaction. This column helps you determine if you can avoid lock contentions by moving the read to a read-only transaction.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       AVG_DISK_IO_COST      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td><p>The average cost of this query in terms of Spanner HDD <a href="https://docs.cloud.google.com/monitoring/api/metrics_gcp_p_z#spanner/instance/disk_load">disk load</a> .</p>
<p>Use this value to make relative HDD I/O cost comparisons between reads that you run in the database. Querying data on HDD storage incurs a charge against the HDD disk load capacity of the instance. A higher value indicates that you are using more HDD disk load and your query might be slower than if it was running on SSD. Furthermore, if your HDD disk load is at capacity, the performance of your queries might be further impacted. You can monitor the total <a href="https://docs.cloud.google.com/monitoring/api/metrics_gcp_p_z#spanner/instance/disk_load">HDD disk load</a> capacity of the instance as a percentage. To add more HDD disk load capacity, you can add more processing units or nodes to your instance. For more information, see <a href="https://docs.cloud.google.com/spanner/docs/create-manage-instances#change-compute-capacity">Change the compute capacity</a> . To improve query performance, also consider moving some data to SSD.</p>
<p>For workloads that consume a lot of disk I/O, we recommend that you store frequently accessed data on SSD storage. Data accessed from SSD doesn't consume HDD disk load capacity. You can store selective tables, columns, or secondary indexes on SSD storage as needed, while keeping infrequently accessed data on HDD storage. For more information, see <a href="https://docs.cloud.google.com/spanner/docs/tiered-storage">Tiered storage overview</a> .</p></td>
</tr>
</tbody>
</table>

### Example queries

This section includes several example SQL statements that retrieve read statistics. You can run these SQL statements using the [client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) , the [gcloud spanner](https://docs.cloud.google.com/spanner/docs/gcloud-spanner#execute_sql_statements) , or the [Google Cloud console](https://docs.cloud.google.com/spanner/docs/create-query-database-console#run_a_query) .

#### List the basic statistics for each read shape in a given time period

The following query returns the raw data for the top read shapes in the most recent 1-minute time intervals.

    SELECT fprint,
           read_columns,
           execution_count,
           avg_cpu_seconds,
           avg_rows,
           avg_bytes,
           avg_locking_delay_seconds,
           avg_client_wait_seconds
    FROM spanner_sys.read_stats_top_minute
    ORDER BY interval_end DESC LIMIT 3;

##### Query output

| fprint                        | read\_columns                                   | execution\_count         | avg\_cpu\_seconds                       | avg\_rows               | avg\_bytes                  | avg\_locking\_delay\_seconds            | avg\_client\_wait\_seconds |
| ----------------------------- | ----------------------------------------------- | ------------------------ | --------------------------------------- | ----------------------- | --------------------------- | --------------------------------------- | -------------------------- |
| `        125062082139       ` | `        ["Singers.id", "Singers.name"]       ` | `        8514387       ` | `        0.000661355290396507       `   | `        310.79       ` | `        205       `        | `        8.3232564943763752e-06       ` | `        0       `         |
| `        151238888745       ` | `        ["Singers.singerinfo"]       `         | `        3341542       ` | `        6.5992827184280315e-05       ` | `        12784       `  | `        54       `         | `        4.6859741349028595e-07       ` | `        0       `         |
| `        14105484       `     | `        ["Albums.id", "Albums.title"]       `  | `        9306619       ` | `        0.00017855774721667873       ` | `        1165.4       ` | `        2964.71875       ` | `        1.4328191393074178e-06       ` | `        0       `         |

#### List the read shapes, ordered by highest total CPU usage

The following query returns the read shapes with the highest CPU usage in the most recent hour:

    SELECT read_columns,
           execution_count,
           avg_cpu_seconds,
           execution_count * avg_cpu_seconds AS total_cpu
    FROM spanner_sys.read_stats_top_hour
    WHERE interval_end =
      (SELECT MAX(interval_end)
       FROM spanner_sys.read_stats_top_hour)
    ORDER BY total_cpu DESC LIMIT 3;

##### Query output

| read\_columns                                   | execution\_count      | avg\_cpu\_seconds                       | total\_cpu                         |
| ----------------------------------------------- | --------------------- | --------------------------------------- | ---------------------------------- |
| `        ["Singers.id", "Singers.name"]       ` | `        1647       ` | `        0.00023380297430622681       ` | `        0.2579       `            |
| `        ["Albums.id", "Albums.title"]       `  | `        720       `  | `        0.00016738889440282034       ` | `        0.221314999999999       ` |
| `        ["Singers.singerinfo""]       `        | `        3223       ` | `        0.00037764625882302246       ` | `        0.188053       `          |

## Aggregate statistics

`  SPANNER_SYS  ` also contains tables to store aggregate read statistics captured by Spanner in a specific time period:

  - `  SPANNER_SYS.READ_STATS_TOTAL_MINUTE  ` : Aggregate statistics for all read shapes during 1 minute intervals.
  - `  SPANNER_SYS.READ_STATS_TOTAL_10MINUTE  ` : Aggregate statistics for all read shapes during 10 minute intervals.
  - `  SPANNER_SYS.READ_STATS_TOTAL_HOUR  ` : Aggregate statistics for all read shapes during 1 hour intervals.

Aggregate statistics tables have the following properties:

  - Each table contains data for non-overlapping time intervals of the length the table name specifies.

  - Intervals are based on clock times. 1 minute intervals end on the minute, 10 minute intervals end every 10 minutes starting on the hour, and 1 hour intervals end on the hour.
    
    For example, at 11:59:30 AM, the most recent intervals available to SQL queries on aggregate read statistics are:
    
      - **1 minute** : 11:58:00–11:58:59 AM
      - **10 minute** : 11:40:00–11:49:59 AM
      - **1 hour** : 10:00:00–10:59:59 AM

  - Each row contains statistics for **all** read shapes executed over the database during the specified interval, aggregated together. There is only one row per time interval.

  - The statistics captured in the `  SPANNER_SYS.READ_STATS_TOTAL_*  ` tables might include read shapes that Spanner did not capture in the `  SPANNER_SYS.READ_STATS_TOP_*  ` tables.

  - Some columns in these tables are exposed as metrics in Cloud Monitoring. The exposed metrics are:
    
      - Rows returned count
      - Read execution count
      - Read CPU time
      - Locking delays
      - Client wait time
      - Leader refresh delay
      - Bytes returned count
    
    For more information, see [Spanner metrics](https://docs.cloud.google.com/monitoring/api/metrics_gcp_p_z#gcp-spanner) .

### Table schema

| Column name                                            | Type                       | Description                                                                                                                                                                                   |
| ------------------------------------------------------ | -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `        INTERVAL_END       `                          | `        TIMESTAMP       ` | End of the time interval that the included read shape executions occurred in.                                                                                                                 |
| `        EXECUTION_COUNT       `                       | `        INT64       `     | Number of times Spanner executed the read shape during the interval.                                                                                                                          |
| `        AVG_ROWS       `                              | `        FLOAT64       `   | Average number of rows that the reads returned.                                                                                                                                               |
| `        AVG_BYTES       `                             | `        FLOAT64       `   | Average number of data bytes that the reads returned, excluding transmission encoding overhead.                                                                                               |
| `        AVG_CPU_SECONDS       `                       | `        FLOAT64       `   | Average number of Spanner server side CPU seconds executing the read, excluding prefetch CPU and other overhead.                                                                              |
| `        AVG_LOCKING_DELAY_SECONDS       `             | `        FLOAT64       `   | Average number of seconds spent waiting due to locking.                                                                                                                                       |
| `        AVG_CLIENT_WAIT_SECONDS       `               | `        FLOAT64       `   | Average number of seconds spent waiting due to throttling.                                                                                                                                    |
| `        AVG_LEADER_REFRESH_DELAY_SECONDS       `      | `        FLOAT64       `   | Average number of seconds spent coordinating the reads across instances in [multi-region configurations](https://docs.cloud.google.com/spanner/docs/instances) .                              |
| `        RUN_IN_RW_TRANSACTION_EXECUTION_COUNT       ` | `        INT64       `     | The number of times that reads were run as part of read-write transactions. This column helps you determine if you can avoid lock contentions by moving some reads to read-only transactions. |

### Example queries

This section includes several example SQL statements that retrieve aggregate read statistics. You can run these SQL statements using the [client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) , the [gcloud spanner](https://docs.cloud.google.com/spanner/docs/gcloud-spanner#execute_sql_statements) , or the [Google Cloud console](https://docs.cloud.google.com/spanner/docs/create-query-database-console#run_a_query) .

#### Find the total CPU usage across all read shapes

The following query returns the number of CPU hours consumed by read shapes in the most recent hour:

    SELECT (avg_cpu_seconds * execution_count / 60 / 60)
      AS total_cpu_hours
    FROM spanner_sys.read_stats_total_hour
    WHERE interval_end =
      (SELECT MAX(interval_end)
       FROM spanner_sys.read_stats_total_hour);

##### Query output

| total\_cpu\_hours                       |
| --------------------------------------- |
| `        0.00026186111111111115       ` |

#### Find the total execution count in a given time period

The following query returns the total number of read shapes executed in the most recent complete 1-minute interval:

    SELECT interval_end,
           execution_count
    FROM spanner_sys.read_stats_total_minute
    WHERE interval_end =
      (SELECT MAX(interval_end)
       FROM spanner_sys.read_stats_total_minute);

##### Query output

| interval\_end                              | execution\_count          |
| ------------------------------------------ | ------------------------- |
| `        2020-05-28 11:02:00-07:00       ` | `        12861966       ` |

## Data retention

At a minimum, Spanner keeps data for each table for the following time periods:

  - `  SPANNER_SYS.READ_STATS_TOP_MINUTE  ` and `  SPANNER_SYS.READ_STATS_TOTAL_MINUTE  ` : Intervals covering the previous 6 hours.

  - `  SPANNER_SYS.READ_STATS_TOP_10MINUTE  ` and `  SPANNER_SYS.READ_STATS_TOTAL_10MINUTE  ` : Intervals covering the previous 4 days.

  - `  SPANNER_SYS.READ_STATS_TOP_HOUR  ` and `  SPANNER_SYS.READ_STATS_TOTAL_HOUR  ` : Intervals covering the previous 30 days.

**Note:** You can't prevent Spanner from collecting read statistics. To delete the data in these tables, you must delete the database associated with the tables or wait until Spanner removes the data automatically. The retention period for these tables is fixed. If you want to keep statistics for longer periods of time, we recommend that you periodically copy data out of these tables.

## Troubleshoot high CPU usage with read statistics

Spanner read statistics come in handy in cases where you need to investigate high CPU usage on your Spanner database or when you are just trying to understand the CPU-heavy read shapes on your database. Inspection of read shapes that use significant amounts of database resources gives Spanner users a potential way to reduce operational costs and possibly improve general system latencies. Using the following steps, we'll show you how to use read statistics to investigate high CPU usage in your database.

### Select a time period to investigate

Start your investigation by looking for a time when your application began to experience high CPU usage. For example, in the following scenario, the issue started occurring around 5:20pm on May 28th 2020.

### Gather read statistics for the selected time period

Having selected a time period to start our investigation, we'll look at statistics gathered in the `  READ_STATS_TOTAL_10MINUTE  ` table around that time. The results of this query might give us clues about how CPU and other read statistics changed over that period of time. The following query returns the aggregated read statistics from `  4:30 pm  ` to `  7:30 pm  ` (inclusive).

    SELECT
      interval_end,
      ROUND(avg_cpu_seconds,4) as avg_cpu_seconds,
      execution_count,
      avg_locking_delay_seconds
    FROM SPANNER_SYS.READ_STATS_TOTAL_10MINUTE
    WHERE
      interval_end >= "2020-05-28T16:30:00"
      AND interval_end <= "2020-05-28T19:30:00"
    ORDER BY interval_end;

The following data is an example of the result we get back from our query.

| interval\_end                 | avg\_cpu\_seconds | execution\_count | avg\_locking\_delay\_seconds |
| :---------------------------- | ----------------: | ---------------: | :--------------------------- |
| **2020-05-28 16:40:00-07:00** |        **0.0004** |     **11111421** | **8.3232564943763752e-06**   |
| 2020-05-28 16:50:00-07:00     |            0.0002 |          8815637 | 8.98734051776406e-05         |
| 2020-05-28 17:00:00-07:00     |            0.0001 |          8260215 | 6.039129247846453e-06        |
| 2020-05-28 17:10:00-07:00     |            0.0001 |          8514387 | 9.0535466616680686e-07       |
| **2020-05-28 17:20:00-07:00** |        **0.0006** |     **13715466** | **2.6801485272173765e-06**   |
| **2020-05-28 17:30:00-07:00** |        **0.0007** |     **12861966** | **4.6859741349028595e-07**   |
| **2020-05-28 17:40:00-07:00** |        **0.0007** |      **3755954** | **2.7131391918005383e-06**   |
| **2020-05-28 17:50:00-07:00** |        **0.0006** |      **4248137** | **1.4328191393074178e-06**   |
| **2020-05-28 18:00:00-07:00** |        **0.0006** |      **3986198** | **2.6973481999639748e-06**   |
| **2020-05-28 18:10:00-07:00** |        **0.0006** |      **3510249** | **3.7577083563017905e-06**   |
| 2020-05-28 18:20:00-07:00     |            0.0004 |          3341542 | 4.0940589703795433e-07       |
| 2020-05-28 18:30:00-07:00     |            0.0002 |          8695147 | 1.9914494947583975e-05       |
| 2020-05-28 18:40:00-07:00     |            0.0003 |         11679702 | 1.8331461539001595e-05       |
| 2020-05-28 18:50:00-07:00     |            0.0003 |          9306619 | 1.2527332321222135e-05       |
| 2020-05-28 19:00:00-07:00     |            0.0002 |          8520508 | 6.2268448078447915e-06       |
| **2020-05-28 19:10:00-07:00** |        **0.0006** |     **13715466** | **2.6801485272173765e-06**   |
| **2020-05-28 19:20:00-07:00** |        **0.0005** |     **11947323** | **3.3029114639321295e-05**   |
| 2020-05-28 19:30:00-07:00     |            0.0002 |          8514387 | 9.0535466616680686e-07       |

Here we see that average CPU time, `  avg_cpu_seconds  ` , is higher in the **highlighted** intervals. The `  interval_end  ` with the value **2020-05-28 19:20:00** has a higher CPU time, so we'll choose that interval to investigate further in the next step.

### Find which read shapes are causing high CPU usage

Digging a little deeper, we now query the `  READ_STATS_TOP_10MINUTE  ` table for the interval which was picked in the preceding step. The results of this query can help indicate which read shapes cause high CPU usage.

    SELECT
      read_columns,
      ROUND(avg_cpu_seconds,4) as avg_cpu_seconds,
      execution_count,
      avg_rows
    FROM SPANNER_SYS.READ_STATS_TOP_10MINUTE
    WHERE
      interval_end = "2020-05-28T19:20:00"
    ORDER BY avg_cpu_seconds DESC LIMIT 3;

The following data as an example of the result we get back from our query, returning information about the top three read shapes ranked by `  avg_cpu_seconds  ` . Note the use of [`  ROUND  `](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#round) in our query to restrict the output of `  avg_cpu_seconds  ` to 4 decimal places.

| read\_columns                                                                                                                            | avg\_cpu\_seconds | execution\_count | avg\_rows      |
| ---------------------------------------------------------------------------------------------------------------------------------------- | ----------------: | ---------------: | :------------- |
| `        [TestHigherLatency._exists,TestHigherLatency.lang_status,TestHigherLatency.score,globalTagAffinity.shares]       ` <sup>1</sup> |            0.4192 |             1182 | 11650.42216582 |
| `        [TestHigherLatency._exists,TestHigherLatency.lang_status,TestHigherLatency.likes,globalTagAffinity.score]       `               |            0.0852 |                4 | 12784          |
| `        [TestHigherLatency._exists,TestHigherLatency.lang_status,TestHigherLatency.score,globalTagAffinity.ugcCount]       `            |            0.0697 |             1140 | 310.7921052631 |

<sup>1</sup> `  _exists  ` is an internal field that is used to check whether a certain row exists or not.

One reason for high CPU usage might be that you start to execute a few read shapes more frequently ( `  execution_count  ` ). Perhaps the average number of rows that the read returned has increased ( `  avg_rows  ` ). If none of those properties of the read shape reveal anything interesting, you can examine other properties such as `  avg_locking_delay_seconds  ` , `  avg_client_wait_seconds  ` or `  avg_bytes  ` .

### Apply best practices to reduce high CPU usage

When you have gone through the preceding steps, consider whether supplying any of these best practices will help your situation.

  - The number of times Spanner executed read shapes during the interval is a good example of a metric that needs a baseline to tell you if a measurement is reasonable or a sign of a problem. Having established a baseline for the metric, you'll be able to detect and investigate the cause of any unexpected deviations from normal behavior.

  - If CPU Usage is relatively constant most of the time, but suddenly shows a spike that can be correlated with a similar sudden spike in user requests or application behavior, it might be an indication that everything is working as expected.

  - Try the following query to find the top read shapes ranked by the number of times Spanner executed for each read shape:
    
        SELECT interval_end, read_columns, execution_count
        FROM SPANNER_SYS.READ_STATS_TOP_MINUTE
        ORDER BY execution_count DESC
        LIMIT 10;

  - If you are looking for the lowest possible read latencies, especially when using multi-region instance configurations, use [stale reads instead of strong reads](https://docs.cloud.google.com/spanner/docs/reads#read_types) to reduce or remove the `  AVG_LEADER_REFRESH_DELAY_SECONDS  ` component of read latency.
    
    **Note:** Stale reads don't provide any latency benefits in regional configurations, so you should almost always use strong reads when your instance does not have a multi-region configuration.

  - If you are only doing reads, and you can express your read using a [single read method](https://docs.cloud.google.com/spanner/docs/reads#single_read_methods) , you should use that single read method. Single reads don't lock, unlike read-write transactions, therefore you should use read-only transactions over more expensive read-write transactions when you are not writing data.

## What's next

  - Learn about other [Introspection tools](https://docs.cloud.google.com/spanner/docs/introspection) .
  - Learn about other information Spanner stores for each database in the database's [information schema](https://docs.cloud.google.com/spanner/docs/information-schema) tables.
  - Learn more about [SQL best practices](https://docs.cloud.google.com/spanner/docs/sql-best-practices) for Spanner.
  - Learn more about [Investigating high CPU utilization](https://docs.cloud.google.com/spanner/docs/introspection/investigate-cpu-utilization) .
