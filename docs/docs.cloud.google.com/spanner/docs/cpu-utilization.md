This page describes the CPU utilization metrics that Spanner provides. You can view these metrics [in the Google Cloud console](/spanner/docs/monitoring-console) and [in the Cloud Monitoring console](/spanner/docs/monitoring-cloud) .

## CPU utilization and task priority

Spanner measures CPU utilization based on the *source* and the *priority* of the task.

  - **Source** : A task can either be initiated by the *user* or the *system* .

  - **Priority** : The [priority](/spanner/docs/reference/rest/v1/RequestOptions#Priority) helps Spanner determine which tasks should execute first. The priority of *system* tasks is predetermined and cannot be configured. *User* tasks run at high priority unless otherwise specified. Many data requests, such as [read](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/read) and [executeSql](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql) , let you specify a lower priority for the request. This can be useful, for example, when you are running batch, maintenance, or analytical queries that don't have strict performance SLOs.
    
    Higher-priority tasks are in general going to be executed ahead of lower-priority tasks. Spanner allows high-priority tasks to utilize up to 100% of the available CPU resources even if there are competing lower-priority tasks. While lower-priority system tasks can be delayed in the short term, they must run eventually. Therefore, you must [provision your instance with enough compute capacity](#reduce) to handle all tasks.
    
    If there are no high-priority tasks, Spanner will utilize up to 100% of the available CPU resources to complete lower-priority tasks more quickly. Spikes in background usage are not a sign of a problem. Lower-priority tasks can yield to higher-priority tasks, including user tasks, almost instantly.

The following table shows examples for each task:

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td>User tasks</td>
<td>System tasks</td>
</tr>
<tr class="even">
<td>High priority</td>
<td>Includes data requests, such as <a href="/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/read">read</a> or <a href="/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql">executeSql</a> , where either no priority or <a href="/spanner/docs/reference/rest/v1/RequestOptions#Priority">PRIORITY_HIGH</a> is specified.</td>
<td>Includes data splitting.</td>
</tr>
<tr class="odd">
<td>Medium priority</td>
<td>Includes:
<ul>
<li>Data requests where <a href="/spanner/docs/reference/rest/v1/RequestOptions#Priority">PRIORITY_MEDIUM</a> is specified</li>
<li>Reads and writes issued from Dataflow jobs, including <a href="/spanner/docs/import">Import</a> and <a href="/spanner/docs/export">Export</a> .</li>
</ul></td>
<td>Includes:
<ul>
<li>Database compaction</li>
<li>Schema change validation</li>
<li>The optimization phase of <a href="/spanner/docs/backup#restore">database restore</a></li>
</ul></td>
</tr>
<tr class="even">
<td>Low priority</td>
<td>Includes data requests where <a href="/spanner/docs/reference/rest/v1/RequestOptions#Priority">PRIORITY_LOW</a> is specified.</td>
<td>Includes:
<ul>
<li>Backfilling an index.</li>
<li>Backfilling a generated column</li>
</ul></td>
</tr>
</tbody>
</table>

**Note:** Backups are not listed in this table because Spanner creates dedicated backup jobs to take backups instead of using instance CPU. For more information, see [Backup time and performance](/spanner/docs/backup/create-backup#time) .

## Available metrics

Spanner provides the following metrics for CPU utilization:

  - **Smoothed CPU utilization** : A rolling average of total CPU utilization, as a percentage of the instance's CPU resources, for each database. Each data point is an average for the previous 24 hours. Use this metric to create alerts and analyze CPU usage over long period of time, for example, 24 hours. You can view a chart for this metric [in the Google Cloud console](/spanner/docs/monitoring-console) or [in the Cloud Monitoring console](/spanner/docs/monitoring-cloud) as **Rolling average 24 hour** .

  - **CPU Utilization by priority** : The CPU utilization, as a percentage of the instance's CPU resources, grouped by priority, user-initiated tasks and system-initiated tasks. Use this metric to create alerts and analyze CPU usage at a high level. You can view a chart for this metric [in the Google Cloud console](/spanner/docs/monitoring-console) or [in the Cloud Monitoring console](/spanner/docs/monitoring-cloud) .

  - **CPU Utilization by operation type** : The CPU utilization, as a percentage of the instance's CPU resources, grouped by user-initiated operations such as reads, writes, and commits. Use this metric to get a detailed breakdown of CPU usage and to troubleshoot further, as explained in [Investigating high CPU utilization](/spanner/docs/introspection/investigate-cpu-utilization) . You can create a chart for this metric [in the Cloud Monitoring console](/spanner/docs/monitoring-cloud) .
    
    You can also use the Cloud Monitoring console to [create alerts for CPU utilization](/spanner/docs/monitoring-cloud#create-alert) , as described later.

## Alerts for high CPU utilization

The following table specifies our recommendations for maximum CPU usage for [regional, dual-region, and multi-region instances](/spanner/docs/instance-configurations) . These numbers are to ensure that your instance has enough [compute capacity](/spanner/docs/compute-capacity) to continue to serve your traffic in the event of the loss of an entire zone (for regional instances) or an entire region (for dual-region and multi-region instances).

<table>
<thead>
<tr class="header">
<th>Metric</th>
<th>Maximum for regional instances</th>
<th>Maximum per region for dual-region and multi-region instances</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>High priority total</strong></td>
<td>65%</td>
<td>45%</td>
</tr>
<tr class="even">
<td><strong>24-hour smoothed aggregate</strong></td>
<td>90%</td>
<td>90%</td>
</tr>
</tbody>
</table>

To help you stay below the recommended maximums, [create alerts in Cloud Monitoring](/spanner/docs/monitoring-cloud#create-alert) that track high-priority CPU utilization and the average CPU utilization over 24 hours.

CPU utilization can have an impact on request latencies. Overloading of an individual backend server will trigger higher request latencies. Applications should run benchmarks and active monitoring to verify that Spanner meets their performance requirements.

Thus, for performance-sensitive applications, you may need to further reduce CPU utilization using techniques described in the following section.

### CPU utilization over 100%

In certain cases the CPU utilization of a Spanner instance may reach above 100%. This means that the instance is using more CPU resources than the amount configured for the instance.

CPU resources above 100% might be used to provide better and more predictable performance during CPU utilization spikes, for example, caused by sudden increase in request traffic.

Any CPU capacity above 100% is NOT guaranteed and shouldn't be relied on for normal database operations.

Running a Spanner instance near or over 100% CPU utilization for an extended period of time has a risk of degrading normal operation performance and latency. Additional CPU resources are not a safe mechanism to rely on for consistent performance.

Customers are not billed for this additional CPU utilization.

## Reducing CPU utilization

This section explains how to reduce an instance's CPU utilization.

In general, we recommend that you increase the [compute capacity](/spanner/docs/compute-capacity) of your instance as a starting point. After you increase the compute capacity, you can investigate and address the root causes of high CPU utilization.

### Increasing compute capacity

If you exceed the recommended maximums for CPU utilization, we strongly recommend increasing the [compute capacity](/spanner/docs/compute-capacity) of your instance so it can continue to operate effectively. If you want to automate this process, you can create an application that monitors CPU utilization, then increases or decreases compute capacity as needed, using the [`  UpdateInstance  `](/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstanceAdmin.UpdateInstance) method.

To determine how much compute capacity you need, consider the peak high-priority CPU utilization as well as the 24-hour smoothed average. Always allocate enough compute capacity to keep the CPU utilization below the recommended maximums. As previously described, you may need to allocate extra compute capacity for performance-sensitive applications (for example, to accommodate workload spikes).

If you don't have enough compute capacity, Spanner postpones tasks by priority level. Low-priority system tasks, like database compaction and schema change validation, can be deferred in favor of user tasks. However, these tasks are critical to the health of your instance, and Spanner cannot defer them indefinitely. If Spanner cannot complete its low-priority system tasks within a certain time window—on the order of several hours to a day—due to insufficient compute resources, Spanner might increase the priority of the system tasks. **This change affects the performance of user tasks** .

### Investigating further with introspection tools

If the **CPU Utilization by operation type** metric indicates that a particular type of operation is contributing to high CPU utilization, use the Spanner introspection tools to troubleshoot further. For more information, see [Investigating high CPU utilization](/spanner/docs/introspection/investigate-cpu-utilization) .

## What's next

  - Monitor your instance with the [Google Cloud console](/spanner/docs/monitoring-console) or the [Cloud Monitoring console](/spanner/docs/monitoring-cloud) .

  - [Create alerts for Spanner CPU utilization](/spanner/docs/monitoring-cloud#create-alert) .

  - Find out how to [change the compute capacity](/spanner/docs/create-manage-instances#change-compute-capacity) of a Spanner instance.

  - Learn how to [find correlations between high latency and other metrics](/spanner/docs/monitoring-cloud#create-charts) .

  - To learn how to troubleshoot high CPU usage caused by a particular operation type, see [Investigating high CPU utilization](/spanner/docs/introspection/investigate-cpu-utilization) .
