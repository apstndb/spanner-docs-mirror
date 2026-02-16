This document describes how to use the system insights dashboard to monitor Spanner instances and databases.

## About system insights

The system insights dashboard displays scorecards and charts with respect to a selected instance or database, and provides measures of latencies, CPU utilization, storage, throughput, and other performance statistics. You can view charts for selectable time periods, ranging from the past 1 hour to the past 30 days.

The system insights dashboard includes the following sections, with numbers corresponding to the following UI screenshot:

1.  **Insight selectors:** Select the databases, [instance partitions](/spanner/docs/geo-partitioning) , and regions that populate the dashboard. System insights shows instance partitions and region selections when multiple instance partitions or regions are available in the instance.
2.  **Time range filter:** Filter statistics by a time range, such as hours, days, or a custom range.
3.  **Dashboard selector:** Select user-customized views, or reset system insights to the default *predefined* view.
4.  **Annotations:** Select insight alert event types to annotate charts.
5.  **Customize dashboards:** Customize the appearance, placement, and content of dashboard widgets and the system insights dashboard. This document describes the predefined dashboard presentation.
6.  **Scorecards:** Display statistics at a point of time, over the selected period.
7.  **Charts:** Display charts of CPU utilization, throughput, latencies, storage use, and more. Insight alerts set by **Annotations** appear on charts with bell icons.

## Required roles

To get the permissions that you need to view or modify insights dashboards, including custom dashboards, ask your administrator to grant you the following IAM roles on the project:

  - To create and edit custom dashboards: [Monitoring Dashboard Configuration Editor](/iam/docs/roles-permissions/monitoring#monitoring.dashboardEditor) ( `  roles/monitoring.dashboardEditor  ` )
  - To open and view Metrics Explorer charts: [Monitoring Dashboard Configuration Viewer](/iam/docs/roles-permissions/monitoring#monitoring.dashboardViewer) ( `  roles/monitoring.dashboardViewer  ` )
  - To create and edit Metrics Explorer alerts: [Monitoring Editor](/iam/docs/roles-permissions/monitoring#monitoring.editor) ( `  roles/monitoring.editor  ` )

For more information about granting roles, see [Manage access to projects, folders, and organizations](/iam/docs/granting-changing-revoking-access) .

These predefined roles contain the permissions required to view or modify insights dashboards, including custom dashboards. To see the exact permissions that are required, expand the **Required permissions** section:

#### Required permissions

The following permissions are required to view or modify insights dashboards, including custom dashboards:

  - To create custom dashboards: `  monitoring.dashboards.create  `
  - To edit custom dashboards: `  monitoring.dashboards.update  `
  - To view custom dashboards: `  monitoring.dashboards.get, monitoring.dashboards.list  `

You might also be able to get these permissions with [custom roles](/iam/docs/creating-custom-roles) or other [predefined roles](/iam/docs/roles-overview#predefined) .

## Customize the system insights dashboard

The system insights dashboard is a predefined dashboard that you can customize to display the information that is most important to you. You can add new charts, change the layout, and filter the data to focus on specific resources.

Changes to the system insights dashboard are non-destructive, and can be reset by setting the *dashboard selector* to **Predefined** .

### Modify the dashboard

To modify the dashboard, click edit **Customize dashboards** . The following options are available to you:

  - **Add a widget:** In the dashboard toolbar, click add **Add widget** , select the widget you want to add, and then configure it.
  - **Edit a widget:** Hover over a widget to show its toolbar, then click edit **Edit** . You can change the widget's type and customize the data it displays.
  - **Clone a widget:** Hover over a widget to show its toolbar, click more\_vert **More chart options** , then **Clone widget** .
  - **Delete a widget:** Hover over a widget to show its toolbar, click more\_vert **More chart options** , then **Delete widget** .
  - **Change the layout:** You can drag widgets to reposition them and drag their corners to resize them.
  - **Name the custom view:** You can set the custom view name in the **Custom view name** box.
  - **Save the dashboard:** You can save the custom view by clicking save **Save** . You can also quit without saving by clicking **Exit edit mode** .

## System insights scorecards, charts, and metrics

The system insights dashboard provides the following charts and metrics to show an instance's current and historical status. Most charts and metrics are available at the instance level. You can also view many charts and metrics for a single database within an instance.

### Available Scorecards

<table>
<thead>
<tr class="header">
<th>Name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>CPU utilization</td>
<td>Total CPU use within an instance or selected database. In a dual-region or multi-region instance, this metric represents the mean of CPU utilization across regions.</td>
</tr>
<tr class="even">
<td>Latency (p99)</td>
<td>P99 latency (99th percentile) for read and write operations within an instance or selected database, representing the time within which 99% of these operations complete.</td>
</tr>
<tr class="odd">
<td>Latency (p50)</td>
<td>P50 latency (50th percentile) for read and write operations within an instance or selected database, representing the time within which 50% of these operations complete.</td>
</tr>
<tr class="even">
<td>Throughput</td>
<td>Amount of uncompressed data that was read from, or written to the instance or database each second. This value is measured in binary bytes, such as KiB, MiB, or GiB.</td>
</tr>
<tr class="odd">
<td>Operations per second</td>
<td>Number of operations per second (rate) of read and writes within an instance or selected database.</td>
</tr>
<tr class="even">
<td>Storage utilization</td>
<td>At the instance level it is the total storage utilization percentage within an instance. At the database level this is the total storage used for the selected database.</td>
</tr>
</tbody>
</table>

### Available charts and metrics

The following is a chart for a sample metric, CPU utilization by operation type:

The toolbar on each chart provides the following standard options. Some elements are hidden unless you hold the pointer over the chart.

  - To zoom into a particular section of a chart, drag your pointer across the section that you want to view. This action sets a custom time range, which you can adjust or revert with the time range filter.

  - To view a description of the chart and its data, click *help* .

  - To view the filters and groupings that are applied to the chart, click *info* .

  - To create an alert based on the chart's data, click *add\_alert* .

  - To explore the data in the chart, click *query\_stats* .

  - To view additional chart options, click *more\_vert* **More chart options** .
    
      - To view a chart in full-screen mode, click **View in full screen** . You can exit full screen by clicking **Cancel** or pressing Esc .
    
      - To expand or collapse the chart legend, click **Expand/Collapse chart legend** .
    
      - To download the chart, click **Download** , and then select a download format.
    
      - To change the visual format of the chart, click **Mode** , and then select a view mode.
    
      - To view the metric in [Metrics Explorer](/monitoring/charts/metrics-explorer) , click **View in Metrics Explorer** . You can view other Spanner metrics in the Metrics Explorer after selecting the **Spanner Database** resource type.

The following table describes the charts that appear by default on the system insights dashboard. The metric type for each chart is listed. The metric type strings follow this prefix: `  spanner.googleapis.com/  ` . [Metric type](/monitoring/api/v3/metric-model#metric_types) describes measurements that can be collected from a monitored resource.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
</colgroup>
<thead>
<tr class="header">
<th>Chart name and metric type<br />
</th>
<th>Description</th>
<th>Available for instances</th>
<th>Available for databases</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><br />
<br />
Dual-region quorum health timeline<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/dual_region_quorum_availability</a></td>
<td><br />
This chart is only shown for <a href="/spanner/docs/instance-configurations#dual-region-configurations">dual-region instance configurations</a> . It shows the health of three quorums: the dual-region quorum ( <code dir="ltr" translate="no">       Global      </code> ), and the single region quorum in each region (for example, <code dir="ltr" translate="no">       Sydney      </code> and <code dir="ltr" translate="no">       Melbourne      </code> ).<br />
<br />
It shows an orange bar in the timeline when there is a service disruption. You can hover over the bar to see the start and end times of the disruption. Use this chart alongside the error rates and latency metrics to help you make self-managed, when-to-failover decisions in the case of regional failures. For more information, see <a href="/spanner/docs/instance-configurations#failover-failback">Failover and failback</a> .<br />
<br />
To failover and failback manually, see <a href="/spanner/docs/change-dual-region-quorum">Change dual-region quorum</a> .</td>
<td><br />
<em>done</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="even">
<td><br />
CPU utilization by priority<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/cpu/utilization_by_priority</a></td>
<td><br />
The percentage of the instance's CPU resources for high, medium, low, or all tasks by priority. These tasks include requests that you initiate and maintenance tasks that Spanner must complete promptly.<br />
<br />
For dual-region or multi-region instances, metrics are grouped by the region and priority.<br />
<br />
<a href="/spanner/docs/cpu-utilization#task-priority">Learn more about high-priority tasks</a> .<br />
<a href="/spanner/docs/cpu-utilization">Learn more about CPU utilization.</a></td>
<td><br />
<em>done</em></td>
<td><br />
<em>close</em></td>
</tr>
<tr class="odd">
<td><br />
<br />
CPU utilization by region<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/cpu/utilization_by_priority</a></td>
<td>Utilization of CPU in the selected instance or database, grouped by region.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="even">
<td><br />
<br />
CPU utilization by database<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/cpu/utilization_by_priority</a></td>
<td>Utilization of CPU in the selected instance, grouped by database and region.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>close</em></td>
</tr>
<tr class="odd">
<td><br />
<br />
CPU utilization by user/system<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/cpu/utilization_by_priority</a></td>
<td>Utilization of CPU in the selected instance or database, grouped by user and system tasks, and by priority.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="even">
<td><br />
CPU utilization by operation type<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/cpu/utilization_by_operation_type</a></td>
<td><br />
A stacked chart of CPU utilization as a percentage of the instance's CPU resources, grouped by user-initiated operations such as reads, writes, and commits. Use this metric to get a detailed breakdown of CPU usage and to troubleshoot further, as explained in <a href="/spanner/docs/introspection/investigate-cpu-utilization">Investigate high CPU utilization</a> .<br />
<br />
You can further filter by priority of the tasks using the option list.<br />
<br />
For dual-region or multi-region instances, metrics in the line chart show the mean percentage among regions.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="odd">
<td><br />
CPU utilization (rolling 24-hour average)<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/cpu/smoothed_utilization</a></td>
<td><br />
A rolling average of total <a href="/spanner/docs/cpu-utilization">CPU Spanner utilization</a> , as a percentage of the instance's CPU resources, for each database. Each data point is an average for the previous 24 hours.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>close</em></td>
</tr>
<tr class="even">
<td><br />
Latency<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">api/request_latencies</a></td>
<td><br />
The amount of time that Spanner took to handle a read or write request. This measurement begins when the Spanner receives a request, and it ends when the Spanner starts to send a response.<br />
<br />
You can view latency metrics for the 50th and 99th percentile latencies by using the option list.</td>
<td><br />
<em>close</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="odd">
<td><br />
Latency by database<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">api/request_latencies</a></td>
<td><br />
The amount of time that Spanner took to handle a read or write request, grouped by database. This measurement begins when Spanner receives a request, and it ends when Spanner starts to send a response.<br />
<br />
You can view metrics for the 50th and 99th percentile latency by using the view list on this chart.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>close</em></td>
</tr>
<tr class="even">
<td><br />
Latency by API method<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">api/request_latencies</a></td>
<td><br />
The amount of time that Spanner took to handle a request, grouped by Spanner API methods. This measurement begins when Spanner receives a request, and it ends when Spanner starts to send a response.<br />
<br />
You can view metrics for the 50th and 99th percentile latencies by using the view list on this chart.</td>
<td><br />
<em>close</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="odd">
<td><br />
<br />
Transaction latency<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">api/request_latencies_by_transaction_type</a></td>
<td><br />
The amount of time that Spanner took to process a transaction. You can select to view metrics for read-write and read-only type transactions.<br />
<br />
The major difference between the Latency chart and the Transaction latency chart is that the Transaction latency chart lets you see the leader involvement for the read-only type. Reads that involve the leader might experience higher latency. You can use this chart to evaluate if you should use stale reads without communicating with the leader, assuming the <a href="/spanner/docs/timestamp-bounds">timestamp bound</a> is at least 15 seconds. For read-write transactions, the leader is always involved in the transaction, so the data shown on the chart always includes the time it took for the request to reach the leader and receive a response. The location corresponds to the region of the <a href="/spanner/docs/latency-points#spanner-api-requests/">Cloud Spanner API frontend</a> .<br />
<br />
You can view metrics for the 50th and 99th percentile latencies by using the view list on this chart.</td>
<td><br />
<em>close</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="even">
<td><br />
Transaction latency by database<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">api/request_latencies_by_transaction_type</a></td>
<td><br />
The amount of time that Spanner took to process a transaction.<br />
<br />
The major difference between the Latency chart and the Transaction latency by database chart is that the Transaction latency by database chart lets you see the leader involvement for the read-only type. Reads that involve the leader might experience higher latency. You can use this chart to evaluate if you should use stale reads without communicating with the leader, assuming the <a href="/spanner/docs/timestamp-bounds">timestamp bound</a> is at least 15 seconds. For read-write transactions, the leader is always involved in the transaction, so the data shown on the chart always includes the time it took for the request to reach the leader and receive a response. The location corresponds to the region of the <a href="/spanner/docs/latency-points#spanner-api-requests/">Cloud Spanner API frontend</a> .<br />
<br />
You can view metrics for the 50th and 99th percentile latencies by using the view list on this chart.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>close</em></td>
</tr>
<tr class="odd">
<td><br />
<br />
Transaction latency by API method<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">api/request_latencies_by_transaction_type</a></td>
<td><br />
The amount of time that Spanner took to process a transaction.<br />
<br />
The major difference between the Latency chart and the Transaction latency by API method chart is that the Transaction latency by API method chart lets you see the leader involvement for the read-only type. Reads which involve the leader might experience higher latency. You can use this chart to evaluate if you should use stale reads without communicating with the leader, assuming the <a href="/spanner/docs/timestamp-bounds">timestamp bound</a> is at least 15 seconds. For read-write transactions, the leader is always involved in the transaction so the data shown on the chart always include the time it took for the request to reach the leader and receive a response. The location corresponds to the region of the <a href="/spanner/docs/latency-points#spanner-api-requests/">Cloud Spanner API frontend</a> .</td>
<td><br />
<em>close</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="even">
<td><br />
Operations per second<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">api/api_request_count</a></td>
<td><br />
The number of read and write operations that Spanner performs per second, or the number of Spanner server errors per second.<br />
<br />
You can choose which operations to view in this chart:<br />

<ul>
<li>Reads and writes (also includes read and write errors)</li>
</ul>
<ul>
<li>Errors on the Spanner server (grouped by read and write)</li>
</ul></td>
<td><br />
<em>close</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="odd">
<td><br />
Operations per second by database<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">api/api_request_count</a></td>
<td><br />
The number of read and write operations that Spanner performs per second, or the number of Spanner server errors per second. This chart is grouped by database.<br />
<br />
You can choose which operations to view in this chart:<br />

<ul>
<li>Reads and writes (also includes read and write errors)</li>
</ul>
<ul>
<li>Errors on the Spanner server (grouped by read and write)</li>
</ul></td>
<td><br />
<em>done</em></td>
<td><br />
<em>close</em></td>
</tr>
<tr class="even">
<td><br />
Operations per second by API method<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">api/api_request_count</a></td>
<td><br />
The number of operations that Spanner performed per second, grouped by Spanner API method</td>
<td><br />
<em>close</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="odd">
<td><br />
Throughput<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">api/sent_bytes_count</a> (read)<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">api/received_bytes_count</a> (write)</td>
<td><br />
The amount of uncompressed data read from and written to the database each second. This value is measured in binary bytes, such as KiB, MiB, or GiB.<br />
<br />
Read throughput includes requests and responses for methods in the <a href="/spanner/docs/reads">read API</a> and for SQL queries. It also includes requests and responses for DML statements.<br />
<br />
Write throughput includes requests and responses to commit data through the <a href="/spanner/docs/modify-mutation-api">mutation API</a> . It excludes requests and responses for DML statements.</td>
<td><br />
<em>close</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="even">
<td><br />
Throughput by database<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">api/sent_bytes_count</a> (read)<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">api/received_bytes_count</a> (write)</td>
<td><br />
The amount of uncompressed data read from and written to the instance each second, grouped by database. This value is measured in binary bytes, such as KiB, MiB, or GiB.<br />
<br />
Read throughput includes requests and responses for methods in the <a href="/spanner/docs/reads">read API</a> and for SQL queries. It also includes requests and responses for DML statements.<br />
<br />
Write throughput includes requests and responses to commit data through the <a href="/spanner/docs/modify-mutation-api">mutation API</a> . It excludes requests and responses for DML statements.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>close</em></td>
</tr>
<tr class="odd">
<td><br />
Throughput by API method<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">api/sent_bytes_count</a> (read)<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">api/received_bytes_count</a> (write)</td>
<td><br />
The amount of uncompressed data that was read from, or written to, the instance or database each second, grouped by API method. This value is measured in binary bytes, such as KiB, MiB, or GiB.<br />
<br />
Read throughput includes requests and responses for methods in the <a href="/spanner/docs/reads">read API</a> and for SQL queries. It also includes requests and responses for DML statements.<br />
<br />
Write throughput includes requests and responses to commit data through the <a href="/spanner/docs/modify-mutation-api">mutation API</a> . It excludes requests and responses for DML statements.</td>
<td><br />
<em>close</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="even">
<td><br />
Total storage<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/storage/used_bytes</a></td>
<td><br />
The amount of data that is stored in the database. This value is measured in binary bytes, such as KiB, MiB, or GiB.</td>
<td><br />
<em>close</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="odd">
<td><br />
Total database storage by database<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/storage/used_bytes</a></td>
<td><br />
The amount of data that is stored in the instance, grouped by database. This value is measured in binary bytes, such as KiB, MiB, or GiB.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>close</em></td>
</tr>
<tr class="even">
<td><br />
Total backup storage<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/backup/used_bytes</a></td>
<td><br />
The amount of data that is stored in the backups that are associated with the database. This value is measured in binary bytes, such as KiB, MiB, or GiB.</td>
<td><br />
<em>close</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="odd">
<td><br />
Lock wait time<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">lock_stat/total/lock_wait_time</a></td>
<td><br />
Lock wait time for a transaction is the time needed to acquire a lock on a resource held by another transaction.<br />
<br />
Total lock wait time for <a href="/spanner/docs/introspection/lock-statistics">lock</a> conflicts is recorded for the entire database.</td>
<td><br />
<em>close</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="even">
<td><br />
Lock wait time by database<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">lock_stat/total/lock_wait_time</a></td>
<td><br />
Lock wait time for a transaction is the time needed to acquire a lock on a resource held by another transaction, grouped by database.<br />
<br />
Total lock wait time for <a href="/spanner/docs/introspection/lock-statistics">lock</a> conflicts is recorded for the entire instance.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>close</em></td>
</tr>
<tr class="odd">
<td><br />
Total backup storage by database<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/backup/used_bytes</a></td>
<td><br />
The amount of data that is stored in the backups that are associated with the instance, grouped by database. This value is measured in binary bytes, such as KiB, MiB, or GiB.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>close</em></td>
</tr>
<tr class="even">
<td><br />
Compute capacity<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/processing_units</a><br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/nodes</a></td>
<td><br />
The <a href="/spanner/docs/compute-capacity">compute capacity</a> is the amount of processing units or nodes available in an instance. You can choose to display the capacity in processing units or in nodes.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>close</em></td>
</tr>
<tr class="odd">
<td><br />
<br />
Leader distribution<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/leader_percentage_by_region</a></td>
<td><br />
For dual-region or multi-region instances, you can view the number of databases with the majority of leaders (&gt;=50%) in a given region. Under the <strong>Regions</strong> list menu, if you select a specific region, the chart shows the total number of databases within that instance that have the selected region as the leader region. If you select <strong>All regions</strong> under the <strong>Regions</strong> list menu, the chart shows one line for each region, and each line shows the total number of databases in the instance that has that region as its leader region.<br />
<br />
For databases in a dual-region or multi-region instance, you can view the percentage of leaders grouped by region. For example, if a database has five leaders, one in <code dir="ltr" translate="no">       us-west1      </code> and four in <code dir="ltr" translate="no">       us-east1      </code> at a point-in-time, the "All regions" chart shows two lines (one per region). One line for <code dir="ltr" translate="no">       us-west1      </code> is at 20%, and the other line for <code dir="ltr" translate="no">       us-east1      </code> is at 80%. The us-west1 chart shows one single line at 20%, and the us-east1 chart shows one single line at 80%.<br />
<br />
Note that if a database was recently created or a leader region was recently modified, the charts might not stabilize right away.<br />
<br />
This chart is only available for dual-region and multi-region instances.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="even">
<td><br />
Peak split CPU usage score<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/peak_split_peak</a></td>
<td>The maximum peak split CPU usage observed across all splits in a database. This metric shows the percentage of the processing unit resources that are being used on a split. A percentage of over 50% is a warm split, which means that the split is using half of the host server's processing unit resources. A percentage of 100% is a hot split, which is a split that's using the majority of the host server's processing unit resources. Spanner uses load-based splitting to resolve hotspots and balance the load. However, Spanner might not be able to balance the load, even after multiple attempts at splitting, due to problematic patterns in the application. Hence, hotspots that lasts for at least 10 minutes might need further troubleshooting and could potentially require application changes. For more information, see <a href="/spanner/docs/introspection/hot-split-statistics">Find hotspots in splits</a> .</td>
<td><br />
<em>done</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="odd">
<td><br />
<br />
Remote service calls<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">query_stat/total/remote_service_calls_count</a></td>
<td><br />
Count of remote service calls, grouped by the service and response codes.<br />
<br />
Responds with an HTTP response code, such as 200 or 500.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="even">
<td><br />
<br />
Latency: Remote service calls<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">query_stat/total/remote_service_calls_latencies</a></td>
<td><br />
The latency of the remote service calls, grouped by service.<br />
<br />
You can view latency metrics for the 50th and 99th percentile latencies by using the option list.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="odd">
<td><br />
<br />
Remote service processed rows<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">query_stat/total/remote_service_processed_rows_count</a></td>
<td><br />
Count of rows processed by a remote service, grouped by the servicer and response codes.<br />
<br />
Responds with an HTTP response code, such as 200 or 500.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="even">
<td><br />
<br />
Latency: Remote service rows<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">query_stat/total/remote_service_processed_rows_latencies</a></td>
<td><br />
Count of rows processed by a remote service, grouped by the service and response codes.<br />
<br />
You can view latency metrics for the 50th and 99th percentile latencies by using the option list.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="odd">
<td><br />
<br />
Remote service network bytes<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">query_stat/total/remote_service_network_bytes_sizes</a></td>
<td><br />
Network bytes exchanged with the remote service, grouped by service and direction.<br />
<br />
This value is measured in binary bytes, such as KiB, MiB, or GiB.<br />
<br />
Direction refers to traffic being sent or received.<br />
<br />
You can view metrics for the 50th and 99th percentile of network bytes exchange by using the option list.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="even">
<td><br />
<br />
Micro service calls<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">query_stat/total/remote_service_calls_count</a></td>
<td>Number of micro service calls, grouped by micro service and response code.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="odd">
<td><br />
<br />
Latency: Micro service calls<br />
<br />
<br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">query_stat/total/remote_service_calls_latencies</a></td>
<td>Latencies of micro service calls, grouped by micro service.</td>
<td><br />
<em>done</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="even">
<td><br />
Database storage by table<br />
<br />
<br />
(none)</td>
<td><br />
The amount of data that is stored in the instance or database, grouped by tables in the selected database. This value is measured in binary bytes, such as KiB, MiB, or GiB.<br />
<br />
This chart obtains its data by querying <code dir="ltr" translate="no">       SPANNER_SYS.TABLE_SIZES_STATS_1HOUR      </code> . For more information, see <a href="/spanner/docs/introspection/table-sizes-statistics">Table sizes statistics</a> .</td>
<td><br />
<em>close</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="odd">
<td><br />
Most-used tables by operations<br />
<br />
<br />
(none)</td>
<td><br />
The 15 most used tables and indexes in the instance or database, determined by the number of read or write or delete operations.<br />
This chart obtains its data by querying the table operations statistics tables. For more information, see <a href="/spanner/docs/introspection/table-operations-statistics">Table operations statistics</a> .</td>
<td><br />
<em>close</em></td>
<td><br />
<em>done</em></td>
</tr>
<tr class="even">
<td><br />
Least-used tables by operations<br />
<br />
<br />
(none)</td>
<td><br />
The 15 least used tables and indexes in the instance or database, determined by the number of read or write or delete operations.<br />
This chart obtains its data by querying the table operations statistics tables. For more information, see <a href="/spanner/docs/introspection/table-operations-statistics">Table operations statistics</a> .</td>
<td><br />
<em>close</em></td>
<td><br />
<em>done</em></td>
</tr>
</tbody>
</table>

#### Managed autoscaler charts and metrics

In addition to the options shown in the previous section, when an instance has managed autoscaler enabled, the compute capacity chart has the **View Logs** button. When you click this button, it displays logs from the managed autoscaler.

The following metrics are available for instances that have the managed autoscaler enabled.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Metric name and type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Compute capacity</strong></td>
<td>With nodes selected.</td>
</tr>
<tr class="even">
<td><br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/autoscaling/min_node_count</a></td>
<td><br />
Minimum number of nodes autoscaler is configured to allocate to the instance.</td>
</tr>
<tr class="odd">
<td><br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/autoscaling/max_node_count</a></td>
<td>Maximum number of nodes autoscaler is configured to allocate to the instance.</td>
</tr>
<tr class="even">
<td><br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/autoscaling/recommended_node_count_for_cpu</a></td>
<td><br />
Recommended number of nodes based on the CPU usage of the instance.</td>
</tr>
<tr class="odd">
<td><br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/autoscaling/recommended_node_count_for_storage</a></td>
<td><br />
Recommended number of nodes based on the storage usage of the instance.</td>
</tr>
<tr class="even">
<td><strong>Compute capacity</strong></td>
<td>With processing units selected.</td>
</tr>
<tr class="odd">
<td><br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/autoscaling/min_processing_units</a></td>
<td><br />
Minimum number of processing units autoscaler is configured to allocate to the instance.</td>
</tr>
<tr class="even">
<td><br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/autoscaling/max_processing_units</a></td>
<td><br />
Maximum number of processing units autoscaler is configured to allocate to the instance.</td>
</tr>
<tr class="odd">
<td><br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/autoscaling/recommended_processing_units_for_cpu</a></td>
<td><br />
Recommended number of processing units. This recommendation is based on the previous CPU usage of the instance.</td>
</tr>
<tr class="even">
<td><br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/autoscaling/recommended_processing_units_for_storage</a></td>
<td><br />
Recommended number of processing units to use. This recommendation is based on the previous storage usage of the instance.</td>
</tr>
<tr class="odd">
<td><strong>CPU utilization by priority</strong></td>
<td></td>
</tr>
<tr class="even">
<td><br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/autoscaling/high_priority_cpu_utilization_target</a></td>
<td><br />
High priority CPU utilization target to use for autoscaling.</td>
</tr>
<tr class="odd">
<td><strong>Total storage</strong></td>
<td>With processing units selected.</td>
</tr>
<tr class="even">
<td><br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/storage/limit_bytes</a></td>
<td><br />
Storage limit for the instance in bytes.</td>
</tr>
<tr class="odd">
<td><br />
<a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/autoscaling/storage_utilization_target</a></td>
<td><br />
Storage utilization target to use for autoscaling.</td>
</tr>
</tbody>
</table>

#### Tiered storage charts and metrics

The following metrics are available for instances that use [tiered storage](/spanner/docs/tiered-storage) .

<table>
<thead>
<tr class="header">
<th>Metric name and type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/storage/used_bytes</a></td>
<td>Total bytes of data stored on SSD and HDD storage.</td>
</tr>
<tr class="even">
<td><a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/storage/combined/limit_bytes</a></td>
<td>Combined SSD and HDD storage limits.</td>
</tr>
<tr class="odd">
<td><a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/storage/combined/limit_per_processing_unit</a></td>
<td>Combined SSD and HDD storage limit for each processing unit.</td>
</tr>
<tr class="even">
<td><a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/storage/combined/utilization</a></td>
<td>Combined SSD and HDD storage used, compared to the combined storage limit.</td>
</tr>
<tr class="odd">
<td><a href="/monitoring/api/metrics_gcp_p_z#gcp-spanner">instance/disk_load</a></td>
<td>HDD load use.</td>
</tr>
</tbody>
</table>

## Data retention

The maximum data retention for most metrics on the system insights dashboard is 6 weeks. However, for the **Database storage by table** chart, the data is consumed from the `  SPANNER_SYS.TABLE_SIZES_STATS_1HOUR  ` table (instead of Spanner), which has a maximum retention of 30 days. See [Data retention](/spanner/docs/introspection/query-statistics#data_retention) to learn more.

## View the system insights dashboard

To view the system insights page, you need the following Identity and Access Management (IAM) permissions in addition to the [Spanner permissions](/monitoring/access-control) and Spanner permissions at the instance and database levels:

  - `  spanner.databases.beginReadOnlyTransaction  `
  - `  spanner.databases.select  `
  - `  spanner.sessions.create  `

For more information about Spanner IAM permissions, see [Access control with IAM](/spanner/docs/iam) .

If you enable [managed autoscaler](/spanner/docs/managed-autoscaler) on your instance, you also need `  logging.logEntries.list  ` , `  logging.logs.list  ` , and `  logging.logServices.list  ` permissions to view managed autoscaler logs.

For more information about this permission, see [Predefined roles](/logging/docs/access-control#permissions_and_roles) .

To view the system insights dashboard, follow these steps:

1.  In the Google Cloud console, open the list of Spanner instances.

2.  Do one of the following:
    
    1.  To see metrics for an instance, click the name of the instance that you want to learn about, then click **System insights** in the navigation menu.
    
    2.  To see metrics for a database, click the name of the instance, select a database, then click **System insights** in the navigation menu.

3.  Optional: To view historical data for a different time period, find the buttons at the top right of the page, then click the time period that you want to view.

4.  Optional: To control what data appears in the chart, click one of the lists in the chart. For example, if the instance uses a [dual-region or multi-region configuration](/spanner/docs/instances#configuration) , some charts provide a list to view data for a specific region. Not all charts have view lists.

## What's next

  - Understand the [CPU utilization](/spanner/docs/cpu-utilization) and [latency](/spanner/docs/latency-guide) metrics for Spanner.
  - [Set up customized charts and alerts](/spanner/docs/monitoring-cloud) with Monitoring.
  - Get details about [types of Spanner instances](/spanner/docs/instances) .
