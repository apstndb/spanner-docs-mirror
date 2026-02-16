This document provides information about disaster recovery tools and techniques that help you protect your data from loss and downtime.

## Spanner disaster recovery features

Spanner is designed to be scalable and globally distributed. Spanner offers the following features to help ensure high data availability:

  - **Multi-regional configurations** : Spanner can maintain replicas of your data in separate zones within a single region or multiple regions to ensure data availability even if a zone or region fails.

  - **Database deletion protection** : You can protect against the accidental deletion of existing databases by users or service accounts that have the necessary Identity and Access Management (IAM) permissions to delete the database.

  - **Database back up and restore** : You can create backups of Spanner databases and restore them to help protect against operator and application errors. All backups are highly available, encrypted, and can be retained for up to a year from the time they are created. You can create full backups on demand or by using a backup schedule. You can only create incremental backups by using a backup schedule.

  - **Export and import** : You can export Spanner databases to Cloud Storage in either CSV or Avro format.

  - **Point-in-time recovery (PITR)** : Spanner point-in-time recovery (PITR) provides protection against logical data corruption, and accidental database deletion or writes. For example, if an operator inadvertently writes data or an application launch corrupts the database, you can recover the data from a point in time in the past (up to 7 days).

  - **Cross-region copy** : You can copy backups to different geographic regions to protect your data against regional failures or to meet your organization's compliance requirements.

### Choose between backing up or exporting your database

Compare backing up your databases to exporting your databases before deciding which one to use. For example, backups can have a maximum retention period of one year. You can consider exporting your database if you want a longer retention period than one year. The following table describes similarities and differences between using back up and restore versus using import and export:

Back up and restore

Import and export

Data consistency

Both backups and exported databases are transactionally and externally consistent.

Performance impact

Backups have no impact on an instance's performance. Spanner performs backups using dedicated jobs that don't draw upon an instance's server resources.

Export runs as a medium-priority task to minimize impact on database performance. For more information, see [task priority](/spanner/docs/cpu-utilization#task-priority) .

Storage format

Uses a proprietary, encrypted format designed for fast restore.

Supports both CSV and [Avro](https://en.wikipedia.org/wiki/Apache_Avro) file formats.

Portability

You [create](/spanner/docs/backup/create-backup) backups in the same instance as their source database.  
  
After a backup is created, you can [copy](/spanner/docs/backup/copy-backup) the backup to an instance in a different region or project if you need a cross-region or cross-project backup. You can then [restore](/spanner/docs/backup/restore-backup-overview) from a backup as a new database to any instance in the same project. The instance that you are restoring to should have the same instance configuration as the instance where the backup is stored.

Exported databases reside in [Cloud Storage](https://cloud.google.com/storage) and the data can be migrated to any system that supports CSV or Avro.

Retention

Backups can be retained for up to one year.

Exported databases are stored in Cloud Storage where, by default, they are retained until they are deleted. You can customize [lifecycle](/storage/docs/lifecycle) and [retention](/storage/docs/bucket-lock) policies.

Pricing

Backups are billed to your Spanner project based on the storage used per unit time. For more details, see the [Pricing](/spanner/docs/backup#pricing) section.

Billing for import and export is more complicated due to its use of [Cloud Storage](https://cloud.google.com/storage) and [Dataflow](https://cloud.google.com/dataflow) . For more information, see [Database export and import pricing](https://cloud.google.com/spanner/pricing#export-import-pricing) .

Restore time

Restore happens in two operations: restore and optimize. The restore operation offers fast time-to-first-byte because the database directly mounts the backup without copying the data. After the restore operation completes, the database is ready for use, though read latency might be slightly higher while it is optimizing. For more information, see [How restore works](/spanner/docs/backup/restore-backup-overview#how-restore-works) .

Import is slower. You need to wait for all the data to be written into the database.

## Disaster recovery techniques

Spanner offers disaster recovery techniques to protect your data against the following disasters:

  - **Zonal failure** : Spanner regional configuration provides automatic protection against zonal failures, ensuring that if one zone within a region fails, your application remains operational.
  - **Regional failure** : If your application requires even higher data availability and protection against regional failures, use [multi-region configurations](/spanner/docs/instance-configurations) that offer 99.999% availability.
  - **Major geographical disasters** : Use Spanner [cross-region backup copy](/spanner/docs/backup#how-backup-copy-works) to make backups available in different regions.
  - **Logical corruption** : Use the following disaster recovery techniques for different retention periods:
      - Set up [point-in-time recovery (PITR)](/spanner/docs/pitr) to restore your data at any point in time within the last 7 days.
      - Set up a [backup schedule](/spanner/docs/backup/create-manage-backup-schedules) that creates full backups or incremental backups at a frequency that meets your requirements. All backups can be retained for up to one year.
      - [Export your database](/spanner/docs/import-export-overview) to Cloud Storage where you can retain the data for compliance, analytics, or reporting.
  - **Accidental database deletion** : Use [database deletion protection](/spanner/docs/prevent-database-deletion) to prevent the accidental deletion of existing databases by users or service accounts that have the necessary IAM permissions.

## Optimize costs for your disaster recovery strategy

You can optimize Spanner disaster recovery costs in the following ways:

  - **Multi-region configurations** : Use multi-region configurations only for applications that require 99.999% availability. For applications that require read-only latencies, consider read replicas in other regions.
  - **Backup frequency** : Schedule backups only as frequently as needed to meet your requirements.
  - **Backup type** : Use incremental backup schedules to save storage costs.
  - **Backup retention** : Determine and set the backup retention period for the shortest time required to meet your recovery and compliance needs.
  - **Large data exports** : Consider using Spanner [Data Boost for Spanner](/spanner/docs/databoost/databoost-overview) for large data exports to offload compute resources from your instance and avoid negative impact on your transaction performance.

## Test your disaster recovery strategy

Consider testing and validating the following components of your disaster recovery plan:

  - Simulate the most likely events that can cause data loss for your organization.
  - Practice restoring your databases from a created backup. For more information on restoring your database, see [Restore overview](/spanner/docs/backup/restore-backup-overview) .
  - Assess the impact of your disaster recovery plan on storage utilization.
  - Assess the impact of backup processes on your application performance.
  - Simulate a zonal or regional failure to test your failover and recovery procedure.

## What's next

  - [Failure scenarios and resiliency with Spanner](https://services.google.com/fh/files/misc/spanner-failover-and-resiliency-whitepaper.pdf)
  - [Backups overview](/spanner/docs/backup)
  - [Create backups](/spanner/docs/backup/create-backups)
  - [Create and manage backup schedules](/spanner/docs/backup/create-manage-backup-schedules)
  - [PITR overview](/spanner/docs/pitr)
