This page describes the storage utilization metrics that Spanner provides.

By default, your data is stored on solid-state drives (SSD) storage. You can choose whether to store your data on SSD or hard disk drives (HDD) by using tiered storage. For more information, see [Tiered storage overview](/spanner/docs/tiered-storage) .

## Storage metrics

Spanner provides the following storage metrics:

  - **Total database storage** : The amount of data that is stored in the database or the databases in the instance. This is subject to the [storage limit](/spanner/quotas#database_limits) .
  - **Total backup storage** : The amount of data that is stored by the backups associated with the instance or database. Backup storage is stored and billed separately and there is no limit on the amount you can store.

You can view charts for these metrics [in the Google Cloud console](/spanner/docs/monitoring-console) or [in the Cloud Monitoring console](/spanner/docs/monitoring-cloud) .

Also, database storage utilization is shown in the **Instances** and **Instance details** pages in the Google Cloud console.

### Multi-version storage

If you use the previous storage metrics to check the size of your data frequently, you might occasionally encounter results contrary to your expectations. For example, you might see the reported total storage of your database decrease by a noticeable amount, even though you hadn't recently removed any data. Conversely, you might see its size remain relatively unchanged right after performing a significant deletion.

These effects stem from Spanner's support for *multi-version storage* . Multi-version storage keeps all deleted or overwritten data in-storage and available for a limited time to enable features that let you read previous data values, such as [stale reads](/spanner/docs/reads#read_types) and [point-in-time recovery](/spanner/docs/pitr) . Performing a large data deletion doesn't immediately get reflected in your database's storage metrics. Similarly, an apparently unprompted drop in a database's total size likely means that Spanner's regular data-compaction process recently cleaned up a large set of data that was deleted or overwritten as far back as several days ago.

Spanner guarantees the continued availability of deleted or overwritten data for the interval defined by the [`  version_retention_period  `](/spanner/docs/reference/rest/v1/projects.instances.databases#Database.FIELDS.version_retention_period) option (one hour, by default). It automatically runs a background process every several days that permanently removes all obsolete data older than this version-retention interval.

### Effects of splitting

During periods of high load or hotspots, Spanner uses splitting as one technique to more evenly distribute your CPU utilization across your provisioned compute resources. One side effect of splitting is a temporary increase in storage utilization. For data being split, over the course of the weekly compaction cycle, there might be up to two copies of the original split range retained at a given time until the cycle has had a chance to shrink down the splits and discard the extra copies of data.

### Storage statistics

All data ingested into Spanner typically shows up in storage statistics within a matter of minutes. However, in certain cases, even though the data will be both accessible for reading (and durable through techniques like write-ahead logging), it will take longer to show up in storage utilization statistics, up to several days.

This happens because all ingested data (aside from a copy logged during commit for durability and recovery purposes) resides temporarily in memory before being written out to physical storage in the background. The amount of data that can and will reside in memory and the amount of time it will live in memory before being written to physical storage depends on the size of your compute and the size and performance of your workload.

## Create storage alerts

You can [create storage alerts](/spanner/docs/monitoring-cloud#create-alert) in the Cloud Monitoring console. We also provide a straightforward way to create a database storage alert directly from the [Google Cloud console](/spanner/docs/monitoring-console) . The **Create alerting policy** link in the chart (see screenshot) takes you to the create alert page in Cloud Monitoring console and automatically prefills the relevant fields.

## Recommendations for database storage utilization

We recommend keeping your total database storage below the [storage limit](/spanner/quotas#database_limits) . This ensures that Spanner has enough headroom to operate normally and perform routine maintenance on the data.

If you are approaching the limit, Spanner may prevent you from performing operations that put you over the limit, such as:

  - Restoring a database from a backup.
  - Modifying the database's schema (for example, adding an index).
  - Reducing the [compute capacity](/spanner/docs/compute-capacity) of your instance.

If you are over the storage limit, Spanner will attempt to operate normally, but you may experience degraded performance or failure due to resource pressure. If you do approach or exceed the recommended maximum, Google Cloud console displays a warning reading " **The instance has reached its maximum storage capacity and may experience degraded activity** " when displaying the affected instance.

You can also [create alerts in Cloud Monitoring](/spanner/docs/monitoring-cloud#create-alert) to notify you.

## Reduce database storage utilization

To reduce an instance's database storage utilization, you can:

  - [Add more compute capacity](/spanner/docs/create-manage-instances#change-compute-capacity) .
  - [Delete unused databases](/spanner/docs/create-manage-databases#delete-database) .
  - [Delete data](/spanner/docs/dml-tasks) from a database. Even though data deletion takes effect immediately from a data-visibility perspective, it might not affect the storage utilization metric until Spanner compacts the data, which usually happens within 12 hours for significant data deletions or within a week otherwise. Therefore, you might notice a delay between when data is deleted and when the changes appear in the metric.

In general, we recommend that you add [compute capacity](/spanner/docs/compute-capacity) to your instance as a starting point. After you add compute capacity, you can investigate and address the root causes of high storage utilization.

If you want to automate this process, you can create an application that monitors database storage utilization, then adds and removes compute capacity as needed, using the [`  UpdateInstance  `](/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstanceAdmin.UpdateInstance) method.

## What's next

  - Monitor your instance with the [Google Cloud console](/spanner/docs/monitoring-console) or the [Cloud Monitoring console](/spanner/docs/monitoring-cloud) .
  - [Create alerts for Spanner](/spanner/docs/monitoring-cloud#create-alert) .
  - Find out how to [change the compute capacity](/spanner/docs/create-manage-instances#change-compute-capacity) of a Spanner instance.
