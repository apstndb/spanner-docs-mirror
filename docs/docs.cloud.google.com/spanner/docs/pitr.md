Spanner point-in-time recovery (PITR) provides protection against accidental deletion or writes. For example, if an operator inadvertently writes data or an application rollout corrupts the database, with PITR you can recover the data from a point-in-time in the past (up to a maximum of seven days) seamlessly. If you need longer-term retention of data, you can use either [Backup and Restore](/spanner/docs/backup) or [Export and Import](/spanner/docs/export) .

By default, your database retains all versions of its data and schema for one hour. You can increase this time limit to as long as seven days through the [`  version_retention_period  `](/spanner/docs/reference/rest/v1/projects.instances.databases#Database.FIELDS.version_retention_period) option. For instructions, see [Set the retention period](/spanner/docs/use-pitr#set-period) . Spanner stores earlier versions of data at microsecond granularity and the database maintains an [`  earliest_version_time  `](/spanner/docs/reference/rest/v1/projects.instances.databases#Database.FIELDS.earliest_version_time) , which represents the earliest time in the past that you can recover earlier versions of the data.

**Note:** PITR provides additional insurance against logical data corruption, but does not protect you in case a user accidentally deletes the database. Make sure that you have other recovery options in place and that access to roles that include the [`  spanner.databases.drop  `](/spanner/docs/iam#databases) permission are set appropriately. For more information, see [Using IAM securely](/iam/docs/using-iam-securely) . You can also enable [database deletion protection](/spanner/docs/prevent-database-deletion) to prevent the accidental deletions of databases.

## Ways to recover data

There are two ways to recover data:

  - To **recover a portion of the database** , perform a [stale read](/spanner/docs/reads#perform-stale-read) specifying a query-condition and timestamp in the past, and then write the results back into the live database. This is typically used for surgical operations on a live database. For example, if you accidentally delete a particular row or incorrectly update a subset of data, you can recover it with this method. For instructions, see [recovering a portion of your database](/spanner/docs/use-pitr#recover-portion) .

  - To **recover the entire database** , [backup](/spanner/docs/backup) or [export](/spanner/docs/export) the database specifying a timestamp in the past and then restore or import it to a new database. This is typically used to recover from data corruption issues when you have to revert the database to a point-in-time before the corruption occurred. Note that backing up or exporting a database could take several hours and that you cannot restore or import to an existing database. For instructions, see [recovering the entire database](/spanner/docs/use-pitr#recover-entire) .

## Performance considerations

Databases with longer retention periods and, in particular, those that frequently overwrite data, use more system resources. This can affect how your database performs, especially if your instance is not provisioned with enough [compute capacity](/spanner/docs/compute-capacity) . If your database has a very high overwrite rate (for example, if your database is overwritten multiple times per day), you might consider increasing the retention period gradually and [monitoring the system](/spanner/docs/monitoring-cloud#storage) . Here are some things to be aware of:

  - **Increased storage utilization** . We recommend setting up [storage alerts](/spanner/docs/storage-utilization#alerts) to ensure that you don't exceed the [storage limit](/spanner/quotas#database_limits) . When you increase the retention period, keep in mind that storage usage will increase gradually as the database accumulates earlier versions of data. This is because the old data that would have expired under the previous retention period, is no longer expired. So, for example, if you increase the retention period from 3 days to 7 days, you need to wait for 4 days for database storage usage to stabilize. We also provide instructions for [estimating the storage increase](/spanner/docs/use-pitr#estimate-storage) .

  - **Increased CPU usage and latency** . Spanner uses additional computing resources to compact and maintain earlier versions of data. [Monitor your instance and database](/spanner/docs/monitoring-console#view-current-status) to ensure that latency and CPU utilization remain at acceptable levels.

  - **Increased time to perform schema updates** . An increased retention period means that [schema versions](/spanner/docs/schema-updates#schema_versions_created_during_schema_updates) must be retained for longer durations potentially causing schema updates to be [`  throttled  `](/spanner/docs/reference/rest/v1/UpdateDatabaseDdlMetadata#FIELDS.throttled) while waiting for server resources. Make sure that you are following [best practices for schema updates](/spanner/docs/schema-updates#best-practices) and staying within the [limits for schema updates](/spanner/docs/schema-updates#frequency) .

## Pricing

There is no additional charge for using PITR. However, if you increase the version retention period of your database from the default one hour, your database storage and compute capacity costs might increase. Your on-demand [backup](/spanner/docs/backup) cost is unaffected because only a single version of your database is stored. For more information, see the [Performance considerations](#performance) section. Before increasing a database's version retention period, you can [estimate the expected increase in database storage](/spanner/docs/use-pitr#estimate-storage) .

For general information about how Spanner is charged, see [Spanner pricing](https://cloud.google.com/spanner/pricing) .

## What's next

  - Learn more about how to [recover data with PITR](/spanner/docs/use-pitr) .
