This document provides an overview of Spanner backups and backup schedules.

Spanner lets you create full backups of databases on demand and full or incremental backups using a backup schedule. Full backups store the entire data of a database, whereas incremental backups contain only the data that has changed since a previous backup.

You can restore the backups when operator or application errors cause logical data corruption.

Backups are highly available, encrypted, and can be retained for up to a year from the time they are created. When you create a backup, the backup resides in the same instance, region, and project as its source database. If you need to restore the backup in a different region or project for compliance or business continuity reasons, you can copy the backup to an instance in a separate region or project.

Each backup has an associated `  createTime  ` and `  versionTime  ` . The `  createTime  ` is the timestamp when Spanner starts creating the backup. The `  versionTime  ` is the timestamp when the database contents are captured in the backup. The backup contains a consistent view of the database at the `  versionTime  ` .

For on-demand backups, the `  createTime  ` and `  versionTime  ` are the same by default. If needed, you can specify an older `  versionTime  ` when creating an on-demand backup if it's within the [version retention period](/spanner/docs/reference/rest/v1/projects.instances.databases#Database.FIELDS.version_retention_period) of the database.

For scheduled backups, the `  versionTime  ` is the time you choose when you create the backup schedule. Spanner starts creating the backup within four hours of the `  versionTime  ` , so the `  createTime  ` falls within this four-hour window. This is unlike on-demand backups, where Spanner starts creating the backup when it receives the request.

For example, suppose you create a backup schedule with a frequency of `  0 7 * * * UTC  ` or every day at 7:00 AM UTC. This means that for each backup, the `  versionTime  ` is set to 7:00 AM UTC and the `  createTime  ` is a timestamp within the four-hour window between 7:00 AM UTC and 11:00 AM UTC.

For more information about using `  createTime  ` and `  versionTime  ` using the API, see [Backup API reference](/spanner/docs/reference/rest/v1/projects.instances.backups) .

## Key features

  - **Data consistency** : Backups of a Spanner database are transactionally and [externally consistent](/spanner/docs/true-time-external-consistency) at the `  versionTime  ` of the backup.

  - **Replication** : Backups reside in the same instance as their source database and are replicated in the same geographic locations. For [regional instances](/spanner/docs/instance-configurations#regional-configurations) , the backup is stored in each of the three read-write zones. For [dual-region](/spanner/docs/instance-configurations#dual-region-configurations) and [multi-regional instances](/spanner/docs/instance-configurations#multi-region-configurations) , the backup is stored in all zones that contain either a read-write or read-only replica. If you need to store the backup of your database in a different region or project, you can copy the completed backup from the source instance to a destination instance located in a different region or project. For more information, see [copy a backup](/spanner/docs/backup/copy-backup) .

  - **Automatic expiration** : All backups have a user-specified expiration date which determines when it is automatically deleted. Spanner deletes expired backups asynchronously, so there might be a lag between when a backup is expired and when it's actually deleted.

## Backup creation

When you create a backup, the backup resides in the same instance, region, and project as its source database.

A backup contains the following information from the database at the `  versionTime  ` of the backup:

  - A full backup contains all of the data. An incremental backup contains only the data that has changed since a previous backup.
  - Schema information, including table names, fields, data types, secondary indexes, change streams, and the relationships between these entities.
  - All database options that are set with the [`  ALTER DATABASE SET OPTIONS  `](/spanner/docs/reference/standard-sql/data-definition-language#alter-database) command.

A Spanner backup does not include the following information:

  - Any modifications to the data or schema after the [`  versionTime  `](/spanner/docs/reference/rest/v1/projects.instances.backups) .
  - [Identity and Access Management (IAM)](/spanner/docs/iam) policies.
  - Change stream data records. Although the change streams schema is stored, the change stream data is meant to be streamed out and consumed near-simultaneously with the changes it describes.

To help ensure external consistency of the backup, Spanner pins the contents of the database at [`  versionTime  `](/spanner/docs/reference/rest/v1/projects.instances.backups) . This prevents the garbage collection system from removing the relevant data values for the duration of the backup operation. Then, every read-write and read-only zone in the instance begins copying the data in parallel. If any zone is temporarily unavailable, the backup is not complete until the zone comes back online. Backups are restorable as soon as the operation is complete. For multi-region instances, all read-write and read-only zones in all regions must complete their backup replicas before the backup is marked as restorable.

### Backup schedules

Spanner lets you schedule full or incremental backups for databases. An incremental backup contains only the data that has changed since a previous backup, whereas full backups store the entire contents of the database. You can specify the backup schedule type (full or incremental) and frequency for Spanner to create backups.

A full backup schedule can create a backup every 12 hours or more. An incremental backup schedule can create a backup every 4 hours or more.

Spanner offers incremental backups for your database through a backup schedule. You can't create an incremental backup on demand.

The backup creation begins within a four-hour window of the scheduled time. You can have a maximum of four backup schedules per database.

**Note:** Backup schedules for newly created databases take up to 24 hours to become active and start creating backups.

### Incremental backups

**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

Incremental backups form *chains* between full backups. The first backup created by an incremental backup schedule is a full backup. The consecutive backups created in the chain are incremental backups, each containing only the data that has changed since the previous backup in the chain.

Spanner permits up to 13 incremental backups per chain, in addition to the initial full backup. A chain is identified by the corresponding `  incrementalBackupChainId  ` value. When a chain reaches its maximum length, Spanner creates a new chain, beginning with the initial full backup.

In some scenarios, Spanner might create a new chain before the maximum chain length occurs. The following are a few of the scenarios:

  - The oldest full backup was taken 28 or more days ago.
  - The newest backup in the chain is deleted.
  - The incremental backup schedule is modified.

Here are some factors that can help you make a decision about using incremental backups:

  - **Restoration** : Restoring an incremental backup might take longer than restoring a full backup that contains the same data.

  - **Deletion** : If you delete a backup in a chain or it expires, Spanner might still keep the backup to support newer backups in the chain, if any exist. Spanner needs all the older backups in the chain to restore an incremental backup. To delete all the data in a chain of backups, including that of expired or deleted backups, delete all the backups in the chain.

  - **Retention** : Each backup schedule has the following terms that offer information about the schedule:
    
      - `  creation_interval  ` : represents the specified schedule frequency for the backup schedule.
      - `  retention_duration  ` : represents how long the backups created by the schedule are retained. For a given chain, the oldest full backup is retained past its original expiration date if it is needed to support newer backups in the chain. The total retention duration for the full backup is at most the lower of the following values:
          - `  retention_duration  ` + 28 days
          - `  retention_duration  ` + ( `  creation_interval  ` \*14)

  - **Backup copy** : When you copy an incremental backup, Spanner also copies all the older backups in the chain required to restore the copied backup. If the destination instance already contains a backup chain ending with an older backup that was copied from the same source chain, Spanner avoids creating redundant copies of existing backups. Instead, Spanner copies only the incremental backup and any older backups not present in the destination chain, and appends these backups to the existing chain. Spanner charges you based on the total storage used.
    
    For example, if you set up a daily incremental backup schedule and copy the latest backup each day, the destination instance maintains a backup chain that mirrors the source chain. Spanner doesn't duplicate previously copied backups within the chain during subsequent copy operations.
    
    While Spanner aims to avoid redundant copies, in rare situations, Spanner might need to copy all the older backups in the chain, even if previously copied backups already exist in the destination instance.

For more information about creating incremental backups, see [Create and manage backup schedules](/spanner/docs/backup/create-manage-backup-schedules) .

### Default backup schedules

When you create a new Spanner instance, you can specify if you want Spanner to create a default backup schedule for every new database in the instance. The default backup schedule creates a full backup every 24 hours. These backups have a retention period of 7 days. You can edit or delete the default backup schedule once it's created.

Default backup schedules are automatically enabled for all new instances. You can enable or disable default backup schedules in an instance when creating the instance or by editing the instance later.

You can enable default backup schedules for existing instances. However, default backup schedules don't apply to existing databases in the instance. Default backup schedules only apply to new databases in the instance.

The default backup schedule takes 24 hours to become active and start creating backups.

You must delete all backups in an instance before you delete an instance. If you are creating and deleting instances for testing purposes, you can delete the new instance within 24 hours to avoid manually deleting its backups.

For instructions on enabling or disabling default backup schedules, see [Edit the default backup schedule type](/spanner/docs/create-manage-instances#edit-default-backup-schedules) .

## Storage costs for full and incremental backups

Each Spanner backup has the following fields that offer information about storage consumption:

  - `  exclusiveSizeBytes  ` : shows the number of bytes required by the backup. This size represents the billable size of the backup.
  - `  freeableSizeBytes  ` : shows the number of bytes that are released if you delete the backup.
  - `  oldestVersionTime  ` : shows the `  versionTime  ` of the oldest full backup in the chain, even if that backup has expired. You can use this field to understand which data is being stored.

Incremental backups can save you storage costs. An incremental backup might have a significantly smaller `  exclusiveSizeBytes  ` field than a full backup as the incremental backup only needs to store the changes since the previous backup in the chain. Adding this field value for each backup in the chain reflects the total number of bytes used by the backups in the chain.

An incremental backup is dependent on all older backups in the same chain for restoration. This means that if a newer incremental backup exists, the data of all older backups in the chain cannot be deleted from the system and the `  freeableSizeBytes  ` field for all older backups in the same chain is zero.

Consider that you created a full backup schedule and an incremental backup schedule for a database that has a size of 100 GB and increases by 10 GB every day. The following table shows possible storage costs for these backup schedules:

<table>
<thead>
<tr class="header">
<th>Day</th>
<th>Full schedule backup size</th>
<th>Incremental schedule backup size</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>100 GB</td>
<td>100 GB</td>
</tr>
<tr class="even">
<td>2</td>
<td>110 GB</td>
<td>10 GB</td>
</tr>
<tr class="odd">
<td>3</td>
<td>120 GB</td>
<td>10 GB</td>
</tr>
<tr class="even">
<td>4</td>
<td>130 GB</td>
<td>10 GB</td>
</tr>
<tr class="odd">
<td>5</td>
<td>140 GB</td>
<td>10 GB</td>
</tr>
</tbody>
</table>

Over 5 days, the full backup schedule uses 600 GB of storage, while the incremental backup schedule uses about 140 GB of storage. For an incremental backup schedule, the full backup size is the sum of the sizes of all backups in the chain, up to that backup and is reflected in the `  sizeBytes  ` field.

### Comparing backup storage to database storage

The storage size of a backup can be smaller or larger than the storage size of the source database when the backup was created.

A backup's storage can be smaller than the database's storage because the backup contains only one version of the data, whereas the live database can contain multiple versions of data due to ongoing operations. Differences in data format and compression can also result in a smaller backup size.

Conversely, the storage of a backup can be larger than the database's storage, depending on the situation of the database and the time when you take the backup.

For similar reasons, when you restore a database from a backup, the backup's storage can be larger than the database's storage. This might occur if a large set of data was deleted and compacted after the backup was taken. Therefore, the backup size depends on when the backup was taken and the subsequent operations on the database.

There is no formula to predict the exact ratio of a backup's size relative to the live database. For a database with a high rate of writes (a *hot database* ), the backup is likely smaller than the live database. However, in some rare situations, the backup size can be larger.

## How backup copying works

Spanner lets you copy a backup of your Spanner database from one instance to another instance in a different region or project, to provide additional data protection and compliance capabilities.

You can't copy a backup if the target or source Google Cloud region is down. To protect your data if there's a region outage, you need to periodically copy your backups to a location outside of the affected region.

The copied backup has the same [key features](/spanner/docs/backup#key-features) as the original backup. Additionally, you can [restore](/spanner/docs/backup/restore-backups#cross-region-project) a copied backup in the same instance as the copied backup to support cross-region and cross-project backup and restore use cases.

## Where Spanner backups are stored

Backups are resources in Spanner. Each [backup resource](/spanner/docs/reference/rest/v1/projects.instances.backups) is organized under the same instance as its source database in the [resource hierarchy](/resource-manager/docs/cloud-platform-resource-hierarchy) and has a resource path that uses the following format:

``` text
projects/PROJECT_ID/instances/INSTANCE_ID/backups/BACKUP_NAME
```

Replace the following:

  - `  PROJECT_ID  ` : the project ID.
  - `  INSTANCE_ID  ` : the instance ID.
  - `  BACKUP_NAME  ` : the backup name.

A backup continues to exist even after its source database has been deleted, but it can't outlive its parent instance. To prevent accidental deletion of backups, you can't delete a Spanner instance if it has backups. If you want to delete the instance, we recommend restoring the backup, then [exporting the restored database](/spanner/docs/export) , before deleting the backup and the instance.

## Encryption

Spanner backups, like databases, are encrypted by either Google-owned and Google-managed encryption keys or by [customer-managed encryption keys (CMEK)](/spanner/docs/cmek) . By default, a backup uses the same [encryption configuration](/spanner/docs/reference/rest/v1/projects.instances.backups/create#CreateBackupEncryptionConfig) as its database, but you can override this behavior by specifying a different encryption configuration when creating the backup. If the backup is CMEK-enabled, it is encrypted using the primary version of the KMS key at the time of backup creation. After the backup is created, its key and key version can't be modified, even if the KMS key is rotated. For more information, see [create a CMEK-enabled backup](/spanner/docs/use-cmek#backup) .

A copied backup uses the same encryption configuration, either Google-owned and Google-managed encryption keys or [customer-managed encryption keys (CMEK)](/spanner/docs/cmek) , as its source backup encryption. You can override this behavior by specifying a different encryption configuration when copying the backup. If you want the copied backup to be encrypted with CMEK when copying across regions, specify the [Cloud Key Management Service key](/kms/docs/use-keys-google-cloud) key corresponding to the destination region.

You can specify the encryption configuration when creating or modifying the backup schedule. If you want the backup schedule to create backups that are encrypted by CMEK keys, you need to specify the key path.

## Performance

This section describes optimal backup performance in Spanner.

### Performance when backing up

When performing a backup, Spanner creates a backup job to copy data directly from the database to backup storage, and sizes this job based on the size of the database. This backup job does not use CPU resources allocated to the database's instance so it does not affect the instance's performance. Moreover, compute load on the database's instance does not affect the speed of the backup operation. To track the progress and completion of a backup operation, see [Show backup progress](/spanner/docs/backup/manage-backups#check-operation-progress) .

Generally, most backups take 1-4 hours. Some backups might take longer due to their size or because there is internal queueing for resources. If a backup is taking longer than usual when no other factors have changed, it might be due to a delay in scheduling the backup task in a zone. This can sometimes take up to 30 minutes. We recommend that you do not cancel and restart the backup, as it's likely you'll encounter the same scheduling delay with the new backup operation.

### Performance when copying a backup

The time it takes to copy a backup depends on factors such as the size of the source backup and the destination region chosen for the copied backup. Generally, most copies complete in 1-4 hours. Some copies might take longer depending on the backup size and destination region. Copying a backup doesn't have any performance implications on the source instance or database. You can make multiple, concurrent copies of the source backup to instances in different regions without any performance implications.

When you copy an incremental backup, Spanner also copies all of the older backups in the chain required to restore the copied backup. To improve performance, Spanner copies all the backups simultaneously instead of sequentially. Spanner also attempts to avoid copying older backups in the same chain when possible. For more information, see [Incremental backups](#incremental-backups) .

## Delete a backup

When you delete a backup in an incremental backup chain, you can't recover storage if there are newer backups present in the chain. The newer backups depend on the data present in the deleted backup and the older backups in the chain. Spanner retains the data and only releases the storage when all the newer incremental backups expire. The `  freeableSizeBytes  ` field shows how much storage space you can regain if you delete the backup.

## Pricing

You are billed based on the amount of storage used by your backups per unit time. Billing begins after the backup operation is complete and continues until the backup is deleted. A completed backup is billed for a minimum of 24 hours. If you create a backup and then delete it soon after it finishes, you are still billed for 24 hours.

A copy of a backup is subject to the [same storage costs](https://cloud.google.com/spanner/pricing#backup-storage) as an original backup. If you create a copy between two instances that occupy different regions, then [outbound data transfer costs](https://cloud.google.com/spanner/pricing#network) apply.

For example, if you copy your database from the source multi-region instance configuration `  nam7  ` to the destination multi-region instance configuration `  nam-eur-asia3  ` , the following charges apply:

  - No charge for overlapping `  us-central1  ` region
  - No charge for witness `  us-central2  ` region
  - *Inter-continental data transfer* charge apply twice: once for each new continent (Europe and Asia)
  - *Data transfer between regions within the same continent* charge apply once for `  us-east1  `
  - *Data transfer between regions within the same continent* charge apply once in Europe

Spanner optimizes the copying process to minimize the number of cross-region transfers. This helps to minimize the data transfer costs while providing a fast copy backup experience.

Backups are stored and billed separately. Backup storage does not affect [billing for database storage](https://cloud.google.com/spanner/pricing#storage) or [database storage limits](/spanner/quotas#database_limits) . For more information, also see [Storage utilization metrics](/spanner/docs/storage-utilization#metrics) .

For more complete information on backup costs, see [Spanner pricing](https://cloud.google.com/spanner/pricing#backup-storage) .

## What's next

  - To create backups, see [Create backups](/spanner/docs/backup/create-backups) .

  - To manage backups, see [Manage backups](/spanner/docs/backup/manage-backups) .
