This document describes how to use the Cloud Monitoring console to monitor your Spanner instances.

The Cloud Monitoring console provides several monitoring tools for Spanner:

  - A *curated dashboard* , which shows pre-made charts for your Spanner resources
  - *Custom charts* , including [ad-hoc charts in the Metrics Explorer](/monitoring/charts/metrics-explorer) as well as [charts in custom dashboards](/monitoring/charts)
  - *Alerts* , which notify you if a metric exceeds a threshold that you specify

If you prefer to monitor Spanner programmatically, use the [Cloud Client Libraries for Cloud Monitoring](/monitoring/docs/reference/libraries) to retrieve metrics.

**Note:** You can also monitor your instances by [viewing charts in the Google Cloud console](/spanner/docs/monitoring-console) . Use the Google Cloud console to get a quick view of the most important metrics for your instance.

## Use the Cloud Monitoring curated dashboard

Cloud Monitoring provides you with a curated dashboard that summarizes key information about your Spanner instances, including:

  - *Incidents* : User-created monitoring alerts that are open, active, or resolved
  - *Events* : A list of Spanner [audit logs](/spanner/docs/logs) (if enabled and available)
  - *Instances* : A high-level summary of your Spanner instances, including [compute capacity](/spanner/docs/compute-capacity) , database count, and instance health
  - *Aggregated charts* of throughput and storage use

To view the Spanner dashboard, do the following:

1.  In the Google Cloud console, select **Monitoring** , or use the following button:

2.  If **Resources** is shown in the navigation pane, then select **Resources** and then select **Cloud Spanner** . Otherwise, select **Dashboards** and then select the dashboard named **Cloud Spanner** .

### View instance and database details

When you open the curated dashboard for Spanner, it shows aggregated data for all of your instances. You can view more details about a specific instance by clicking the instance's name under **Instances** .

The dashboard displays information such as instance metadata, databases in the instance, and charts of various metrics broken down by region.

From the instance dashboard page, you can also see charts for a specific database in the instance:

1.  On the right-hand side, above the instance metrics charts, click **Database metrics** .

2.  In the **Select a breakdown** drop-down list, select the database that you want to examine.
    
    The Cloud Monitoring console displays charts for the database.

## Create custom charts for Spanner metrics

You can use Cloud Monitoring to create custom charts for Spanner metrics. You can use the Metrics Explorer to create temporary, ad-hoc charts, or you can create charts that appear on custom dashboards.

In particular, Cloud Monitoring lets you create a custom chart that shows whether two or more metrics are correlated with each other. For example, you can check for a correlation between [CPU utilization](/spanner/docs/cpu-utilization) and [latency](/spanner/docs/latency-guide) in a Spanner instance, which might indicate that your instance needs more [compute capacity](/spanner/docs/compute-capacity) or that some of your queries are causing high CPU utilization.

To get started with this example, follow these steps:

1.  In the Google Cloud console, select **Monitoring** , or use the following button:

2.  If **Metrics Explorer** is shown in the navigation pane, select it. Otherwise, select **Resources** and then select **Metrics Explorer** .

3.  Click the **View options** tab, then select the **Log scale on Y-axis** checkbox. This option helps you compare multiple metrics when one metric has much larger values than the others.

4.  In the drop-down list above the right pane, select **Line** .

5.  Click the **Metrics** tab. You can now add metrics to the chart.

To add latency metrics to the chart, follow these steps:

1.  In the **Find resource type and metric** box, enter the value `  spanner.googleapis.com/api/request_latencies  ` , then click the row that appears below the box.

2.  In the **Filter** box, enter the value `  instance_id  ` , then enter the instance ID you want to examine and click **Apply** .

3.  In the **Aggregator** drop-down list, click **max** .

4.  Optional: Change the latency percentile:
    
    1.  Click **Show advanced options** .
    2.  Click the **Aligner** drop-down list, then click the latency percentile that you want to view.
    
    In most cases, you should look at either the 50th percentile latency, to understand the typical amount of latency, or the 99th percentile latency, to understand the latency for the slowest 1% of requests.

To add CPU utilization metrics to the chart, follow these steps:

1.  Click **add Add metric** .
2.  In the **Find resource type and metric** box, enter the value `  spanner.googleapis.com/instance/cpu/utilization  ` , then click the row that appears below the box.
3.  In the **Filter** box, enter the value `  instance_id  ` , then enter the instance ID you want to examine and click **Apply** .
4.  In the **Aggregator** drop-down list, click **max** .

You now have a chart that shows the CPU utilization and latency metrics for a Spanner instance. If both metrics are higher than expected at the same time, you can [take additional steps to correct the issue](/spanner/docs/latency-metrics) .

For more information about creating custom charts, see the [Cloud Monitoring documentation](/monitoring/charts) .

## Create alerts for Spanner metrics

When you create a Spanner [instance](/spanner/docs/instances) , you choose the [compute capacity](/spanner/docs/compute-capacity) for the instance. As the instance's workload changes, Spanner does not automatically adjust compute capacity of the instance. As a result, you need to set up several alerts to ensure that the instance stays within the [recommended maximums for CPU utilization](/spanner/docs/cpu-utilization#recommended-max) and the [recommended limit for storage](/spanner/docs/limits) .

The following examples show how to set up alerting policies for some Spanner metrics. For a full list of available metrics, see [metrics list for Spanner](/monitoring/api/metrics_gcp_p_z#gcp-spanner) .

### High-priority CPU

To create an alerting policy that triggers when your high priority cpu utilization for [Spanner](/spanner/docs) is above a recommended threshold, use the following settings.

#### Steps to create an [alerting policy](/monitoring/alerts/using-alerting-ui#create-policy) .

To create an alerting policy, do the following:

1.  In the Google Cloud console, go to the *notifications* **Alerting** page:
    
    If you use the search bar to find this page, then select the result whose subheading is **Monitoring** .

2.  If you haven't created your notification channels and if you want to be notified, then click **Edit Notification Channels** and add your notification channels. Return to the **Alerting** page after you add your channels.

3.  From the **Alerting** page, select **Create policy** .

4.  To select the resource, metric, and filters, expand the **Select a metric** menu and then use the values in the **New condition** table:
    
    1.  Optional: To limit the menu to relevant entries, enter the resource or metric name in the filter bar.
    2.  Select a **Resource type** . For example, select **VM instance** .
    3.  Select a **Metric category** . For example, select **instance** .
    4.  Select a **Metric** . For example, select **CPU Utilization** .
    5.  Select **Apply** .

5.  Click **Next** and then configure the alerting policy trigger. To complete these fields, use the values in the **Configure alert trigger** table.

6.  Click **Next** .

7.  Optional: To add notifications to your alerting policy, click **Notification channels** . In the dialog, select one or more notification channels from the menu, and then click **OK** .
    
    To be notified when incidents are openend and closed, check **Notify on incident closure** . By default, notifications are sent only when incidents are openend.

8.  Optional: Update the **Incident autoclose duration** . This field determines when Monitoring closes incidents in the absence of metric data.

9.  Optional: Click **Documentation** , and then add any information that you want included in a notification message.

10. Click **Alert name** and enter a name for the alerting policy.

11. Click **Create Policy** .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>New condition</strong><br />
Field</th>
<th><br />
Value</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Resource and Metric</strong></td>
<td>In the <strong>Resources</strong> menu, select <strong>Spanner Instance</strong> .<br />
In the <strong>Metric categories</strong> menu, select <strong>Instance</strong> .<br />
In the <strong>Metrics</strong> menu, select <strong>CPU Utilization by priority</strong> .<br />
<br />
(The metric.type is <code dir="ltr" translate="no">         spanner.googleapis.com/instance/cpu/utilization_by_priority        </code> ).</td>
</tr>
<tr class="even">
<td><strong>Filter</strong></td>
<td><code dir="ltr" translate="no">         instance_id =                   YOUR_INSTANCE_ID         </code><br />
<code dir="ltr" translate="no">         priority = high        </code></td>
</tr>
<tr class="odd">
<td><strong>Across time series<br />
Time series group by</strong></td>
<td><code dir="ltr" translate="no">         location        </code> for multi-region instances;<br />
leave it blank for regional instances.</td>
</tr>
<tr class="even">
<td><strong>Across time series<br />
Time series aggregation</strong></td>
<td><code dir="ltr" translate="no">         sum        </code><br />
</td>
</tr>
<tr class="odd">
<td><strong>Rolling window</strong></td>
<td><code dir="ltr" translate="no">         10 m        </code><br />
</td>
</tr>
<tr class="even">
<td><strong>Rolling window function</strong></td>
<td><code dir="ltr" translate="no">         mean        </code></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Configure alert trigger</strong><br />
Field</th>
<th><br />
Value</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Condition type</strong></td>
<td><code dir="ltr" translate="no">         Threshold        </code></td>
</tr>
<tr class="even">
<td><strong>Alert trigger</strong></td>
<td><code dir="ltr" translate="no">         Any time series violates        </code></td>
</tr>
<tr class="odd">
<td><strong>Threshold position</strong></td>
<td><code dir="ltr" translate="no">         Above threshold        </code></td>
</tr>
<tr class="even">
<td><strong>Threshold value</strong></td>
<td><code dir="ltr" translate="no">         45%        </code> for multi-region instances;<br />
<code dir="ltr" translate="no">         65%        </code> for regional instances.</td>
</tr>
<tr class="odd">
<td><strong>Retest window</strong></td>
<td><code dir="ltr" translate="no">         10 minutes        </code></td>
</tr>
</tbody>
</table>

### 24 hour rolling average CPU

To create an alerting policy that triggers when the 24 hour rolling average of your cpu utilization for [Spanner](/spanner/docs) is above a recommended threshold, use the following settings.

#### Steps to create an [alerting policy](/monitoring/alerts/using-alerting-ui#create-policy) .

To create an alerting policy, do the following:

1.  In the Google Cloud console, go to the *notifications* **Alerting** page:
    
    If you use the search bar to find this page, then select the result whose subheading is **Monitoring** .

2.  If you haven't created your notification channels and if you want to be notified, then click **Edit Notification Channels** and add your notification channels. Return to the **Alerting** page after you add your channels.

3.  From the **Alerting** page, select **Create policy** .

4.  To select the resource, metric, and filters, expand the **Select a metric** menu and then use the values in the **New condition** table:
    
    1.  Optional: To limit the menu to relevant entries, enter the resource or metric name in the filter bar.
    2.  Select a **Resource type** . For example, select **VM instance** .
    3.  Select a **Metric category** . For example, select **instance** .
    4.  Select a **Metric** . For example, select **CPU Utilization** .
    5.  Select **Apply** .

5.  Click **Next** and then configure the alerting policy trigger. To complete these fields, use the values in the **Configure alert trigger** table.

6.  Click **Next** .

7.  Optional: To add notifications to your alerting policy, click **Notification channels** . In the dialog, select one or more notification channels from the menu, and then click **OK** .
    
    To be notified when incidents are openend and closed, check **Notify on incident closure** . By default, notifications are sent only when incidents are openend.

8.  Optional: Update the **Incident autoclose duration** . This field determines when Monitoring closes incidents in the absence of metric data.

9.  Optional: Click **Documentation** , and then add any information that you want included in a notification message.

10. Click **Alert name** and enter a name for the alerting policy.

11. Click **Create Policy** .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>New condition</strong><br />
Field</th>
<th><br />
Value</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Resource and Metric</strong></td>
<td>In the <strong>Resources</strong> menu, select <strong>Spanner Instance</strong> .<br />
In the <strong>Metric categories</strong> menu, select <strong>Instance</strong> .<br />
In the <strong>Metrics</strong> menu, select <strong>Smoothed CPU utilization</strong> .<br />
<br />
(The metric.type is <code dir="ltr" translate="no">         spanner.googleapis.com/instance/cpu/smoothed_utilization        </code> ).</td>
</tr>
<tr class="even">
<td><strong>Filter</strong></td>
<td><code dir="ltr" translate="no">         instance_id =                   YOUR_INSTANCE_ID         </code></td>
</tr>
<tr class="odd">
<td><strong>Across time series<br />
Time series aggregation</strong></td>
<td><code dir="ltr" translate="no">         sum        </code><br />
</td>
</tr>
<tr class="even">
<td><strong>Rolling window</strong></td>
<td><code dir="ltr" translate="no">         10 m        </code><br />
</td>
</tr>
<tr class="odd">
<td><strong>Rolling window function</strong></td>
<td><code dir="ltr" translate="no">         mean        </code></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Configure alert trigger</strong><br />
Field</th>
<th><br />
Value</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Condition type</strong></td>
<td><code dir="ltr" translate="no">         Threshold        </code></td>
</tr>
<tr class="even">
<td><strong>Alert trigger</strong></td>
<td><code dir="ltr" translate="no">         Any time series violates        </code></td>
</tr>
<tr class="odd">
<td><strong>Threshold position</strong></td>
<td><code dir="ltr" translate="no">         Above threshold        </code></td>
</tr>
<tr class="even">
<td><strong>Threshold</strong></td>
<td><code dir="ltr" translate="no">         90%        </code></td>
</tr>
<tr class="odd">
<td><strong>Retest window</strong></td>
<td><code dir="ltr" translate="no">         10 minutes        </code></td>
</tr>
</tbody>
</table>

### Storage

To create an alerting policy that triggers when your storage for your [Spanner](/spanner/docs) instance is above a recommended threshold, use the following settings.

#### Steps to create an [alerting policy](/monitoring/alerts/using-alerting-ui#create-policy) .

To create an alerting policy, do the following:

1.  In the Google Cloud console, go to the *notifications* **Alerting** page:
    
    If you use the search bar to find this page, then select the result whose subheading is **Monitoring** .

2.  If you haven't created your notification channels and if you want to be notified, then click **Edit Notification Channels** and add your notification channels. Return to the **Alerting** page after you add your channels.

3.  From the **Alerting** page, select **Create policy** .

4.  To select the resource, metric, and filters, expand the **Select a metric** menu and then use the values in the **New condition** table:
    
    1.  Optional: To limit the menu to relevant entries, enter the resource or metric name in the filter bar.
    2.  Select a **Resource type** . For example, select **VM instance** .
    3.  Select a **Metric category** . For example, select **instance** .
    4.  Select a **Metric** . For example, select **CPU Utilization** .
    5.  Select **Apply** .

5.  Click **Next** and then configure the alerting policy trigger. To complete these fields, use the values in the **Configure alert trigger** table.

6.  Click **Next** .

7.  Optional: To add notifications to your alerting policy, click **Notification channels** . In the dialog, select one or more notification channels from the menu, and then click **OK** .
    
    To be notified when incidents are openend and closed, check **Notify on incident closure** . By default, notifications are sent only when incidents are openend.

8.  Optional: Update the **Incident autoclose duration** . This field determines when Monitoring closes incidents in the absence of metric data.

9.  Optional: Click **Documentation** , and then add any information that you want included in a notification message.

10. Click **Alert name** and enter a name for the alerting policy.

11. Click **Create Policy** .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>New condition</strong><br />
Field</th>
<th><br />
Value</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Resource and Metric</strong></td>
<td>In the <strong>Resources</strong> menu, select <strong>Spanner Instance</strong> .<br />
In the <strong>Metric categories</strong> menu, select <strong>Instance</strong> .<br />
In the <strong>Metrics</strong> menu, select <strong>Storage used</strong> .<br />
<br />
(The metric.type is <code dir="ltr" translate="no">         spanner.googleapis.com/instance/storage/utilization        </code> ).</td>
</tr>
<tr class="even">
<td><strong>Filter</strong></td>
<td><code dir="ltr" translate="no">         instance_id =                   YOUR_INSTANCE_ID         </code></td>
</tr>
<tr class="odd">
<td><strong>Across time series<br />
Time series aggregation</strong></td>
<td><code dir="ltr" translate="no">         sum        </code><br />
</td>
</tr>
<tr class="even">
<td><strong>Rolling window</strong></td>
<td><code dir="ltr" translate="no">         10 m        </code><br />
</td>
</tr>
<tr class="odd">
<td><strong>Rolling window function</strong></td>
<td><code dir="ltr" translate="no">         max        </code></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Configure alert trigger</strong><br />
Field</th>
<th><br />
Value</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Condition type</strong></td>
<td><code dir="ltr" translate="no">         Threshold        </code></td>
</tr>
<tr class="even">
<td><strong>Condition triggers if</strong></td>
<td><code dir="ltr" translate="no">         Any time series violates        </code></td>
</tr>
<tr class="odd">
<td><strong>Threshold position</strong></td>
<td><code dir="ltr" translate="no">         Above threshold        </code></td>
</tr>
<tr class="even">
<td><strong>Threshold value</strong></td>
<td>You don't need to set a specific threshold for the maximum storage per node. However, we recommended that you set up an alert for when you are approaching the maximum storage limit. To learn more, see <a href="/spanner/docs/storage-utilization#recommended-max">Storage utilization metrics</a> .</td>
</tr>
<tr class="odd">
<td><strong>Retest window</strong></td>
<td><code dir="ltr" translate="no">         10 minutes        </code></td>
</tr>
</tbody>
</table>

**Note:** Spanner usage amounts are calculated in *binary terabytes* , where 1 TB is 2 <sup>40</sup> bytes. This unit of measurement is also known as a [tebibyte (TiB)](https://en.wikipedia.org/wiki/Tebibyte) .

## What's next

  - Understand the [CPU utilization](/spanner/docs/cpu-utilization) and [latency](/spanner/docs/latency-guide) metrics for Spanner.
  - [Use the Google Cloud console](/spanner/docs/monitoring-console) to get a quick view of the most important metrics for your instance.
  - Learn more about [Cloud Monitoring](/monitoring/docs) .
