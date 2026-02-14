This page describes how to use CPU utilization metrics and charts, along with other introspection tools, to investigate high CPU usage in your database.

## Identify whether a system or user task is causing high CPU utilization

The [Google Cloud console](/spanner/docs/monitoring-console) provides several monitoring tools for Spanner, allowing you to see the status of the most essential metrics for your instance. One of these is a chart called **CPU utilization - Total** . This chart shows you the total CPU utilization, as a percentage of the instance's CPU resources, broken down by task priority and operation type. There are two types of tasks: *user tasks* , such as reads and writes, and *system tasks* , which covers automated background tasks like compaction and index backfilling.

**Figure 1** shows an example of the **CPU utilization - Total** chart.

**Figure 1.** **CPU utilization - total** chart in the Monitoring dashboard in Google Cloud console.

Now, imagine you receive an alert from Cloud Monitoring that CPU usage has increased significantly. You open the **Monitoring** dashboard for your instance in Google Cloud console and examine the **CPU Utilization - Total** chart in the Google Cloud console. As shown in **Figure 1** , you can see the increased CPU utilization from high-priority user tasks. The next step is to find out what high-priority user operation is causing this CPU usage increase.

You can visualize this and other metrics on a time series by using [Query insights](/spanner/docs/using-query-insights#the_dashboard) dashboards. These prebuilt dashboards help you view spikes in CPU utilization and identify inefficient queries.

## Identify what user operation is causing the CPU utilization spike

The **CPU utilization - Total** chart in **Figure 1** shows that high-priority user tasks are the cause of higher CPU usage.

Next, you'll examine the **CPU Utilization by operation type** chart in Google Cloud console. This chart shows the CPU utilization broken down by high-, medium-, and low-priority user-initiated operations.

### What is a user-initiated operation?

A *user-initiated* operation is an operation that is initiated through an API request. Spanner groups these requests into operation types or categories, and you can display each operation type as a line on the **CPU utilization by operation type** chart. The following table describes the API methods that are included in each operation type.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Operation</th>
<th>API methods</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>read_readonly</td>
<td>Read<br />
StreamingRead</td>
<td>Includes reads which fetch rows from the database using key lookups and scans.</td>
</tr>
<tr class="even">
<td>read_readwrite</td>
<td>Read<br />
StreamingRead</td>
<td>Includes reads inside read-write transactions.</td>
</tr>
<tr class="odd">
<td>read_withpartitiontoken</td>
<td>Read<br />
StreamingRead</td>
<td>Includes read operations performed using a set of partition tokens.</td>
</tr>
<tr class="even">
<td>executesql_select_readonly</td>
<td>ExecuteSql<br />
ExecuteStreamingSql</td>
<td>Includes execute Select SQL statement and change stream queries.</td>
</tr>
<tr class="odd">
<td>executesql_select_readwrite</td>
<td>ExecuteSql<br />
ExecuteStreamingSql</td>
<td>Includes execute Select statement inside read-write transactions.</td>
</tr>
<tr class="even">
<td>executesql_select_withpartitiontoken</td>
<td>ExecuteSql<br />
ExecuteStreamingSql</td>
<td>Includes execute Select statement performed using a set of partition tokens.</td>
</tr>
<tr class="odd">
<td>executesql_dml_readwrite</td>
<td>ExecuteSql<br />
ExecuteStreamingSql<br />
ExecuteBatchDml</td>
<td>Includes execute DML SQL statement.</td>
</tr>
<tr class="even">
<td>executesql_dml_partitioned</td>
<td>ExecuteSql<br />
ExecuteStreamingSql<br />
ExecuteBatchDml</td>
<td>Includes execute Partitioned DML SQL statement.</td>
</tr>
<tr class="odd">
<td>beginorcommit</td>
<td>BeginTransaction<br />
Commit<br />
Rollback</td>
<td>Includes begin, commit, and rollback transactions.</td>
</tr>
<tr class="even">
<td>misc</td>
<td>PartitionQuery<br />
PartitionRead<br />
GetSession<br />
CreateSession</td>
<td>Includes PartitionQuery, PartitionRead, Create Database, Create Instance, session related operations, internal time-critical serving operations, etc.</td>
</tr>
</tbody>
</table>

Here's an example chart of the **CPU utilization by operation types** metric.

**Figure 2.** **CPU utilization by operation type** chart in Google Cloud console.

You can limit display to a specific priority by using the **Priority** menu on top of the chart. It plots each operation type or category on a line graph. The categories listed below the chart identify each graph. You can hide and show each graph by selecting or deselecting its respective category filter.

Alternatively, you can also create this chart in metrics explorer as described below:

### Create a chart for CPU utilization by operations type in Metrics Explorer

1.  In the Google Cloud console, select **Monitoring** , or use the following button:
2.  Select **Metrics Explorer** in the navigation pane.
3.  In the **Find resource type and metric** field, enter the value `  spanner.googleapis.com/instance/cpu/utilization_by_operation_type  ` , then select the row that appears below the box.
4.  In the **Filter** field, enter the value `  instance_id  ` , then enter the instance ID you want to examine and click **\>Apply** .
5.  In the **Group By** field, select `  category  ` from the dropdown list. The chart will show CPU utilization of user tasks grouped by operation type, or category.

While the **CPU utilization by priority** metric in the preceding section helped determine whether a user or system task caused an increase in CPU resource usage, with the **CPU utilization by operation type** metric you can dig deeper and find out the type of user-initiated operation behind this rise in CPU usage.

## Identify which user request is contributing to increased CPU usage

To determine which specific user request is responsible for the spike in CPU utilization in the **executesql\_select\_readonly** operation type graph you see in **Figure 2** , you'll use the built-in introspection statistics tables to gain more insight.

Use the following table as a guide to determine which statistics table to query based on the operation type that is causing high CPU usage.

<table>
<thead>
<tr class="header">
<th>Operation type</th>
<th style="text-align: center;">Query</th>
<th style="text-align: center;">Read</th>
<th style="text-align: center;">Transaction</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>read_readonly</td>
<td style="text-align: center;">No</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">No</td>
</tr>
<tr class="even">
<td>read_readwrite</td>
<td style="text-align: center;">No</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">Yes</td>
</tr>
<tr class="odd">
<td>read_withpartitiontoken</td>
<td style="text-align: center;">No</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">No</td>
</tr>
<tr class="even">
<td>executesql_select_readonly</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">No</td>
<td style="text-align: center;">No</td>
</tr>
<tr class="odd">
<td>executesql_select_withpartitiontoken</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">No</td>
<td style="text-align: center;">No</td>
</tr>
<tr class="even">
<td>executesql_select_readwrite</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">No</td>
<td style="text-align: center;">Yes</td>
</tr>
<tr class="odd">
<td>executesql_dml_readwrite</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">No</td>
<td style="text-align: center;">Yes</td>
</tr>
<tr class="even">
<td>executesql_dml_partitioned</td>
<td style="text-align: center;">No</td>
<td style="text-align: center;">No</td>
<td style="text-align: center;">Yes</td>
</tr>
<tr class="odd">
<td>beginorcommit</td>
<td style="text-align: center;">No</td>
<td style="text-align: center;">No</td>
<td style="text-align: center;">Yes</td>
</tr>
</tbody>
</table>

**Note:** Query statistics and read statistics tables both contain CPU usage data. However, the transaction statistics table doesn't contain CPU usage data per transaction shape. To troubleshoot elevated CPU using transaction statistics, you can use `  AVG_TOTAL_LATENCY_SECONDS  ` or `  AVG_COMMIT_LATENCY_SECONDS  ` because, as the latency increases, CPU usage to process the transaction increases accordingly.

For example, if **read\_withpartitiontoken** is the problem, troubleshoot using [read statistics](/spanner/docs/introspection/read-statistics) .

In this scenario, the **executesql\_select\_readonly** operation seems to be the reason for the CPU usage increase you are observing. Based on the preceding table, you should look at [query statistics](/spanner/docs/introspection/query-statistics) next to find out what queries are expensive, run frequently or scan a lot of data.

To find out the queries with the highest CPU usage in the previous hour, you can run the following query on the `  query_stats_top_hour  ` statistics table.

``` text
SELECT text,
       execution_count AS count,
       avg_latency_seconds AS latency,
       avg_cpu_seconds AS cpu,
       execution_count * avg_cpu_seconds AS total_cpu
FROM spanner_sys.query_stats_top_hour
WHERE interval_end =
  (SELECT MAX(interval_end)
   FROM spanner_sys.query_stats_top_hour)
ORDER BY total_cpu DESC;
```

The output will show queries sorted by CPU usage. Once you identify the query with the highest CPU usage, you can try the following options to tune it.

  - Review the [query execution plan](/spanner/docs/query-execution-plans) to identify any possible inefficiencies that might contribute to high CPU utilization.

  - Review your query to make sure it is following [SQL best practices](/spanner/docs/sql-best-practices) .

  - Review the database [schema design](/spanner/docs/schema-design) and update the schema to allow for more efficient queries.

  - Establish a baseline for the number of times Spanner executes a query during an interval. Using this baseline, you'll be able to detect and investigate the cause of any unexpected deviations from normal behavior.

If you didn't manage to find a CPU-intensive query, add [compute capacity](/spanner/docs/compute-capacity) to the instance. Adding compute capacity provides more CPU resources and enables Spanner to handle a larger workload. For more information, see [Increasing compute capacity](/spanner/docs/cpu-utilization#add-compute-capacity) .

## What's next

  - Learn about [CPU utilization metrics](/spanner/docs/cpu-utilization) .

  - Learn about other [Introspection tools](/spanner/docs/introspection) .

  - Learn about [Monitoring with Cloud Monitoring](/spanner/docs/monitoring-cloud) .

  - Learn more about [SQL best practices](/spanner/docs/sql-best-practices) for Spanner.

  - See the list of [Metrics from Spanner](/monitoring/api/metrics_gcp_p_z#gcp-spanner) .
