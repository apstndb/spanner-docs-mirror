**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes how the managed autoscaler works and describes costs and limitations when using the Spanner managed autoscaler. It also provides information to help you determine how to configure the managed autoscaler.

## How managed autoscaler works

When you enable the managed autoscaler, Spanner automatically adjusts the size of your instance for you. The managed autoscaler feature reacts to changes in your instance's workload or storage needs as your load increases or decreases. Managed autoscaling either *scales up* , adding compute capacity to the instance, or it *scales down* , removing compute capacity from the instance.

When you configure the managed autoscaler, you can use either processing units for small instances, or nodes for large instances. In this document, we use the term *compute capacity* to mean nodes or processing units.

The Spanner managed autoscaler determines how much compute capacity is required, based on the following:

  - High priority CPU utilization target
  - Storage utilization target
  - Minimum limit
  - Maximum limit

Each scaling dimension generates a recommended instance size, and Spanner automatically uses the highest one. This means, for example, that if your instance needs 10 nodes to meet your storage utilization target but 12 nodes to meet your CPU utilization target, Spanner scales the instance to 12 nodes.

As the amount of compute capacity changes, Spanner continually optimizes the storage. It rebalances data across all servers to ensure that traffic is spread evenly and no individual server is overloaded. See [Limitations](#limitations) for more information.

If the managed autoscaler scales an instance up to its maximum limit but the workload is still causing higher CPU utilization than [the target](#autoscaler-parameters) , workload requests might have a higher latency or fail. If an instance scales up to its maximum compute capacity target but the workload needs more storage than the maximum storage limit, write requests can fail. To find out if the maximum target is reached, you can view the managed autoscaler system event logs in the Google Cloud console on the **System insights** page. For more information, see [storage limits](/spanner/quotas#node_limits) .

When Spanner scales an instance down, it removes compute capacity at a slower rate than when scaling up, to reduce any impact on latency.

You can choose to asymmetrically autoscale your read-only replicas. For more information, see [Asymmetric read-only autoscaling](#asymmetric-read-only-autoscaling) .

## Costs

Your total Spanner costs might be lower or higher depending on how you configured your Spanner instance prior to enabling the managed autoscaler and the limits you set for the managed autoscaler.

For example, if you used to manually configure your Spanner instance to have enough compute capacity to handle peak workloads at any time, your costs with the managed autoscaler might be lower because it reduces compute capacity when the instance is idle.

If you used to manually configure your Spanner instance to have enough compute capacity for average workloads and overall performance degrades when your workload traffic increases, your costs with the managed autoscaler might be higher because the managed autoscaler might increase compute capacity when the instance is busy. However, this provides your users with more consistent performance.

You can limit the maximum cost of your Spanner instance by setting the maximum nodes or processing units limit to the level you want to spend.

## Limitations

The following limitations apply when you enable or change the managed autoscaling feature on an instance:

  - You can't [move an instance](/spanner/docs/move-instance) when the managed autoscaler feature is enabled. You must first disable the managed autoscaler and then move the instance. After you move the instance, you can re-enable the managed autoscaler.
  - You must set the min limit on the autoscaling instance to 1000 processing units or greater, or 1 node or greater.
  - When you enable autoscaling on an existing instance, the existing instance capacity can be lower than the min limit value you [configure on the managed autoscaler](/spanner/docs/managed-autoscaler#configure_the_managed_autoscaler) . However, the instance automatically scales up to the configured minimum value when you start it. For example, if your instance has one node but you set the minimum value to two nodes, when you start your instance, it automatically scales up to 2 nodes.
  - The managed autoscaler scales CPU for high priority workloads to follow Spanner's [high-priority CPU recommendation](/spanner/docs/cpu-utilization#recommended-max) in the event of loss of a zone or region. It doesn't take total CPU utilization into consideration. When [CPU utilization is over 100%](/spanner/docs/cpu-utilization#cpu_utilization_over_100) , it can potentially cause the performance to degrade. If your workload is latency or performance sensitive, consider customizing the [open source Autoscaler tool](/spanner/docs/autoscaler-tool-overview) to scale based on total CPU.

## Managed autoscaler parameters

When you create or edit an instance and choose to enable the managed autoscaler, you define the values shown in the following table.

<table>
<thead>
<tr class="header">
<th>Parameter</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>High priority CPU utilization target</td>
<td>A percentage of the instance's high priority CPU capacity. This value must be from 10% to 90%. When an instance's CPU utilization exceeds the target that you have set, Spanner immediately adds compute capacity to the instance. When CPU utilization is substantially lower than the target, Spanner removes compute capacity. For guidance, see <a href="#determine-cpu">Determine the CPU utilization target</a> .</td>
</tr>
<tr class="even">
<td>Storage utilization target</td>
<td>The percentage of storage on a node that you can use before Spanner scales up. This target ensures that you always have enough compute capacity to handle fluctuations in the amount of data that you store. This value must be between 10-99%. For guidance, see <a href="#determine-storage">Determine the storage utilization target</a> .</td>
</tr>
<tr class="odd">
<td>Minimum limit</td>
<td>The lowest amount of compute capacity that Spanner scales the instance down to. The minimum value can't be lower than 10% of the value you set for the maximum limit. For example, if the maximum limit is 40 nodes, the minimum limit must be at least 4 nodes. The 10% requirement is a hard limit. For guidance, see <a href="#determine-minimum">Determine the minimum limit</a> .</td>
</tr>
<tr class="even">
<td>Maximum limit</td>
<td>The highest amount of compute capacity that Spanner scales the instance up to. For nodes, this value must be greater than 1 node (or 1000 processing units) and equal to or greater than the minimum number of nodes or processing units. The value can't be more than 10 times the number that you choose for the minimum amount of compute capacity. This 10 times requirement is a hard limit. For guidance, see <a href="#determine-maximum">Determine the maximum limit</a> .</td>
</tr>
</tbody>
</table>

### Configure the managed autoscaler

This section describes how to determine what numbers to choose for your managed autoscaler parameters. After you set your initial values, [monitor your instance](#monitoring) and adjust the numbers if necessary.

#### Determine the high priority CPU utilization target

The optimal target for your instance depends on the latency and throughput requirements of your workload. To view our recommendations for maximum CPU usage for regional, dual-region, and multi-region instance configurations, see [Alerts for high CPU utilization](/spanner/docs/cpu-utilization#recommended-max) .

The managed autoscaler considers high priority workloads when scaling CPU usage. It doesn't consider total CPU utilization. When CPU utilization is over 100%, it can potentially cause performance to degrade. If your workload is latency or performance sensitive, consider customizing the [open source Autoscaler tool](/spanner/docs/autoscaler-tool-overview) to scale based on total CPU.

In general, if you observe unacceptably high latency, you should lower the CPU utilization target.

#### Determine the storage utilization target

For autoscaling, the storage utilization target is expressed as a percentage per node. For instances that are 1 node (1000 processing units) and larger, storage size is limited to 10 TiB per node.

#### Determine the maximum limit

The value that you choose as the maximum amount of compute capacity is equal to the amount of compute capacity that the instance needs to handle the heaviest traffic, even if you don't expect to reach that volume most of the time. Spanner never scales up to more compute capacity than it needs. You can also think of this number as the highest amount of compute capacity that you are willing to pay for. See [Autoscaler parameters](#autoscaler-parameters) for details on accepted values.

The maximum limit must allow for both the CPU utilization target and the storage utilization target that you set for autoscaling.

  - If you're changing an instance from manual allocation to managed autoscaling, find the highest amount of compute capacity that the instance has had over the last one or two months. Your managed autoscaler maximum limit should be at least that high.

  - If you are enabling the managed autoscaler for a new instance, look at metrics from other instances and use them as a guide when you set the maximum limit.

  - If you have a new workload and you're not sure how it's going to grow, you can estimate the amount of compute capacity that you need to meet the built-in storage utilization target and then adjust the number later.

You also need to know how much quota is remaining on your node because the managed autoscaler can't configure your instance to have more compute capacity than your quota. For more information, see [Node limits](/spanner/quotas#node_limits) .

After your instance is up and running with autoscaling enabled, [monitor the instance](#monitoring) and make sure that the value you chose for the maximum limit is at least as high as the recommended limit for CPU target and the recommended limit for the storage target.

#### Determine the minimum limit

You set a minimum limit for managed autoscaler to ensure that your Spanner instance can scale down to the smallest, most cost-efficient size. Spanner automatically prevents the node count from dropping below the minimum needed to maintain the CPU and storage utilization targets.

The smallest minimum value that the managed autoscaler permits is 1 node or 1000 processing units. When you enable autoscaling for an existing instance that has less capacity than the minimum value configured for the managed autoscaler, the instance automatically scales up to this minimum when you start it.

After starting the instance that has managed autoscaling, you should perform an initial test to ensure it works at the minimum set size. You should test again periodically to ensure it continues to work as expected.

For more information about accepted values, see [Managed autoscaler parameters](#autoscaler-parameters) in this document.

In many cases you want to set the minimum value to more than one. Choose a higher number or raise the minimum limit for the following situations:

  - You have an upcoming peak scale event when you expect your traffic to temporarily increase, and you want to make sure you have enough compute capacity.
  - Your application sends spiky traffic. When you add new compute capacity, Spanner automatically rebalances to use the new nodes or processing units. Because this process can take several minutes, you might want to consider taking a conservative approach and choosing a higher minimum. That way, your instance seamlessly accommodate the spikes.
  - You increase the maximum compute capacity. The minimum must always be ten percent or more of the maximum compute capacity target. For example, if you set the maximum number of nodes to `  30  ` , you must set the minimum number of nodes to at least `  3  ` .

If you increase the value for the minimum compute capacity on an instance, Spanner immediately tries to scale the instance to the new minimum. The [standard constraints](/spanner/quotas#node_limits) apply. When you're out of quota, your request to change the managed autoscaler configuration fails and the configuration isn't updated.

After you first configure managed autoscaler, and periodically thereafter, test your instance to ensure that it works at the minimum size.

#### Google Cloud CLI parameter flags and limitations

When you use Google Cloud CLI to configure the managed autoscaler, there are some required flags you must set. There are optional flags that you use to indicate whether you want to use nodes or processing units. For more information about creating a new instance with the managed autoscaler, or enabling managed autoscaling on an existing instance, see the following:

  - [Create an instance](/spanner/docs/create-manage-instances#gcloud)
  - [Enable or modify the managed autoscaler on an instance](/spanner/docs/create-manage-instances#gcloud_4)

The following flags are required when enabling the managed autoscaler on your instance:

  - `  autoscaling-high-priority-cpu-percent  `
  - `  autoscaling-storage-percent  `

If you choose to use nodes, you must also use both of the following flags when you enable the managed autoscaler:

  - `  autoscaling-min-nodes  `
  - `  autoscaling-max-nodes  `

If you choose to use processing units, you must also use both of the following flags when you enable the managed autoscaler:

  - `  autoscaling-min-processing-units  `
  - `  autoscaling-max-processing-units  `

The following limitations apply when adding the managed autoscaler to an existing instance using Google Cloud CLI:

  - You can't use the `  --nodes  ` flag with the `  --autoscaling-min-nodes  ` or `  --autoscaling-max-nodes  ` flags because using `  --nodes  ` sets a specific number of nodes rather than a scaling range. Similarly, you can't use the `  --processing-units  ` flag with the `  autoscaling-min-processing-units  ` or `  autoscaling-max-processing-units  ` flags because using `  --processing-units  ` sets a specific number of processing units rather than a scaling range.
  - You can't mix the flags for nodes and processing units together. For example, you can't use `  --autoscaling-max-nodes  ` with `  autoscaling-min-processing-units  ` .

### Fine-tune your settings

Keep an eye on your compute capacity usage and adjust your settings, if necessary, especially after you first enable the managed autoscaler. We recommend using the [System insights](/spanner/docs/monitoring-console) page in the Google Cloud console.

## Asymmetric read-only autoscaling

**Note:** You can't use asymmetric autoscaling with read-write and witness replicas.

After you enable managed autoscaler, you can also enable and autoscale your read-only replicas independently from other replicas. Asymmetric read-only autoscaling lets you control the compute capacity limits and CPU utilization targets of your read-only regions based on their usage. This optimizes local read traffic patterns and improves cost efficiency. The following autoscaling configuration parameters are configurable for each read-only replica region:

  - Minimum compute capacity limit
  - Maximum compute capacity limit
  - High priority CPU utilization target

You can enable asymmetric autoscaling and configure these parameters by [creating a new instance](/spanner/docs/create-manage-instances#create-instance) or by [updating an existing instance](/spanner/docs/create-manage-instances#modify-managed-autoscaler) .

For each replica, the following rules apply when you enable asymmetric autoscaling on an existing instance:

  - If the current compute capacity of the replica is between the autoscaling minimum and maximum that is set for the region, then the compute capacity of the replica doesn't change.
  - If the current compute capacity of the replica is below the autoscaling minimum that is set for the region, then the compute capacity is adjusted to match the autoscaling minimum.
  - If the current compute capacity of the replica is above the autoscaling maximum that is set for the region, then the compute capacity is adjusted to match the autoscaling maximum.

## Access control

To configure the managed autoscaler, you need to be a principal in a role that has [create and update permissions](/spanner/docs/grant-permissions#add_instance-level_permissions) for the instance that you are configuring.

## Monitoring

Spanner provides several metrics to help you understand how well the managed autoscaler is working as it scales up and down to meet workload requirements. The metrics can also help you gauge whether your settings are optimal to meet your business's workload and cost requirements. For example, if you observe that the node count for an instance is often close to the maximum number of nodes, you might consider raising the maximum. To learn more about monitoring your Spanner resources, see [Monitor instances with Cloud Monitoring](/spanner/docs/monitoring-cloud) .

The following metrics are [displayed in graphs](/spanner/docs/monitoring-console) on the **System insights** page in the Google Cloud console. You can also view these metrics using [Cloud Monitoring](/spanner/docs/monitoring-cloud) .

  - `  spanner.googleapis.com/instance/autoscaling/min_node_count  `
  - `  spanner.googleapis.com/instance/autoscaling/max_node_count  `
  - `  spanner.googleapis.com/instance/autoscaling/min_processing_units  `
  - `  spanner.googleapis.com/instance/autoscaling/max_processing_units  `
  - `  spanner.googleapis.com/instance/autoscaling/high_priority_cpu_target_utilization  `
  - `  spanner.googleapis.com/instance/autoscaling/storage_target_utilization  `

## Logging

Spanner creates a system event audit log each time it scales an instance. Each event log has description text and metadata related to the autoscaling event.

### View logs on the System insights page

You can view the managed autoscaler system event logs in the Google Cloud console on the **System insights** page.

1.  In the Google Cloud console, open Spanner:

2.  Select the autoscaling-enabled instance.

3.  In the navigation menu, click **System insights** .

4.  On the System insights page, navigate to the **Compute capacity** metric.

5.  Click **View logs** to open the log panel.
    
    The **Compute capacity logs** pane displays the logs for the last hour.
    
    If asymmetric read-only autoscaling is enabled for your instance, then the log summary provides a description and location of every replica's compute capacity changes. For example, `  Increased from 1 to 2 nodes in us-central1 to maintain high priority CPU utilization at 80%  ` . If you're not using asymmetric autoscaling, location information isn't provided in the log summary. For example, `  Increased from 9 to 10 nodes to maintain high priority CPU utilization at 65%  ` .

### View logs using Logs Explorer

You can also view logs using Logs Explorer:

1.  In the Google Cloud console, open the Logs Explorer:

2.  Select the appropriate Google Cloud project.

3.  In the **Query** field, enter the following:
    
    ``` text
     protoPayload.methodName="AutoscaleInstance"
    ```
    
    You can add the following query to filter down further into the logs:
    
    ``` text
    resource.type="spanner_instance"
    resource.labels.instance_id=INSTANCE_ID
    resource.labels.project_id=PROJECT_ID
    logName="projects/span-cloud-testing/logs/cloudaudit.googleapis.com%2Fsystem_event"
    protoPayload.methodName="AutoscaleInstance"
    ```

4.  Click **Run query** .

The **Query results** pane displays the logs for the last hour.

To learn more about viewing logs, see [Cloud Logging](/logging/docs/overview) . You can set up [log-based alerts](/logging/docs/alerting/log-based-alerts#lba-definition) in the **Logs explorer** page in the Google Cloud or by using the [Cloud Monitoring API](/logging/docs/alerting/log-based-alerts#lba-by-api) .

## What's next

  - Learn how to [create an instance with the managed autoscaler enabled](/spanner/docs/create-manage-instances#create-instance)
  - Learn how to [modify an instance to add autoscaling or change autoscaling settings](/spanner/docs/create-manage-instances#modify-managed-autoscaler)
  - Learn how to [change an instance from using autoscaling to manual scaling](/spanner/docs/create-manage-instances#remove-managed-autoscaler)
