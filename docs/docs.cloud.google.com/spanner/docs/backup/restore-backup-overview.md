You can restore a backup of a Spanner database into a new database. The restored database will have all the data and schema from the original database at the `  version_time  ` of the backup, including all database options that are set with the [`  ALTER DATABASE SET OPTIONS  `](/spanner/docs/reference/standard-sql/data-definition-language#alter-database) command. However, the following aren't included in the restored database:

  - Identity and Access Management (IAM) permissions (except for those inherited from the instance containing the restored database). You must apply appropriate IAM permissions after the restore completes.
  - Internal data of any change streams.
  - Time to live (TTL) defined by a row deletion policy. You must reconfigure these policies after the restore completes. For more information, see [Backups and TTL](/spanner/docs/ttl#backups) .
  - Split points you created when pre-splitting a database. For more information, see [Pre-splitting overview](/spanner/docs/pre-splitting-overview) .

If you need to restore from a backup in a different region or project for compliance or business continuity reasons, you can [copy the backup](/spanner/docs/backup/manage-backups#copy-backup) to an instance in a separate region or project, then restore from the copied backup.

You can use restore from a backup in the following ways:

  - In the [Google Cloud console](/spanner/docs/backup/gcp#restore)
  - Using the [Google Cloud CLI](/spanner/docs/backup/gcloud#restore)
  - Using the [client libraries](/spanner/docs/backup/libraries#restore)
  - Using the [REST](/spanner/docs/reference/rest/v1/projects.instances.backups#restore) or [RPC](/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.Backup) APIs

## How database restoration from a backup works

When you restore a Spanner database, you must specify a source backup and a new target database. You cannot restore to an existing database. The newly restored database must be in the same project as the backup and be in an instance with the same [instance configuration](/spanner/docs/instances#configuration) and same (or higher-tier) [Spanner edition](/spanner/docs/editions-overview) as the backup. For example, if a backup is in an instance configured `  us-west3  ` and uses the Enterprise edition, it can be restored to any instance in the project that is also configured `  us-west3  ` and uses the Enterprise edition. If you restore a backup in an Enterprise edition instance into a Standard edition instance, the restore might fail if the database uses Enterprise edition features. The [compute capacity](/spanner/docs/compute-capacity) of the instances doesn't need to be the same.

The restore process is designed for high-availability. You can restore the database provided that the majority quorum of the regions and zones in the *target instance* is available.

To restore a CMEK-enabled backup, both the key and key version must be available to Spanner. The restored database, by default, uses the same [encryption configurations](/spanner/docs/reference/rest/v1/projects.instances.databases/restore#RestoreDatabaseEncryptionConfig) as the backup. You can override this behavior by specifying a different encryption configuration when restoring the database. For more information, see [restore from a CMEK-enabled backup](/spanner/docs/use-cmek#restore) .

**Note:** Before restoring a database, make sure your instance is properly provisioned with enough storage and compute capacity to handle the additional storage and traffic associated with the restored database. If the target instance is not properly provisioned, restoring a database could adversely affect the performance of existing databases in the instance.

### Restore a backup to a different region or project

If you need to restore the backup to a different region or project, first, [copy the backup](/spanner/docs/backup/manage-backups#copy-backup) to the chosen region or project. Copied backups are restorable as soon as the copy finishes. You can restore the backup either in the destination instance (as long as it uses the edition as the source backup instance) or in any instance that has the same instance configuration and same (or higher-tier) edition as the destination instance. Before restoring, make sure that the destination instance has enough nodes or processing units provisioned to support the database size according to the 10 TB per node storage limit (that is, you need at least 2 nodes to restore a 20 TB backup). If you have copied the backup to a different project, and if you want to restore it there, make sure that your destination project has enough node quotas required for the restore. Restoring a copied backup works the same way as a normal restore.

## Restoration states

A restored database transitions through three [states](/spanner/docs/reference/rest/v1/projects.instances.databases#state) , tracked by two [long-running operations](/spanner/docs/manage-long-running-operations) .

  - `  CREATING  ` : Spanner begins restoring by creating a new database and mounting files from the backup. During this initial `  CREATING  ` state, the restored database is not yet ready for use. This state typically completes within one hour. Once the `  CREATING  ` state is complete, your database is ready to use.
    
    To track the progress of this state, you can query the [long-running restore operation](/spanner/docs/manage-long-running-operations) that Spanner makes available during this process. It returns a [`  RestoreDatabaseMetadata  `](/spanner/docs/reference/rest/v1/RestoreDatabaseMetadata) object.
    
    Note the following caveats regarding the `  CREATING  ` state:
    
      - If you are restoring to a different instance, the restore operation belongs to the instance containing the restored database, not the instance containing the backup.
      - Spanner won't allow you to delete the backup while it is being restored. You can delete it after the restore completes and the database enters the `  READY  ` state.
      - An instance can have at most ten databases in the `  CREATING  ` state due to restoration from backups. You won't be able to restore another backup to the instance until one of the ten restored databases transitions to the `  READY_OPTIMIZING  ` or `  READY  ` state.

  - `  READY_OPTIMIZING  ` : After Spanner mounts the backup, it starts to copy the backup data into the new database while optimizing its stored size. Your database is ready for use during this process. This phase of the restore usually takes a few hours to complete for databases less than 100TB in size.
    
    While you can use your database as usual during `  READY_OPTIMIZING  ` , the following caveats apply:
    
      - Read latencies might be slightly higher than usual.
      - [Storage metrics](/spanner/docs/storage-utilization) display the size of the new database, not the backup. Therefore, with the data transfer still in progress, Spanner storage metrics might show results that don't reflect the total size of all your data.
      - As with the `  CREATING  ` state, Spanner won't allow you to delete the mounted backup.
    
    Spanner makes another [long-running restore operation](/spanner/docs/manage-long-running-operations) available during this state, this time returning a [`  OptimizeRestoredDatabaseMetadata  `](/spanner/docs/reference/rest/v1/OptimizeRestoredDatabaseMetadata) metadata object.

  - `  READY  ` : Once the copy-and-optimize operation completes, the database transitions to the `  READY  ` state. The database is fully restored, and no longer references or requires the backup.

## Access control (IAM)

The role `  spanner.restoreAdmin  ` gives you permission to restore from a backup. For more information, see [Access control with IAM](/spanner/docs/backup#iam) .

The following roles also have access to Spanner restore operations:

  - `  spanner.admin  ` : has full access to restore. This role has complete access to all Spanner resources.
  - `  owner  ` : has full access to restore.
  - `  editor  ` : has full access to restore.
  - `  viewer  ` : has access to view restore and restore operations. This role can't create, update, delete, or copy a backup.

## Pricing

There is no charge for restoring from a backup.

## What's next

  - To restore a database from a backup, see [Restore from a backup](/spanner/docs/backup/restore-backups) .
