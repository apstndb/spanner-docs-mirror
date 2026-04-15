> **Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

> **Note:** You can enable the Spanner managed autoscaler on both [instances](https://docs.cloud.google.com/spanner/docs/instances) and [instance partitions (in Preview)](https://docs.cloud.google.com/spanner/docs/geo-partitioning#how_geo-partitioning_works) . For brevity, this page uses the term *instance* to refer to both resources, unless the context specifically requires a distinction.

This page describes how the managed autoscaler works and describes costs and limitations when using the Spanner managed autoscaler. It also provides information to help you determine how to configure the managed autoscaler.

## How managed autoscaler works

When you enable the managed autoscaler, Spanner automatically adjusts the size of your instance for you. You can enable the managed autoscaler in your Spanner instance or instance partition (in [Preview](https://cloud.google.com/products#product-launch-stages) ). The managed autoscaler feature reacts to changes in your instance's workload or storage needs as your load increases or decreases. Managed autoscaler either *scales up* , adding compute capacity to the instance, or it *scales down* , removing compute capacity from the instance.

When you configure the managed autoscaler, you can use either processing units for small instances, or nodes for large instances. In this document, we use the term *compute capacity* to mean nodes or processing units.

The Spanner managed autoscaler determines how much compute capacity is required, based on the following:

  - High priority CPU utilization target
  - Total CPU utilization target
  - Storage utilization target
  - Minimum limit
  - Maximum limit

Each scaling dimension generates a recommended instance size, and Spanner automatically uses the highest one. This means, for example, that if your instance needs 10 nodes to meet your storage utilization target but 12 nodes to meet your CPU utilization target, Spanner scales the instance to 12 nodes.

As the amount of compute capacity changes, Spanner continually optimizes the storage. It rebalances data across all servers to ensure that traffic is spread evenly and no individual server is overloaded. For more information, see [Limitations](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#limitations) .

If the managed autoscaler scales an instance up to its maximum limit but the workload is still causing higher CPU utilization than [the target](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#autoscaler-parameters) , workload requests might have a higher latency or fail. If an instance scales up to its maximum compute capacity target but the workload needs more storage than the maximum storage limit, write requests can fail. To find out if the maximum target is reached, you can view the managed autoscaler system event logs in the Google Cloud console on the **System insights** page. For more information, see [storage limits](https://docs.cloud.google.com/spanner/quotas#node_limits) .

When Spanner scales an instance down, it removes compute capacity at a slower rate than when scaling up, to reduce any impact on latency.

You can choose to asymmetrically autoscale read-only replicas in your instances. You can't asymmetrically autoscale instance partitions. For more information, see [Asymmetric read-only autoscaling](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#asymmetric-read-only-autoscaling) .

## Pricing

Your total Spanner costs might be lower or higher depending on how you configured your Spanner instance or instance partition prior to enabling the managed autoscaler and the limits you set for the managed autoscaler.

For example, if you used to manually configure your Spanner instance to have enough compute capacity to handle peak workloads at any time, your costs with the managed autoscaler might be lower because it reduces compute capacity when the instance is idle.

If you used to manually configure your Spanner instance to have enough compute capacity for average workloads and overall performance degrades when your workload traffic increases, your costs with the managed autoscaler might be higher because the managed autoscaler might increase compute capacity when the instance is busy. However, this provides your users with more consistent performance.

You can limit the maximum cost of your Spanner instance by setting the maximum nodes or processing units limit to the level you want to spend.

You might see an increase in compute capacity used and therefore an increase in costs when you set a total CPU utilization target on your Spanner instance in comparison to only setting a high priority CPU utilization target. However, the end user has a significantly better experience and improved performance when this option is set.

## Limitations

The following limitations apply when you enable or change the managed autoscaling feature on an instance or instance partition:

  - You can't [move an instance](https://docs.cloud.google.com/spanner/docs/move-instance) when the managed autoscaler feature is enabled. You must first disable the managed autoscaler, and then move the instance. After you move the instance, you can re-enable the managed autoscaler.
  - You must set the minimum limit on the autoscaling instance to 1000 processing units or greater, or 1 node or greater.
  - When you enable autoscaling on an existing instance, the existing instance capacity can be lower than the minimum limit value you [configure on the managed autoscaler](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#configure_the_managed_autoscaler) . However, the instance automatically scales up to the configured minimum value when you start it. For example, if your instance has one node but you set the minimum value to two nodes, when you start your instance, it automatically scales up to two nodes.
  - You can't asymmetrically autoscale instance partitions.
  - If the number of placement rows in your partition is greater than 100 million, don't enable autoscaling. This is a geo-partitioning [limit](https://docs.cloud.google.com/spanner/quotas#geo-partitioning_limits) .

## Managed autoscaler parameters

When you create or edit an instance or instance partition and choose to enable the managed autoscaler, you define the values shown in the following table.

| Parameter                            | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| High priority CPU utilization target | A percentage of the instance's CPU capacity to use for high priority tasks. This value must be from 10% to 90%. When an instance's high priority CPU utilization exceeds the target that you have set, Spanner immediately adds compute capacity to the instance. When CPU utilization is substantially lower than the target, Spanner removes compute capacity. For more information, see [Determine the high priority CPU utilization target](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#determine-high-priority-cpu) .        |
| Total CPU utilization target         | A percentage of the instance's total CPU capacity to use for high-, medium-, and low-priority tasks. This value must be from 10% to 90%. When an instance's total CPU utilization exceeds the target that you have set, Spanner immediately adds compute capacity to the instance. When total CPU utilization is substantially lower than the target, Spanner removes compute capacity. For more information, see [Determine the total CPU utilization target](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#determine-total-cpu) . |
| Storage utilization target           | The percentage of storage on a node that you can use before Spanner scales up. This target ensures that you always have enough compute capacity to handle fluctuations in the amount of data that you store. This value must be between 10-99%. For guidance, see [Determine the storage utilization target](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#determine-storage) .                                                                                                                                                     |
| Minimum limit                        | The lowest amount of compute capacity that Spanner scales the instance down to. The minimum value can't be lower than 10% of the value you set for the maximum limit. For example, if the maximum limit is 40 nodes, the minimum limit must be at least 4 nodes. The 10% requirement is a hard limit. For guidance, see [Determine the minimum limit](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#determine-minimum) .                                                                                                            |
| Maximum limit                        | The highest amount of compute capacity that Spanner scales the instance up to. For nodes, this value must be greater than 1 node (or 1000 processing units) and equal to or greater than the minimum number of nodes or processing units. The value can't be more than 10 times the number that you choose for the minimum amount of compute capacity. This 10 times requirement is a hard limit. For guidance, see [Determine the maximum limit](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#determine-maximum) .                |

### Configure the managed autoscaler

This section describes how to determine what numbers to choose for your managed autoscaler parameters. After you set your initial values, [monitor your instance](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#monitoring) and adjust the numbers if necessary.

#### Determine the high-priority CPU utilization target

The optimal target for your instance or instance partition depends on the latency and throughput requirements of your workload. To view our recommendations for maximum CPU usage for regional, dual-region, and multi-region instance configurations, see [Alerts for high CPU utilization](https://docs.cloud.google.com/spanner/docs/cpu-utilization#recommended-max) .

When [CPU utilization is near or over 100%](https://docs.cloud.google.com/spanner/docs/cpu-utilization#cpu_utilization_over_100) , it can potentially cause performance to degrade. If your workload is latency or performance sensitive, consider customizing the total CPU target to a lower value. Note that doing so might incur higher costs.

In general, if you observe unacceptably high latency, you should lower the CPU utilization target.

You can also configure targets for both total and high-priority CPU utilization. For more information, see [Determine both CPU utilization targets](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#determine-both-cpu-targets) .

#### Determine the total CPU utilization target

When you set the total CPU utilization target, Spanner autoscales to ensure sufficient capacity for [high, medium, and low priority tasks](https://docs.cloud.google.com/spanner/docs/cpu-utilization#task-priority) .

If your workloads are latency sensitive, or if you want system tasks to finish sooner, you have to set total CPU target to ensure that the instance has enough capacity. When the total CPU target is set, you might pay more, but your applications provide a better experience for your customers.

If the total CPU target is set and you still observe unacceptably high latency, you should lower the total CPU utilization target.

To optimize throughput of writes and index creation, we recommend a total CPU target of 70% for regional instances and 50% for multi-region instances. This also works well during failover, if high-priority target is not selected. However, these targets might incur higher costs. If cost is a concern, we recommend a total CPU target of 85%. This provides overhead to absorb spikes without triggering [latency caused by resource saturation (100% utilization)](https://docs.cloud.google.com/spanner/docs/cpu-utilization#cpu_utilization_over_100) .

By default, Spanner prioritizes user-facing traffic by throttling resource-intensive background operations (such as index creation). You can accelerate these background operations by configuring a lower total CPU utilization target (for example, \<=60%). This signals the autoscaler to provision additional compute resources, increasing the throughput of system tasks. However, this might increase costs. If you want to temporarily increase throughput for index creation, you can set lower total CPU targets until index creation completes.

You can also configure targets for both total and high-priority CPU utilization. For more information, see [Determine both CPU utilization targets](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#determine-both-cpu-targets) .

#### Determine both CPU utilization targets

If you configure targets for both total CPU and high-priority CPU utilization, the autoscaler evaluates both metrics simultaneously. It then selects the higher of the two recommended node or processing unit counts. This ensures that the instance scales up to satisfy whichever requirement is more demanding, maintaining performance for critical workloads while completing background tasks.

When both the high-priority CPU and total CPU utilization targets are set, the CPU utilization for high-priority tasks are part of that total, along with low and medium priority tasks. The value of the high priority CPU utilization target has to be smaller than the total CPU utilization target when both options are selected.

In general, if you observe unacceptably high latency, you should lower the CPU utilization target.

Generally, we recommend the following CPU utilization targets for reliable failover:

| Instance type         | Total CPU utilization target | High priority CPU utilization target |
| :-------------------- | :--------------------------- | :----------------------------------- |
| Regional instance     | 70%                          | 65%                                  |
| Multi-region instance | 50%                          | 45%                                  |

Depending on your workload, we also recommend the following more specific CPU utilization targets:

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">Workload type</th>
<th style="text-align: left;">Recommended CPU targets</th>
<th style="text-align: left;">Trade-off</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Throughput sensitive, write-heavy workload</td>
<td style="text-align: left;">Total CPU utilization target: 70%</td>
<td style="text-align: left;">Higher throughput at the expense of latency</td>
</tr>
<tr class="even">
<td style="text-align: left;">Latency sensitive, read-heavy workload</td>
<td style="text-align: left;">Total CPU utilization target: 80%<br />
<br />
High priority CPU utilization target: 65% (regional) or 45% (multi-region)</td>
<td style="text-align: left;">Predictable tail latency at higher cost</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Workload prioritizing cost efficiency</td>
<td style="text-align: left;">Total CPU utilization target: 85%<br />
<br />
High priority CPU utilization target: 65% (regional) or 45% (multi-region)</td>
<td style="text-align: left;">Reasonable cost and performance with potentially delayed index creation</td>
</tr>
</tbody>
</table>

#### Determine the storage utilization target

For autoscaling, the storage utilization target is expressed as a percentage per node. For instances or instance partitions that are 1 node (1000 processing units) and larger, storage size is limited to 10 TiB per node.

#### Determine the maximum limit

The value that you choose as the maximum amount of compute capacity is equal to the amount of compute capacity that the instance or instance partition needs to handle the heaviest traffic, even if you don't expect to reach that volume most of the time. Spanner never scales up to more compute capacity than it needs. You can also think of this number as the highest amount of compute capacity that you are willing to pay for. For more information on accepted values, see [Autoscaler parameters](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#autoscaler-parameters) .

The maximum limit must allow for both the CPU utilization target and the storage utilization target that you set for autoscaling.

  - If you're changing an instance from manual allocation to managed autoscaling, find the highest amount of compute capacity that the instance has had over the last one or two months. Your managed autoscaler maximum limit should be at least that high.

  - If you are enabling the managed autoscaler for a new instance, look at metrics from other instances and use them as a guide when you set the maximum limit.

  - If you have a new workload and you're not sure how it's going to grow, you can estimate the amount of compute capacity that you need to meet the built-in storage utilization target and then adjust the number later.

You also need to know how much quota is remaining on your node because the managed autoscaler can't configure your instance to have more compute capacity than your quota. For more information, see [Node limits](https://docs.cloud.google.com/spanner/quotas#node_limits) .

After your instance is up and running with autoscaling enabled, [monitor the instance](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#monitoring) and make sure that the value you chose for the maximum limit is at least as high as the recommended limit for CPU target and the recommended limit for the storage target.

#### Determine the minimum limit

You set a minimum limit for managed autoscaler to ensure that your Spanner instance or instance partition can scale down to the smallest, most cost-efficient size. Spanner automatically prevents the node count from dropping below the minimum needed to maintain the CPU and storage utilization targets.

The smallest minimum value that the managed autoscaler permits is 1 node or 1000 processing units. When you enable autoscaling for an existing instance that has less capacity than the minimum value configured for the managed autoscaler, the instance automatically scales up to this minimum when you start it.

After starting the instance that has managed autoscaling, you should perform an initial test to ensure it works at the minimum set size. You should test again periodically to ensure it continues to work as expected.

For more information about accepted values, see [Managed autoscaler parameters](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#autoscaler-parameters) .

In many cases you want to set the minimum value to more than one. Choose a higher number or raise the minimum limit for the following situations:

  - You have an upcoming peak scale event when you expect your traffic to temporarily increase, and you want to make sure you have enough compute capacity.
  - Your application sends spiky traffic. When you add new compute capacity, Spanner automatically rebalances to use the new nodes or processing units. Because this process can take several minutes, you might want to consider taking a conservative approach and choosing a higher minimum. That way, your instance seamlessly accommodate the spikes.
  - You increase the maximum compute capacity. The minimum must always be ten percent or more of the maximum compute capacity target. For example, if you set the maximum number of nodes to `30` , you must set the minimum number of nodes to at least `3` .

If you increase the value for the minimum compute capacity on an instance, Spanner immediately tries to scale the instance to the new minimum. The [standard constraints](https://docs.cloud.google.com/spanner/quotas#node_limits) apply. When you're out of quota, your request to change the managed autoscaler configuration fails and the configuration isn't updated.

After you first configure managed autoscaler, and periodically thereafter, test your instance to ensure that it works at the minimum size.

#### Google Cloud CLI parameter flags and limitations

When you use Google Cloud CLI to configure the managed autoscaler, there are some required flags you must set. There are optional flags that you use to indicate whether you want to use nodes or processing units. For more information about creating a new instance or instance partition with the managed autoscaler, or enabling managed autoscaler on an existing instance or instance partition, see the following how-to guides:

  - [Create an instance](https://docs.cloud.google.com/spanner/docs/create-manage-instances#gcloud)
  - [Enable or modify the managed autoscaler on an instance](https://docs.cloud.google.com/spanner/docs/create-manage-instances#gcloud_4)
  - [Create an instance partition](https://docs.cloud.google.com/spanner/docs/create-manage-partitions#create-instance-partition)
  - [Enable or modify the managed autoscaler on an instance partition](https://docs.cloud.google.com/spanner/docs/create-manage-partitions#enable-modify-managed-autoscaler)

The following flags are required when enabling the managed autoscaler on your instance:

  - `autoscaling-high-priority-cpu-percent`
  - `autoscaling-total-cpu-percent`
  - `autoscaling-storage-percent`

When setting the CPU percent, you can select either one or both options.

If you choose to use nodes, you must also use both of the following flags when you enable the managed autoscaler:

  - `autoscaling-min-nodes`
  - `autoscaling-max-nodes`

If you choose to use processing units, you must also use both of the following flags when you enable the managed autoscaler:

  - `autoscaling-min-processing-units`
  - `autoscaling-max-processing-units`

The following limitations apply when adding the managed autoscaler to an existing instance using Google Cloud CLI:

  - You can't use the `--nodes` flag with the `--autoscaling-min-nodes` or `--autoscaling-max-nodes` flags because using `--nodes` sets a specific number of nodes rather than a scaling range. Similarly, you can't use the `--processing-units` flag with the `autoscaling-min-processing-units` or `autoscaling-max-processing-units` flags because using `--processing-units` sets a specific number of processing units rather than a scaling range.
  - You can't mix the flags for nodes and processing units together. For example, you can't use `--autoscaling-max-nodes` with `autoscaling-min-processing-units` .

### Fine-tune your settings

Keep an eye on your compute capacity usage and adjust your settings, if necessary, especially after you first enable the managed autoscaler. We recommend using the [System insights](https://docs.cloud.google.com/spanner/docs/monitoring-console) page in the Google Cloud console.

## Asymmetric read-only autoscaling

> **Note:** You can't use asymmetric autoscaling with read-write and witness replicas. You also can't enable asymmetric autoscaling on instance partitions.

After you enable managed autoscaler, you can also enable and autoscale your read-only replicas independently from other replicas. Asymmetric read-only autoscaling lets you control the compute capacity limits and CPU utilization targets of your read-only regions based on their usage. This optimizes local read traffic patterns and improves cost efficiency. The following autoscaling configuration parameters are configurable for each read-only replica region:

  - Minimum compute capacity limit
  - Maximum compute capacity limit
  - High priority CPU utilization target
  - Total CPU utilization target
  - Disable total CPU
  - Disable high priority CPU

You can enable asymmetric autoscaling and configure these parameters by [creating a new instance](https://docs.cloud.google.com/spanner/docs/create-manage-instances#create-instance) or by [updating an existing instance](https://docs.cloud.google.com/spanner/docs/create-manage-instances#modify-managed-autoscaler) .

For each replica, the following rules apply when you enable asymmetric autoscaling on an existing instance:

  - If the current compute capacity of the replica is between the autoscaling minimum and maximum that is set for the region, then the compute capacity of the replica doesn't change.
  - If the current compute capacity of the replica is below the autoscaling minimum that is set for the region, then the compute capacity is adjusted to match the autoscaling minimum.
  - If the current compute capacity of the replica is above the autoscaling maximum that is set for the region, then the compute capacity is adjusted to match the autoscaling maximum.
  - If both CPU targets are set at the base level and you want to disable the CPU target at the replica level, you must explicitly use `disable_total_cpu_autoscaling` or `disable_high_priority_cpu_autoscaling` .

In addition, when using the asymmetric autoscaler, we recommend setting the same set of targets across all replicas to ensure consistent autoscaling behavior during failover events. For more information, see [Failover concerns](https://docs.cloud.google.com/spanner/docs/managed-autoscaler#failover-concerns) .

### Failover concerns

To maintain high availability and performance during an outage, you must ensure your instance has sufficient compute capacity to handle traffic if a zone (for regional instances) or an entire region (for dual-region and multi-region instances) becomes unavailable.

When using the asymmetric autoscaler, it's critical to apply the same utilization targets across all replicas. Inconsistent configurations can lead to capacity bottlenecks during a failover.

Consider the following scenario:

  - Replica A is configured with both high-priority and total CPU targets.
  - Replica B is configured with only a high-priority CPU target.

If a failover shifts traffic from Replica A to Replica B, Replica B only scales based on high-priority requests. Consequently, medium and low-priority tasks (such as background system processes or analytical queries) don't trigger the necessary autoscaling on Replica B, potentially leading to task starvation or increased latency for non-critical workloads.

To prevent problems, we recommend the following:

  - Always define identical autoscaler targets across all replicas to ensure consistent autoscaling behavior. For example, consider a scenario where you configure a read-only replica with both a high-priority CPU target and a total CPU target. If the read-write replica only sets the high-priority CPU target, then during failover, medium- and low-priority traffic won't trigger autoscaling on the read-write replica.
  - Ensure your target utilization has capacity for traffic bursts that occur when one replica must suddenly absorb the load of a failed peer.
  - Periodically review your Cloud Monitoring metrics to verify that secondary replicas have the capacity required to support the combined traffic of your primary deployment.

## Access control

To configure the managed autoscaler, you need to be a principal in a role that has [create and update permissions](https://docs.cloud.google.com/spanner/docs/grant-permissions#add_instance-level_permissions) for the instance or instance partition that you are configuring.

## Monitoring

Spanner provides several metrics to help you understand how well the managed autoscaler is working as it scales up and down to meet workload requirements. The metrics can also help you gauge whether your settings are optimal to meet your business's workload and cost requirements. For example, if you observe that the node count for an instance or instance partition is often close to the maximum number of nodes, you might consider raising the maximum. To learn more about monitoring your Spanner resources, see [Monitor instances with Cloud Monitoring](https://docs.cloud.google.com/spanner/docs/monitoring-cloud) .

The following metrics are [displayed in graphs](https://docs.cloud.google.com/spanner/docs/monitoring-console) on the **System insights** page in the Google Cloud console. You can also view these metrics using [Cloud Monitoring](https://docs.cloud.google.com/spanner/docs/monitoring-cloud) .

  - `spanner.googleapis.com/instance/autoscaling/min_node_count`
  - `spanner.googleapis.com/instance/autoscaling/max_node_count`
  - `spanner.googleapis.com/instance/autoscaling/min_processing_units`
  - `spanner.googleapis.com/instance/autoscaling/max_processing_units`
  - `spanner.googleapis.com/instance/autoscaling/high_priority_cpu_target_utilization`
  - `spanner.googleapis.com/instance/autoscaling/total_cpu_target_utilization`
  - `spanner.googleapis.com/instance/autoscaling/storage_target_utilization`

## Logging

Spanner creates a system event audit log each time it scales an instance or instance partition. Each event log has description text and metadata related to the autoscaling event.

### View logs on the System insights page

You can view the managed autoscaler system event logs in the Google Cloud console on the **System insights** page.

1.  In the Google Cloud console, open Spanner:

2.  Select the autoscaling-enabled instance or instance partition.

3.  In the navigation menu, click **System insights** .

4.  On the System insights page, navigate to the **Compute capacity** metric.

5.  Click **View logs** to open the log panel.
    
    The **Compute capacity logs** pane displays the logs for the last hour.
    
    If asymmetric read-only autoscaling is enabled for your instance, then the log summary provides a description and location of every replica's compute capacity changes. For example, `Increased from 1 to 2 nodes in us-central1 to maintain high priority CPU utilization at 80%` . If you're not using asymmetric autoscaling, location information isn't provided in the log summary. For example, `Increased from 9 to 10 nodes to maintain high priority CPU utilization at 65%` . You can also see when nodes are increased to maintain the total CPU utilization target.

### View logs using Logs Explorer

You can also view logs using Logs Explorer:

1.  In the Google Cloud console, open the Logs Explorer:

2.  Select the appropriate Google Cloud project.

3.  In the **Query** field, enter the following:
    
    ``` 
     protoPayload.methodName="AutoscaleInstance"
    ```
    
    You can add the following query to filter down further into the logs:
    
        resource.type="spanner_instance"
        resource.labels.instance_id=INSTANCE_ID
        resource.labels.project_id=PROJECT_ID
        logName="projects/PROJECT_ID/logs/cloudaudit.googleapis.com%2Fsystem_event"
        protoPayload.methodName="AutoscaleInstance"
    
    To view logs for queries run in an non-default instance partition, enter:
    
        resource.type="spanner_instance"
        resource.labels.instance_id=INSTANCE_ID
        resource.labels.project_id=PROJECT_ID
        logName="projects/PROJECT_ID/logs/cloudaudit.googleapis.com%2Fsystem_event"
        protoPayload.methodName="AutoscaleInstancePartition"

4.  Click **Run query** .

The **Query results** pane displays the logs for the last hour.

To learn more about viewing logs, see [Cloud Logging](https://docs.cloud.google.com/logging/docs/overview) . You can set up [log-based alerts](https://docs.cloud.google.com/logging/docs/alerting/log-based-alerts#lba-definition) in the **Logs explorer** page in the Google Cloud or by using the [Cloud Monitoring API](https://docs.cloud.google.com/logging/docs/alerting/log-based-alerts#lba-by-api) .

## What's next

  - Learn how to [create an instance with the managed autoscaler enabled](https://docs.cloud.google.com/spanner/docs/create-manage-instances#create-instance)
  - Learn how to [modify an instance to use autoscaling or change autoscaling settings](https://docs.cloud.google.com/spanner/docs/create-manage-instances#modify-managed-autoscaler)
  - Learn how to [change an instance from using autoscaling to manual scaling](https://docs.cloud.google.com/spanner/docs/create-manage-instances#remove-managed-autoscaler)
  - Learn how to [create an instance partition with the managed autoscaler enabled](https://docs.cloud.google.com/spanner/docs/create-manage-partitions#create-instance-partition)
  - Learn how to [modify an instance partition to use autoscaling or change autoscaling settings](https://docs.cloud.google.com/spanner/docs/create-manage-partitions#enable-modify-managed-autoscaler)
