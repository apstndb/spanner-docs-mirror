---
name: documents/docs.cloud.google.com/spanner/docs/iam
uri: https://docs.cloud.google.com/spanner/docs/iam
title: IAM overview
description: Control access to your Spanner resources using IAM, granting granular permissions to users and service accounts.
data_source: docs.cloud.google.com
---

[Identity and Access Management](https://docs.cloud.google.com/iam/docs/overview) (IAM) lets you control user and group access to Spanner resources at the project, Spanner instance, Spanner database, backup, and backup schedule levels. For example, you can specify that a user has full control of a specific database in a specific instance in your project, but cannot create, modify, or delete any instances in your project. Using access control with IAM lets you grant a permission to a user or group without having to modify each Spanner instance or database permission individually.

This document focuses on the IAM *permissions* relevant to Spanner and the IAM *roles* that grant those permissions. For a detailed description of IAM and its features, see the [IAM](https://docs.cloud.google.com/iam/docs/overview) developer's guide. In particular, see the [Managing IAM policies](https://docs.cloud.google.com/iam/docs/granting-changing-revoking-access) section.

## Permissions

Permissions allow users to perform specific actions on Spanner resources. For example, the `spanner.databases.read` permission allows a user to read from a database using Spanner's read API, while `spanner.databases.select` allows a user to execute a SQL select statement on a database. You don't directly give users permissions; instead, you grant them [predefined roles](https://docs.cloud.google.com/spanner/docs/iam#roles) or [custom roles](https://docs.cloud.google.com/spanner/docs/iam#custom-roles) , which have one or more permissions bundled within them.

The following tables list the IAM permissions that are associated with Spanner.

### Instance configurations

The following permissions apply to Spanner instance configurations. For more information, see the instance configuration references for [REST](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs) and [RPC](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstanceConfig) APIs.

| Instance configuration permission name | Description                              |
|----------------------------------------|------------------------------------------|
| `spanner.instanceConfigs.create`       | Create a custom instance configuration.  |
| `spanner.instanceConfigs.delete`       | Delete a custom instance configuration.  |
| `spanner.instanceConfigs.get`          | Get an instance configuration.           |
| `spanner.instanceConfigs.list`         | List the set of instance configurations. |
| `spanner.instanceConfigs.update`       | Update a custom instance configuration.  |

### Instance configuration operations

The following permissions apply to Spanner instance configuration operations. For more information, see the following API references:

- REST: [`projects.instanceConfigOperations`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigOperations) and [`projects.instanceConfigs.operations`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs.operations)
- RPC: [`ListInstanceConfigOperations`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstanceAdmin.ListInstanceConfigOperations) and [`Operations`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.longrunning#google.longrunning.Operations)

| Instance configuration operation permission name | Description                                 |
|--------------------------------------------------|---------------------------------------------|
| `spanner.instanceConfigOperations.cancel`        | Cancel an instance configuration operation. |
| `spanner.instanceConfigOperations.delete`        | Delete an instance configuration operation. |
| `spanner.instanceConfigOperations.get`           | Get an instance configuration operation.    |
| `spanner.instanceConfigOperations.list`          | List instance configuration operations.     |

### Instances

The following permissions apply to Spanner instances. For more information, see the instance references for [REST](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances) and [RPC](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.Instance) APIs.

| Instance permission name              | Description                                                                     |
|---------------------------------------|---------------------------------------------------------------------------------|
| `spanner.instances.create`            | Create an instance.                                                             |
| `spanner.instances.createTagBinding`  | Attach a [tag](https://docs.cloud.google.com/spanner/docs/tags) to an instance. |
| `spanner.instances.delete`            | Delete an instance.                                                             |
| `spanner.instances.deleteTagBinding`  | Detach a tag from an instance.                                                  |
| `spanner.instances.get`               | Get the configuration of a specific instance.                                   |
| `spanner.instances.getIamPolicy`      | Get an instance's IAM Policy.                                                   |
| `spanner.instances.list`              | List instances.                                                                 |
| `spanner.instances.listEffectiveTags` | List the tags that are attached to or inherited by an instance.                 |
| `spanner.instances.listTagBindings`   | List the tags that are attached directly to an instance.                        |
| `spanner.instances.setIamPolicy`      | Set an instance's IAM Policy.                                                   |
| `spanner.instances.update`            | Update an instance.                                                             |

### Instance operations

The following permissions apply to Spanner instance operations. For more information, see the instance references for [REST](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.operations) and [RPC](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.longrunning#google.longrunning.Operations) APIs.

| Instance operation permission name  | Description                        |
|-------------------------------------|------------------------------------|
| `spanner.instanceOperations.cancel` | Cancel an instance operation.      |
| `spanner.instanceOperations.delete` | Delete an instance operation.      |
| `spanner.instanceOperations.get`    | Get a specific instance operation. |
| `spanner.instanceOperations.list`   | List instance operations.          |

### Instance partitions

The following permissions apply to Spanner instance partitions. For more information, see the instance partition references for [REST](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions) and [RPC](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstancePartition) APIs.

| Instance partition permission name  | Description                                             |
|-------------------------------------|---------------------------------------------------------|
| `spanner.instancePartitions.create` | Create an instance partition.                           |
| `spanner.instancePartitions.delete` | Delete an instance partition.                           |
| `spanner.instancePartitions.get`    | Get the configuration of a specific instance partition. |
| `spanner.instancePartitions.list`   | List instance partitions.                               |
| `spanner.instancePartitions.update` | Update an instance partition.                           |

### Instance partition operations

The following permissions apply to Spanner instance partition operations. For more information, see the following API references:

- REST: [`projects.instances.instancePartitionOperations`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations) and [`projects.instances.instancePartitions.operations`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions.operations)
- RPC: [`ListInstancePartitionOperations`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstanceAdmin.ListInstancePartitionOperations) and [`Operations`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.longrunning#google.longrunning.Operations)

| Instance partition operation permission name | Description                                  |
|----------------------------------------------|----------------------------------------------|
| `spanner.instancePartitionOperations.cancel` | Cancel an instance partition operation.      |
| `spanner.instancePartitionOperations.delete` | Delete an instance partition operation.      |
| `spanner.instancePartitionOperations.get`    | Get a specific instance partition operation. |
| `spanner.instancePartitionOperations.list`   | List instance partition operations.          |

### Databases

The following permissions apply to Spanner databases. For more information, see the database references for [REST](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases) and [RPC](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.Database) APIs.

| Database permission name                                | Description                                                                                                                                                                                                                                                                                                                      |
|---------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `spanner.databases.adapt`                               | Lets the [Spanner Adapter API](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.adapter.v1) interact directly with Spanner. Also requires `spanner.sessions.create` to create an adapter session, and `spanner.databases.select` or `spanner.databases.write` to read or modify data through the adapter. |
| `spanner.databases.addSplitPoints`                      | Add [split points](https://docs.cloud.google.com/spanner/docs/create-manage-split-points) to the tables and indexes of a database.                                                                                                                                                                                               |
| `spanner.databases.beginOrRollbackReadWriteTransaction` | Begin or roll back a [read-write transaction](https://docs.cloud.google.com/spanner/docs/transactions#read-write_transactions) on a Spanner database.                                                                                                                                                                            |
| `spanner.databases.beginPartitionedDmlTransaction`      | Begin a [Partitioned DML](https://docs.cloud.google.com/spanner/docs/dml-partitioned) transaction on a Spanner database. Running the `UPDATE` or `DELETE` statement in the transaction also requires `spanner.databases.write` .                                                                                                 |
| `spanner.databases.beginReadOnlyTransaction`            | Begin a [read-only transaction](https://docs.cloud.google.com/spanner/docs/transactions#read-only_transactions) on a Spanner database.                                                                                                                                                                                           |
| `spanner.databases.changequorum`                        | Change the [quorum](https://docs.cloud.google.com/spanner/docs/change-dual-region-quorum) of a database in a dual-region instance configuration.                                                                                                                                                                                 |
| `spanner.databases.create`                              | Create a database.                                                                                                                                                                                                                                                                                                               |
| `spanner.databases.createBackup`                        | Create a backup from the database. Also requires `spanner.backups.create` to create the backup resource.                                                                                                                                                                                                                         |
| `spanner.databases.drop`                                | Drop a database.                                                                                                                                                                                                                                                                                                                 |
| `spanner.databases.get`                                 | Get a database's metadata.                                                                                                                                                                                                                                                                                                       |
| `spanner.databases.getDdl`                              | Get a database's schema.                                                                                                                                                                                                                                                                                                         |
| `spanner.databases.getIamPolicy`                        | Get a database's IAM policy.                                                                                                                                                                                                                                                                                                     |
| `spanner.databases.list`                                | List databases.                                                                                                                                                                                                                                                                                                                  |
| `spanner.databases.partitionQuery`                      | Partition a SQL query so that you can [read data in parallel](https://docs.cloud.google.com/spanner/docs/reads#read_data_in_parallel) .                                                                                                                                                                                          |
| `spanner.databases.partitionRead`                       | Partition a read request so that you can read data in parallel.                                                                                                                                                                                                                                                                  |
| `spanner.databases.read`                                | Read from a database using the read API.                                                                                                                                                                                                                                                                                         |
| `spanner.databases.runGraphAlgorithms`                  | Run [graph algorithms](https://docs.cloud.google.com/spanner/docs/graph/graph-algorithms-overview) on a Spanner Graph database.                                                                                                                                                                                                  |
| `spanner.databases.select`                              | Execute a SQL select statement on a database.                                                                                                                                                                                                                                                                                    |
| `spanner.databases.setIamPolicy`                        | Set a database's IAM policy.                                                                                                                                                                                                                                                                                                     |
| `spanner.databases.update`                              | Update a database's metadata.                                                                                                                                                                                                                                                                                                    |
| `spanner.databases.updateDdl`                           | Update a database's schema.                                                                                                                                                                                                                                                                                                      |
| `spanner.databases.useDataBoost`                        | Use the compute resources of [Spanner Data Boost](https://docs.cloud.google.com/spanner/docs/databoost/databoost-overview) to run partitioned queries and reads.                                                                                                                                                                 |
| `spanner.databases.useRoleBasedAccess`                  | Use [fine-grained access control](https://docs.cloud.google.com/spanner/docs/fgac-about) .                                                                                                                                                                                                                                       |
| `spanner.databases.write`                               | Write into a database.                                                                                                                                                                                                                                                                                                           |

### Database roles

The following permissions apply to Spanner database roles. For more information, see the database references for [REST](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles) and [RPC](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseRole) APIs.

| Database role permission name | Description                    |
|-------------------------------|--------------------------------|
| `spanner.databaseRoles.list`  | List database roles.           |
| `spanner.databaseRoles.use`   | Use a specified database role. |

### Database operations

The following permissions apply to Spanner database operations. For more information, see the following API references:

- REST: [`projects.instances.databaseOperations`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databaseOperations) and [`projects.instances.databases.operations`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.operations)
- RPC: [`ListDatabaseOperations`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.ListDatabaseOperations) and [`Operations`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.longrunning#google.longrunning.Operations)

| Database operation permission name  | Description                                    |
|-------------------------------------|------------------------------------------------|
| `spanner.databaseOperations.cancel` | Cancel a database operation.                   |
| `spanner.databaseOperations.get`    | Get a specific database operation.             |
| `spanner.databaseOperations.list`   | List database and restore database operations. |

### Backups

The following permissions apply to Spanner backups. For more information, see the backups references for [REST](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups) and [RPC](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.Backup) APIs.

| Backup permission name            | Description                                                                                                                      |
|-----------------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| `spanner.backups.copy`            | Copy a backup. Also requires `spanner.backups.create` on the destination instance.                                               |
| `spanner.backups.create`          | Create a backup. Also requires `spanner.databases.createBackup` on the source database.                                          |
| `spanner.backups.delete`          | Delete a backup.                                                                                                                 |
| `spanner.backups.get`             | Get a backup.                                                                                                                    |
| `spanner.backups.getIamPolicy`    | Get a backup's IAM policy.                                                                                                       |
| `spanner.backups.list`            | List backups.                                                                                                                    |
| `spanner.backups.restoreDatabase` | Restore database from a backup. Also requires `spanner.databases.create` to create the restored database on the target instance. |
| `spanner.backups.setIamPolicy`    | Set a backup's IAM policy.                                                                                                       |
| `spanner.backups.update`          | Update a backup.                                                                                                                 |

### Backup operations

The following permissions apply to Spanner backup operations. For more information, see the database references for [REST](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups.operations) and [RPC](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1) APIs.

| Backup operation permission name  | Description                      |
|-----------------------------------|----------------------------------|
| `spanner.backupOperations.cancel` | Cancel a backup operation.       |
| `spanner.backupOperations.get`    | Get a specific backup operation. |
| `spanner.backupOperations.list`   | List backup operations.          |

### Backup schedules

The following permissions apply to Spanner backup schedules. For more information, see the backup schedule references for the [REST](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules) and [RPC](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.BackupSchedule) APIs.

| Backup schedule permission name        | Description                                                                                      |
|----------------------------------------|--------------------------------------------------------------------------------------------------|
| `spanner.backupSchedules.create`       | Create a backup schedule. Also requires `spanner.databases.createBackup` on the source database. |
| `spanner.backupSchedules.delete`       | Delete a backup schedule.                                                                        |
| `spanner.backupSchedules.get`          | Get a backup schedule.                                                                           |
| `spanner.backupSchedules.getIamPolicy` | Get a backup schedule's IAM policy.                                                              |
| `spanner.backupSchedules.list`         | List backup schedules.                                                                           |
| `spanner.backupSchedules.setIamPolicy` | Set a backup schedule's IAM policy.                                                              |
| `spanner.backupSchedules.update`       | Update a backup schedule.                                                                        |

### Sessions

The following permissions apply to Spanner sessions. For more information, see the database references for [REST](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions) and [RPC](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Session) APIs.

> **Note:** The Spanner client libraries and the Google Cloud console create sessions for you, so reading or writing data requires `spanner.sessions.create` even if you don't manage sessions directly. Running `CALL cancel_query` to cancel a query also requires `spanner.sessions.delete` . You can't list or delete multiplexed sessions. For more information, see [Sessions](https://docs.cloud.google.com/spanner/docs/sessions) .

| Session permission name   | Description       |
|---------------------------|-------------------|
| `spanner.sessions.create` | Create a session. |
| `spanner.sessions.delete` | Delete a session. |
| `spanner.sessions.get`    | Get a session.    |
| `spanner.sessions.list`   | List sessions.    |

## Predefined roles

A predefined role is a bundle of one or more [permissions](https://docs.cloud.google.com/spanner/docs/iam#permissions) . For example, the predefined role `roles/spanner.databaseUser` includes permissions such as `spanner.databases.read` and `spanner.databases.write` .

The following table lists the access control with IAM predefined roles, including a list of the permissions associated with each role:

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Role</th>
<th>Permissions</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Cloud Spanner Admin
<p>( <code>roles/ spanner.admin</code> )</p>
<p>Has complete access to all Spanner resources in a Google Cloud project. A principal with this role can:</p>
<ul>
<li>Grant and revoke permissions to other principals for all Spanner resources in the project.</li>
<li>Allocate and delete chargeable Spanner resources.</li>
<li>Issue get/list/modify operations on Cloud Spanner resources.</li>
<li>Read from and write to all Cloud Spanner databases in the project.</li>
<li>Fetch project metadata.</li>
</ul>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td><p><code>cloudkms.keyHandles.*</code></p>
<ul>
<li><code>cloudkms.keyHandles.create</code></li>
<li><code>cloudkms.keyHandles.get</code></li>
<li><code>cloudkms.keyHandles.list</code></li>
</ul>
<p><code>cloudkms.operations.get</code></p>
<p><code>cloudkms. projects. showEffectiveAutokeyConfig</code></p>
<p><code>monitoring.timeSeries.*</code></p>
<ul>
<li><code>monitoring.timeSeries.create</code></li>
<li><code>monitoring.timeSeries.list</code></li>
</ul>
<p><code>resourcemanager.projects.get</code></p>
<p><code>resourcemanager.projects.list</code></p>
<p><code>spanner.*</code></p>
<ul>
<li><code>spanner. backupOperations. cancel</code></li>
<li><code>spanner.backupOperations.get</code></li>
<li><code>spanner.backupOperations.list</code></li>
<li><code>spanner.backupSchedules.create</code></li>
<li><code>spanner.backupSchedules.delete</code></li>
<li><code>spanner.backupSchedules.get</code></li>
<li><code>spanner. backupSchedules. getIamPolicy</code></li>
<li><code>spanner.backupSchedules.list</code></li>
<li><code>spanner. backupSchedules. setIamPolicy</code></li>
<li><code>spanner.backupSchedules.update</code></li>
<li><code>spanner.backups.copy</code></li>
<li><code>spanner.backups.create</code></li>
<li><code>spanner.backups.delete</code></li>
<li><code>spanner.backups.get</code></li>
<li><code>spanner.backups.getIamPolicy</code></li>
<li><code>spanner.backups.list</code></li>
<li><code>spanner. backups. restoreDatabase</code></li>
<li><code>spanner.backups.setIamPolicy</code></li>
<li><code>spanner.backups.update</code></li>
<li><code>spanner. databaseOperations. cancel</code></li>
<li><code>spanner.databaseOperations.get</code></li>
<li><code>spanner. databaseOperations. list</code></li>
<li><code>spanner.databaseRoles.list</code></li>
<li><code>spanner.databases.adapt</code></li>
<li><code>spanner. databases. addSplitPoints</code></li>
<li><code>spanner. databases. beginOrRollbackReadWriteTransaction</code></li>
<li><code>spanner. databases. beginPartitionedDmlTransaction</code></li>
<li><code>spanner. databases. beginReadOnlyTransaction</code></li>
<li><code>spanner.databases.changequorum</code></li>
<li><code>spanner.databases.create</code></li>
<li><code>spanner.databases.createBackup</code></li>
<li><code>spanner.databases.drop</code></li>
<li><code>spanner.databases.get</code></li>
<li><code>spanner.databases.getDdl</code></li>
<li><code>spanner.databases.getIamPolicy</code></li>
<li><code>spanner.databases.list</code></li>
<li><code>spanner. databases. partitionQuery</code></li>
<li><code>spanner. databases. partitionRead</code></li>
<li><code>spanner.databases.read</code></li>
<li><code>spanner. databases. runGraphAlgorithms</code></li>
<li><code>spanner.databases.select</code></li>
<li><code>spanner.databases.setIamPolicy</code></li>
<li><code>spanner.databases.update</code></li>
<li><code>spanner.databases.updateDdl</code></li>
<li><code>spanner.databases.useDataBoost</code></li>
<li><code>spanner. databases. useRoleBasedAccess</code></li>
<li><code>spanner.databases.write</code></li>
<li><code>spanner. instanceConfigOperations. cancel</code></li>
<li><code>spanner. instanceConfigOperations. delete</code></li>
<li><code>spanner. instanceConfigOperations. get</code></li>
<li><code>spanner. instanceConfigOperations. list</code></li>
<li><code>spanner.instanceConfigs.create</code></li>
<li><code>spanner.instanceConfigs.delete</code></li>
<li><code>spanner.instanceConfigs.get</code></li>
<li><code>spanner.instanceConfigs.list</code></li>
<li><code>spanner.instanceConfigs.update</code></li>
<li><code>spanner. instanceOperations. cancel</code></li>
<li><code>spanner. instanceOperations. delete</code></li>
<li><code>spanner.instanceOperations.get</code></li>
<li><code>spanner. instanceOperations. list</code></li>
<li><code>spanner. instancePartitionOperations. cancel</code></li>
<li><code>spanner. instancePartitionOperations. delete</code></li>
<li><code>spanner. instancePartitionOperations. get</code></li>
<li><code>spanner. instancePartitionOperations. list</code></li>
<li><code>spanner. instancePartitions. create</code></li>
<li><code>spanner. instancePartitions. delete</code></li>
<li><code>spanner.instancePartitions.get</code></li>
<li><code>spanner. instancePartitions. list</code></li>
<li><code>spanner. instancePartitions. update</code></li>
<li><code>spanner.instances.create</code></li>
<li><code>spanner. instances. createTagBinding</code></li>
<li><code>spanner.instances.delete</code></li>
<li><code>spanner. instances. deleteTagBinding</code></li>
<li><code>spanner.instances.get</code></li>
<li><code>spanner.instances.getIamPolicy</code></li>
<li><code>spanner.instances.list</code></li>
<li><code>spanner. instances. listEffectiveTags</code></li>
<li><code>spanner. instances. listTagBindings</code></li>
<li><code>spanner.instances.setIamPolicy</code></li>
<li><code>spanner.instances.update</code></li>
<li><code>spanner.sessions.create</code></li>
<li><code>spanner.sessions.delete</code></li>
<li><code>spanner.sessions.get</code></li>
<li><code>spanner.sessions.list</code></li>
</ul></td>
</tr>
<tr class="even">
<td>Cloud Spanner Editor
<p>( <code>roles/ spanner.editor</code> )</p>
<p>Editor role for Cloud Spanner</p></td>
<td><p><code>monitoring.timeSeries.list</code></p>
<p><code>resourcemanager.projects.get</code></p>
<p><code>resourcemanager.projects.list</code></p>
<p><code>spanner.backupOperations.*</code></p>
<ul>
<li><code>spanner. backupOperations. cancel</code></li>
<li><code>spanner.backupOperations.get</code></li>
<li><code>spanner.backupOperations.list</code></li>
</ul>
<p><code>spanner.backupSchedules.create</code></p>
<p><code>spanner.backupSchedules.delete</code></p>
<p><code>spanner.backupSchedules.get</code></p>
<p><code>spanner. backupSchedules. getIamPolicy</code></p>
<p><code>spanner.backupSchedules.list</code></p>
<p><code>spanner.backupSchedules.update</code></p>
<p><code>spanner.backups.copy</code></p>
<p><code>spanner.backups.create</code></p>
<p><code>spanner.backups.delete</code></p>
<p><code>spanner.backups.get</code></p>
<p><code>spanner.backups.getIamPolicy</code></p>
<p><code>spanner.backups.list</code></p>
<p><code>spanner. backups. restoreDatabase</code></p>
<p><code>spanner.backups.update</code></p>
<p><code>spanner.databaseOperations.*</code></p>
<ul>
<li><code>spanner. databaseOperations. cancel</code></li>
<li><code>spanner.databaseOperations.get</code></li>
<li><code>spanner. databaseOperations. list</code></li>
</ul>
<p><code>spanner.databaseRoles.list</code></p>
<p><code>spanner.databases.adapt</code></p>
<p><code>spanner. databases. addSplitPoints</code></p>
<p><code>spanner. databases. beginOrRollbackReadWriteTransaction</code></p>
<p><code>spanner. databases. beginPartitionedDmlTransaction</code></p>
<p><code>spanner. databases. beginReadOnlyTransaction</code></p>
<p><code>spanner.databases.changequorum</code></p>
<p><code>spanner.databases.create</code></p>
<p><code>spanner.databases.createBackup</code></p>
<p><code>spanner.databases.drop</code></p>
<p><code>spanner.databases.get</code></p>
<p><code>spanner.databases.getDdl</code></p>
<p><code>spanner.databases.getIamPolicy</code></p>
<p><code>spanner.databases.list</code></p>
<p><code>spanner. databases. partitionQuery</code></p>
<p><code>spanner. databases. partitionRead</code></p>
<p><code>spanner.databases.read</code></p>
<p><code>spanner. databases. runGraphAlgorithms</code></p>
<p><code>spanner.databases.select</code></p>
<p><code>spanner.databases.update</code></p>
<p><code>spanner.databases.updateDdl</code></p>
<p><code>spanner.databases.useDataBoost</code></p>
<p><code>spanner. databases. useRoleBasedAccess</code></p>
<p><code>spanner.databases.write</code></p>
<p><code>spanner. instanceConfigOperations.*</code></p>
<ul>
<li><code>spanner. instanceConfigOperations. cancel</code></li>
<li><code>spanner. instanceConfigOperations. delete</code></li>
<li><code>spanner. instanceConfigOperations. get</code></li>
<li><code>spanner. instanceConfigOperations. list</code></li>
</ul>
<p><code>spanner.instanceConfigs.*</code></p>
<ul>
<li><code>spanner.instanceConfigs.create</code></li>
<li><code>spanner.instanceConfigs.delete</code></li>
<li><code>spanner.instanceConfigs.get</code></li>
<li><code>spanner.instanceConfigs.list</code></li>
<li><code>spanner.instanceConfigs.update</code></li>
</ul>
<p><code>spanner.instanceOperations.*</code></p>
<ul>
<li><code>spanner. instanceOperations. cancel</code></li>
<li><code>spanner. instanceOperations. delete</code></li>
<li><code>spanner.instanceOperations.get</code></li>
<li><code>spanner. instanceOperations. list</code></li>
</ul>
<p><code>spanner. instancePartitionOperations.*</code></p>
<ul>
<li><code>spanner. instancePartitionOperations. cancel</code></li>
<li><code>spanner. instancePartitionOperations. delete</code></li>
<li><code>spanner. instancePartitionOperations. get</code></li>
<li><code>spanner. instancePartitionOperations. list</code></li>
</ul>
<p><code>spanner.instancePartitions.*</code></p>
<ul>
<li><code>spanner. instancePartitions. create</code></li>
<li><code>spanner. instancePartitions. delete</code></li>
<li><code>spanner.instancePartitions.get</code></li>
<li><code>spanner. instancePartitions. list</code></li>
<li><code>spanner. instancePartitions. update</code></li>
</ul>
<p><code>spanner.instances.create</code></p>
<p><code>spanner.instances.delete</code></p>
<p><code>spanner.instances.get</code></p>
<p><code>spanner.instances.getIamPolicy</code></p>
<p><code>spanner.instances.list</code></p>
<p><code>spanner. instances. listEffectiveTags</code></p>
<p><code>spanner. instances. listTagBindings</code></p>
<p><code>spanner.instances.update</code></p>
<p><code>spanner.sessions.*</code></p>
<ul>
<li><code>spanner.sessions.create</code></li>
<li><code>spanner.sessions.delete</code></li>
<li><code>spanner.sessions.get</code></li>
<li><code>spanner.sessions.list</code></li>
</ul></td>
</tr>
<tr class="odd">
<td>Cloud Spanner Viewer
<p>( <code>roles/ spanner.viewer</code> )</p>
<p>A principal with this role can:</p>
<ul>
<li>View all Spanner instances (but cannot modify instances).</li>
<li>View all Spanner databases (but cannot modify or read from databases).</li>
</ul>
<p>For example, you can combine this role with the <code>roles/spanner.databaseUser</code> role to grant a user with access to a specific database, but only view access to other instances and databases.</p>
<p>This role is recommended at the Google Cloud project level for users interacting with Cloud Spanner resources in the Google Cloud console.</p>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td><p><code>monitoring.timeSeries.list</code></p>
<p><code>resourcemanager.projects.get</code></p>
<p><code>resourcemanager.projects.list</code></p>
<p><code>spanner.backupOperations.get</code></p>
<p><code>spanner.backupOperations.list</code></p>
<p><code>spanner.backupSchedules.get</code></p>
<p><code>spanner. backupSchedules. getIamPolicy</code></p>
<p><code>spanner.backupSchedules.list</code></p>
<p><code>spanner.backups.get</code></p>
<p><code>spanner.backups.getIamPolicy</code></p>
<p><code>spanner.backups.list</code></p>
<p><code>spanner.databaseOperations.get</code></p>
<p><code>spanner. databaseOperations. list</code></p>
<p><code>spanner.databaseRoles.list</code></p>
<p><code>spanner. databases. beginReadOnlyTransaction</code></p>
<p><code>spanner.databases.get</code></p>
<p><code>spanner.databases.getDdl</code></p>
<p><code>spanner.databases.getIamPolicy</code></p>
<p><code>spanner.databases.list</code></p>
<p><code>spanner.databases.useDataBoost</code></p>
<p><code>spanner. instanceConfigOperations. get</code></p>
<p><code>spanner. instanceConfigOperations. list</code></p>
<p><code>spanner.instanceConfigs.get</code></p>
<p><code>spanner.instanceConfigs.list</code></p>
<p><code>spanner.instanceOperations.get</code></p>
<p><code>spanner. instanceOperations. list</code></p>
<p><code>spanner. instancePartitionOperations. get</code></p>
<p><code>spanner. instancePartitionOperations. list</code></p>
<p><code>spanner.instancePartitions.get</code></p>
<p><code>spanner. instancePartitions. list</code></p>
<p><code>spanner.instances.get</code></p>
<p><code>spanner.instances.getIamPolicy</code></p>
<p><code>spanner.instances.list</code></p>
<p><code>spanner. instances. listEffectiveTags</code></p>
<p><code>spanner. instances. listTagBindings</code></p></td>
</tr>
<tr class="even">
<td>Cloud Spanner Backup Admin
<p>( <code>roles/ spanner.backupAdmin</code> )</p>
<p>A principal with this role can:</p>
<ul>
<li>Create, view, update, and delete backups.</li>
<li>View and manage a backup's allow policy.</li>
</ul>
<p>This role cannot restore a database from a backup.</p>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td><p><code>monitoring.timeSeries.list</code></p>
<p><code>resourcemanager.projects.get</code></p>
<p><code>resourcemanager.projects.list</code></p>
<p><code>spanner.backupOperations.*</code></p>
<ul>
<li><code>spanner. backupOperations. cancel</code></li>
<li><code>spanner.backupOperations.get</code></li>
<li><code>spanner.backupOperations.list</code></li>
</ul>
<p><code>spanner.backupSchedules.create</code></p>
<p><code>spanner.backupSchedules.delete</code></p>
<p><code>spanner.backupSchedules.get</code></p>
<p><code>spanner.backupSchedules.list</code></p>
<p><code>spanner.backupSchedules.update</code></p>
<p><code>spanner.backups.copy</code></p>
<p><code>spanner.backups.create</code></p>
<p><code>spanner.backups.delete</code></p>
<p><code>spanner.backups.get</code></p>
<p><code>spanner.backups.getIamPolicy</code></p>
<p><code>spanner.backups.list</code></p>
<p><code>spanner.backups.setIamPolicy</code></p>
<p><code>spanner.backups.update</code></p>
<p><code>spanner.databases.createBackup</code></p>
<p><code>spanner.databases.get</code></p>
<p><code>spanner.databases.list</code></p>
<p><code>spanner.instancePartitions.get</code></p>
<p><code>spanner. instancePartitions. list</code></p>
<p><code>spanner. instances. createTagBinding</code></p>
<p><code>spanner. instances. deleteTagBinding</code></p>
<p><code>spanner.instances.get</code></p>
<p><code>spanner.instances.list</code></p>
<p><code>spanner. instances. listEffectiveTags</code></p>
<p><code>spanner. instances. listTagBindings</code></p></td>
</tr>
<tr class="odd">
<td>Cloud Spanner Backup Writer
<p>( <code>roles/ spanner.backupWriter</code> )</p>
<p>This role is intended to be used by scripts that automate backup creation. A principal with this role can create backups, but cannot update or delete them.</p>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td><p><code>spanner.backupOperations.get</code></p>
<p><code>spanner.backupOperations.list</code></p>
<p><code>spanner.backupSchedules.create</code></p>
<p><code>spanner.backupSchedules.get</code></p>
<p><code>spanner.backupSchedules.list</code></p>
<p><code>spanner.backups.copy</code></p>
<p><code>spanner.backups.create</code></p>
<p><code>spanner.backups.get</code></p>
<p><code>spanner.backups.list</code></p>
<p><code>spanner.databases.createBackup</code></p>
<p><code>spanner.databases.get</code></p>
<p><code>spanner.databases.list</code></p>
<p><code>spanner.instancePartitions.get</code></p>
<p><code>spanner.instances.get</code></p></td>
</tr>
<tr class="even">
<td>Cloud Spanner Database Admin
<p>( <code>roles/ spanner.databaseAdmin</code> )</p>
<p>A principal with this role can:</p>
<ul>
<li>Get/list all Spanner instances in the project.</li>
<li>Create/list/drop databases in an instance.</li>
<li>Grant/revoke access to databases in the project.</li>
<li>Read from and write to all Cloud Spanner databases in the project.</li>
</ul>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td><p><code>cloudkms.keyHandles.*</code></p>
<ul>
<li><code>cloudkms.keyHandles.create</code></li>
<li><code>cloudkms.keyHandles.get</code></li>
<li><code>cloudkms.keyHandles.list</code></li>
</ul>
<p><code>cloudkms.operations.get</code></p>
<p><code>cloudkms. projects. showEffectiveAutokeyConfig</code></p>
<p><code>monitoring.timeSeries.*</code></p>
<ul>
<li><code>monitoring.timeSeries.create</code></li>
<li><code>monitoring.timeSeries.list</code></li>
</ul>
<p><code>resourcemanager.projects.get</code></p>
<p><code>resourcemanager.projects.list</code></p>
<p><code>spanner.databaseOperations.*</code></p>
<ul>
<li><code>spanner. databaseOperations. cancel</code></li>
<li><code>spanner.databaseOperations.get</code></li>
<li><code>spanner. databaseOperations. list</code></li>
</ul>
<p><code>spanner.databaseRoles.list</code></p>
<p><code>spanner.databases.adapt</code></p>
<p><code>spanner. databases. addSplitPoints</code></p>
<p><code>spanner. databases. beginOrRollbackReadWriteTransaction</code></p>
<p><code>spanner. databases. beginPartitionedDmlTransaction</code></p>
<p><code>spanner. databases. beginReadOnlyTransaction</code></p>
<p><code>spanner.databases.changequorum</code></p>
<p><code>spanner.databases.create</code></p>
<p><code>spanner.databases.drop</code></p>
<p><code>spanner.databases.get</code></p>
<p><code>spanner.databases.getDdl</code></p>
<p><code>spanner.databases.getIamPolicy</code></p>
<p><code>spanner.databases.list</code></p>
<p><code>spanner. databases. partitionQuery</code></p>
<p><code>spanner. databases. partitionRead</code></p>
<p><code>spanner.databases.read</code></p>
<p><code>spanner. databases. runGraphAlgorithms</code></p>
<p><code>spanner.databases.select</code></p>
<p><code>spanner.databases.setIamPolicy</code></p>
<p><code>spanner.databases.update</code></p>
<p><code>spanner.databases.updateDdl</code></p>
<p><code>spanner.databases.useDataBoost</code></p>
<p><code>spanner. databases. useRoleBasedAccess</code></p>
<p><code>spanner.databases.write</code></p>
<p><code>spanner.instancePartitions.get</code></p>
<p><code>spanner. instancePartitions. list</code></p>
<p><code>spanner. instances. createTagBinding</code></p>
<p><code>spanner. instances. deleteTagBinding</code></p>
<p><code>spanner.instances.get</code></p>
<p><code>spanner.instances.getIamPolicy</code></p>
<p><code>spanner.instances.list</code></p>
<p><code>spanner. instances. listEffectiveTags</code></p>
<p><code>spanner. instances. listTagBindings</code></p>
<p><code>spanner.sessions.*</code></p>
<ul>
<li><code>spanner.sessions.create</code></li>
<li><code>spanner.sessions.delete</code></li>
<li><code>spanner.sessions.get</code></li>
<li><code>spanner.sessions.list</code></li>
</ul></td>
</tr>
<tr class="odd">
<td>Cloud Spanner Database Reader
<p>( <code>roles/ spanner.databaseReader</code> )</p>
<p>A principal with this role can:</p>
<ul>
<li>Read from the Spanner database.</li>
<li>Execute SQL queries on the database.</li>
<li>View schema for the database.</li>
</ul>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td><p><code>monitoring.timeSeries.create</code></p>
<p><code>spanner. databases. beginReadOnlyTransaction</code></p>
<p><code>spanner.databases.get</code></p>
<p><code>spanner.databases.getDdl</code></p>
<p><code>spanner. databases. partitionQuery</code></p>
<p><code>spanner. databases. partitionRead</code></p>
<p><code>spanner.databases.read</code></p>
<p><code>spanner.databases.select</code></p>
<p><code>spanner.instancePartitions.get</code></p>
<p><code>spanner.instances.get</code></p>
<p><code>spanner.sessions.*</code></p>
<ul>
<li><code>spanner.sessions.create</code></li>
<li><code>spanner.sessions.delete</code></li>
<li><code>spanner.sessions.get</code></li>
<li><code>spanner.sessions.list</code></li>
</ul></td>
</tr>
<tr class="even">
<td>Cloud Spanner Database Reader with DataBoost
<p>( <code>roles/ spanner.databaseReaderWithDataBoost</code> )</p>
<p>Includes all permissions in the spanner.databaseReader role enabling access to read and/or query a Cloud Spanner database using instance resources, as well as the permission to access the database with Data Boost, a fully managed serverless service that provides independent compute resources.</p>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td><p><code>monitoring.timeSeries.create</code></p>
<p><code>spanner. databases. beginReadOnlyTransaction</code></p>
<p><code>spanner.databases.get</code></p>
<p><code>spanner.databases.getDdl</code></p>
<p><code>spanner. databases. partitionQuery</code></p>
<p><code>spanner. databases. partitionRead</code></p>
<p><code>spanner.databases.read</code></p>
<p><code>spanner.databases.select</code></p>
<p><code>spanner.databases.useDataBoost</code></p>
<p><code>spanner.instancePartitions.get</code></p>
<p><code>spanner.instances.get</code></p>
<p><code>spanner.sessions.*</code></p>
<ul>
<li><code>spanner.sessions.create</code></li>
<li><code>spanner.sessions.delete</code></li>
<li><code>spanner.sessions.get</code></li>
<li><code>spanner.sessions.list</code></li>
</ul></td>
</tr>
<tr class="odd">
<td>Cloud Spanner Database Role User
<p>( <code>roles/ spanner.databaseRoleUser</code> )</p>
<p>In conjunction with the IAM role Cloud Spanner Fine-grained Access User, grants permissions to individual Spanner database roles. Add a condition for each desired Spanner database role that includes the resource type of `spanner.googleapis.com/DatabaseRole` and the resource name ending with `/YOUR_SPANNER_DATABASE_ROLE`.</p>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td></td>
</tr>
<tr class="even">
<td>Cloud Spanner Database User
<p>( <code>roles/ spanner.databaseUser</code> )</p>
<p>A principal with this role can:</p>
<ul>
<li>Read from and write to the Spanner database.</li>
<li>Execute SQL queries on the database, including DML and Partitioned DML.</li>
<li>View and update schema for the database.</li>
</ul>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td><p><code>monitoring.timeSeries.create</code></p>
<p><code>spanner.databaseOperations.*</code></p>
<ul>
<li><code>spanner. databaseOperations. cancel</code></li>
<li><code>spanner.databaseOperations.get</code></li>
<li><code>spanner. databaseOperations. list</code></li>
</ul>
<p><code>spanner.databases.adapt</code></p>
<p><code>spanner. databases. beginOrRollbackReadWriteTransaction</code></p>
<p><code>spanner. databases. beginPartitionedDmlTransaction</code></p>
<p><code>spanner. databases. beginReadOnlyTransaction</code></p>
<p><code>spanner.databases.changequorum</code></p>
<p><code>spanner.databases.get</code></p>
<p><code>spanner.databases.getDdl</code></p>
<p><code>spanner. databases. partitionQuery</code></p>
<p><code>spanner. databases. partitionRead</code></p>
<p><code>spanner.databases.read</code></p>
<p><code>spanner.databases.select</code></p>
<p><code>spanner.databases.updateDdl</code></p>
<p><code>spanner.databases.write</code></p>
<p><code>spanner.instancePartitions.get</code></p>
<p><code>spanner.instances.get</code></p>
<p><code>spanner.sessions.*</code></p>
<ul>
<li><code>spanner.sessions.create</code></li>
<li><code>spanner.sessions.delete</code></li>
<li><code>spanner.sessions.get</code></li>
<li><code>spanner.sessions.list</code></li>
</ul></td>
</tr>
<tr class="odd">
<td>Cloud Spanner Fine-grained Access User
<p>( <code>roles/ spanner.fineGrainedAccessUser</code> )</p>
<p>Grants permissions to use Spanner's fine-grained access control framework. To grant access to specific database roles, also add the `roles/spanner.databaseRoleUser` IAM role and its necessary conditions.</p>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td><p><code>spanner.databaseRoles.list</code></p>
<p><code>spanner. databases. useRoleBasedAccess</code></p></td>
</tr>
<tr class="even">
<td>Cloud Spanner Database Graph Intelligence features user
<p>( <code>roles/ spanner.graphIntelligenceUser</code> )</p>
<p>Access to Graph Intelligence features.</p></td>
<td><p><code>monitoring.timeSeries.create</code></p>
<p><code>spanner. databases. beginReadOnlyTransaction</code></p>
<p><code>spanner.databases.get</code></p>
<p><code>spanner.databases.getDdl</code></p>
<p><code>spanner. databases. partitionQuery</code></p>
<p><code>spanner. databases. partitionRead</code></p>
<p><code>spanner.databases.read</code></p>
<p><code>spanner. databases. runGraphAlgorithms</code></p>
<p><code>spanner.databases.select</code></p>
<p><code>spanner.databases.useDataBoost</code></p>
<p><code>spanner.instancePartitions.get</code></p>
<p><code>spanner.instances.get</code></p>
<p><code>spanner.sessions.*</code></p>
<ul>
<li><code>spanner.sessions.create</code></li>
<li><code>spanner.sessions.delete</code></li>
<li><code>spanner.sessions.get</code></li>
<li><code>spanner.sessions.list</code></li>
</ul></td>
</tr>
<tr class="odd">
<td>Cloud Spanner Restore Admin
<p>( <code>roles/ spanner.restoreAdmin</code> )</p>
<p>A principal with this role can restore databases from backups.</p>
<p>If you need to restore a backup to a different instance, apply this role at the project level or to both instances. This role cannot create backups.</p>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td><p><code>monitoring.timeSeries.list</code></p>
<p><code>resourcemanager.projects.get</code></p>
<p><code>resourcemanager.projects.list</code></p>
<p><code>spanner.backups.get</code></p>
<p><code>spanner.backups.list</code></p>
<p><code>spanner. backups. restoreDatabase</code></p>
<p><code>spanner.databaseOperations.*</code></p>
<ul>
<li><code>spanner. databaseOperations. cancel</code></li>
<li><code>spanner.databaseOperations.get</code></li>
<li><code>spanner. databaseOperations. list</code></li>
</ul>
<p><code>spanner.databases.create</code></p>
<p><code>spanner.databases.get</code></p>
<p><code>spanner.databases.list</code></p>
<p><code>spanner.instancePartitions.get</code></p>
<p><code>spanner. instancePartitions. list</code></p>
<p><code>spanner. instances. createTagBinding</code></p>
<p><code>spanner. instances. deleteTagBinding</code></p>
<p><code>spanner.instances.get</code></p>
<p><code>spanner.instances.list</code></p>
<p><code>spanner. instances. listEffectiveTags</code></p>
<p><code>spanner. instances. listTagBindings</code></p></td>
</tr>
</tbody>
</table>

### Service agent roles

Service agent roles should only be granted to [service agents](https://docs.cloud.google.com/iam/docs/service-agents) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Role</th>
<th>Permissions</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Cloud Spanner API Service Agent
<p>( <code>roles/ spanner.serviceAgent</code> )</p>
<p>Cloud Spanner API Service Agent</p>
<blockquote>
<strong>Warning:</strong> Do not grant service agent roles to any principals except <a href="https://docs.cloud.google.com/iam/docs/service-agents">service agents</a> .
</blockquote></td>
<td><p><code>aiplatform.endpoints.get</code></p>
<p><code>aiplatform.endpoints.list</code></p>
<p><code>aiplatform.endpoints.predict</code></p>
<p><code>aiplatform.models.get</code></p>
<p><code>aiplatform.models.list</code></p>
<p><code>compute.disks.create</code></p>
<p><code>compute.disks.createTagBinding</code></p>
<p><code>compute.disks.use</code></p>
<p><code>compute.instances.create</code></p>
<p><code>compute. instances. createTagBinding</code></p>
<p><code>compute.instances.delete</code></p>
<p><code>compute.instances.get</code></p>
<p><code>compute.instances.setLabels</code></p>
<p><code>compute.instances.setMetadata</code></p>
<p><code>compute. instances. setServiceAccount</code></p>
<p><code>compute.networks.create</code></p>
<p><code>compute.networks.use</code></p>
<p><code>compute.networks.useExternalIp</code></p>
<p><code>compute.subnetworks.create</code></p>
<p><code>compute.subnetworks.use</code></p>
<p><code>compute. subnetworks. useExternalIp</code></p>
<p><code>discoveryengine. rankingConfigs. rank</code></p>
<p><code>logging.logEntries.create</code></p>
<p><code>run.jobs.run</code></p>
<p><code>run.routes.invoke</code></p>
<p><code>spanner. databases. beginOrRollbackReadWriteTransaction</code></p>
<p><code>spanner. databases. beginReadOnlyTransaction</code></p>
<p><code>spanner. databases. partitionQuery</code></p>
<p><code>spanner.databases.select</code></p>
<p><code>spanner.databases.useDataBoost</code></p>
<p><code>spanner.databases.write</code></p>
<p><code>spanner.sessions.create</code></p>
<p><code>storage.buckets.create</code></p>
<p><code>storage.buckets.get</code></p>
<p><code>storage.buckets.list</code></p>
<p><code>storage.objects.create</code></p>
<p><code>storage.objects.delete</code></p>
<p><code>storage.objects.get</code></p>
<p><code>storage.objects.list</code></p></td>
</tr>
</tbody>
</table>

> **Note:** Some client libraries reserve a portion of their session pool for read-write transactions. If you use one of these client libraries and the principal has only the `roles/spanner.databaseReader` role, requests for a read-only transaction might occasionally fail with a permissions error. To resolve this problem, see [Manage the write-sessions fraction](https://docs.cloud.google.com/spanner/docs/sessions#write-sessions-fraction) .

### Basic roles

Basic roles are project-level roles that predate IAM. See [Basic roles](https://docs.cloud.google.com/iam/docs/roles-overview#basic) for additional details.

Although Spanner supports the following basic roles, you should use one of the predefined roles shown earlier whenever possible. Basic roles include broad permissions that apply to all of your Google Cloud resources; in contrast, Spanner's predefined roles include fine-grained permissions that apply only to Spanner.

| Basic role     | Description                                                                                                                                                                                                                                         |
|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `roles/editor` | Can do all that a `roles/viewer` can do. Can also create, update, and delete Spanner resources, including custom instance configurations, instances, instance partitions, databases, backups, and backup schedules, and write data into a database. |
| `roles/owner`  | Can do all that a `roles/editor` can do. Can also modify access to instances, databases, backups, and backup schedules, and attach or detach tags on instances.                                                                                     |
| `roles/viewer` | Can list and get the metadata of schemas and instances. Can also read and query using SQL on a database.                                                                                                                                            |

## Custom roles

If the [predefined roles](https://docs.cloud.google.com/spanner/docs/iam#roles) for Spanner don't address your business requirements, you can define your own custom roles with permissions that you specify.

Before you create a custom role, you must identify the tasks that you need to perform. You can then identify the permissions that are required for each task and add these permissions to the custom role.

### Custom roles for service account tasks

For most tasks, it's obvious which permissions you need to add to your custom role. For example, if you want your service account to be able to create a database, add the permission `spanner.databases.create` to your custom role.

However, when you're reading or writing data in a Spanner table, you need to add several different permissions to your custom role. The following table shows which permissions are required for reading and writing data.

| Service account task                        | Required permissions                                                                                                                                                                                                                          |
|---------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Create a backup                             | `spanner.backups.create` `spanner.databases.createBackup`                                                                                                                                                                                     |
| Read data using SQL queries                 | `spanner.databases.select` `spanner.sessions.create` `spanner.sessions.delete`                                                                                                                                                                |
| Read data using the read API                | `spanner.databases.read` `spanner.sessions.create` `spanner.sessions.delete`                                                                                                                                                                  |
| Read data in a read-only transaction        | `spanner.databases.beginReadOnlyTransaction` , plus the permissions to read data using SQL queries or the read API                                                                                                                            |
| Read data in parallel                       | `spanner.databases.partitionQuery` or `spanner.databases.partitionRead` , plus the permissions to read data in a read-only transaction                                                                                                        |
| Restore a database                          | `spanner.backups.restoreDatabase` `spanner.databases.create`                                                                                                                                                                                  |
| Insert, update, or delete data              | `spanner.databases.beginOrRollbackReadWriteTransaction` `spanner.sessions.create` `spanner.sessions.delete` `spanner.databases.write` To read data in the same transaction, also add `spanner.databases.select` or `spanner.databases.read` . |
| Update or delete data using Partitioned DML | `spanner.databases.beginPartitionedDmlTransaction` `spanner.databases.write` `spanner.sessions.create` `spanner.sessions.delete`                                                                                                              |

### Custom roles for Google Cloud console tasks

To identify the list of permissions you need for a given task in the Google Cloud console, you determine the workflow for that task and compile the permissions for that workflow. For example, to view the data in a table, you would follow these steps in the Google Cloud console:

| Step                              | Permissions                                                                                                                        |
|-----------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| 1\. Access the project            | `resourcemanager.projects.get`                                                                                                     |
| 2\. View the list of instances    | `spanner.instances.list` , `spanner.instanceConfigs.list`                                                                          |
| 3\. Select an instance            | `spanner.instances.get` , `spanner.instanceConfigs.list` , `spanner.databases.list`                                                |
| 4\. View the list of databases    | `spanner.databases.list`                                                                                                           |
| 5\. Select a database and a table | `spanner.databases.getDdl` , `spanner.databases.select` , `spanner.databases.beginReadOnlyTransaction` , `spanner.sessions.create` |
| 6\. View data in a table          | `spanner.databases.select` , `spanner.databases.beginReadOnlyTransaction` , `spanner.sessions.create` , `spanner.sessions.delete`  |

In this example, you need these permissions:

- `resourcemanager.projects.get`
- `spanner.databases.beginReadOnlyTransaction`
- `spanner.databases.getDdl`
- `spanner.databases.list`
- `spanner.databases.select`
- `spanner.instanceConfigs.list`
- `spanner.instances.get`
- `spanner.instances.list`
- `spanner.sessions.create`
- `spanner.sessions.delete`

The following table lists the permissions required for actions in the Google Cloud console.

| Permissions                                                                                                                                                        | Action                                                                                       |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------|
| `spanner.databases.setIamPolicy`                                                                                                                                   | Add principals on the Permissions tab of the Database details page                           |
| `spanner.instances.setIamPolicy`                                                                                                                                   | Add principals on the Permissions tab of the Instance page                                   |
| `spanner.backups.setIamPolicy`                                                                                                                                     | Add principals to a backup from the permissions panel on the Backup/Restore page             |
| `spanner.backups.create` `spanner.databases.createBackup` `spanner.databases.list` <sup>1</sup> `spanner.backupOperations.list` <sup>1</sup>                       | Create a backup                                                                              |
| `spanner.backupSchedules.create` `spanner.databases.createBackup`                                                                                                  | Create a backup schedule                                                                     |
| `spanner.databases.create` `spanner.databases.get` `spanner.databaseOperations.get` `spanner.databaseOperations.list`                                              | Create a database                                                                            |
| `spanner.instancePartitions.list` `spanner.instanceConfigs.list` `spanner.instancePartitionOperations.get` `spanner.instancePartitions.create`                     | Create an instance partition                                                                 |
| `spanner.databaseOperations.get` `spanner.databaseOperations.list` `spanner.databases.updateDdl`                                                                   | Create a table Update a table schema                                                         |
| `spanner.instanceConfigs.list` `spanner.instanceOperations.get` `spanner.instances.create`                                                                         | Create an instance                                                                           |
| `spanner.backups.delete`                                                                                                                                           | Delete a backup                                                                              |
| `spanner.backupSchedules.delete`                                                                                                                                   | Delete a backup schedule                                                                     |
| `spanner.databases.drop`                                                                                                                                           | Delete a database                                                                            |
| `spanner.instancePartitions.list` `spanner.instancePartitionOperations.get` `spanner.instancePartitions.delete`                                                    | Delete an instance partition                                                                 |
| `spanner.instances.delete`                                                                                                                                         | Delete an instance                                                                           |
| `spanner.instancePartitions.list` `spanner.instanceConfigs.list` `spanner.instancePartitionOperations.get` `spanner.instancePartitions.update`                     | Modify an instance partition                                                                 |
| `spanner.instanceOperations.get` `spanner.instances.update`                                                                                                        | Modify an instance                                                                           |
| `spanner.databases.beginOrRollbackReadWriteTransaction` `spanner.databases.select` `spanner.databases.write` `spanner.sessions.create` `spanner.sessions.delete`   | Modify data in a table                                                                       |
| `spanner.instanceConfigs.list` `spanner.instances.get` `spanner.backups.get` `spanner.backups.restoreDatabase` `spanner.instances.list` `spanner.databases.create` | Restore a database from a backup                                                             |
| `spanner.databases.get` `spanner.databases.getDdl` `spanner.databases.select` `spanner.databases.beginReadOnlyTransaction` `spanner.sessions.create`               | Select a database from the database list and view the schema on the Database details page    |
| `spanner.instances.get` `spanner.instanceConfigs.list` `spanner.databases.list`                                                                                    | Select an instance from the instance list to view the Instance Details page                  |
| `spanner.backups.get` `spanner.backups.update`                                                                                                                     | Update a backup                                                                              |
| `spanner.backupSchedules.update`                                                                                                                                   | Update a backup schedule                                                                     |
| `spanner.databases.select` `spanner.databases.beginReadOnlyTransaction` `spanner.sessions.create` `spanner.sessions.delete`                                        | View data in the Data tab of the Database details page Create and run a query                |
| `spanner.backups.list` `spanner.backups.get`                                                                                                                       | View the Backup/Restore page                                                                 |
| `monitoring.metricDescriptors.get` `monitoring.metricDescriptors.list` `monitoring.timeSeries.list` `spanner.instances.get`                                        | View the graphs in the Monitor tab on the Instance details page or the Database details page |
| `spanner.backupOperations.list`                                                                                                                                    | View the list of backup operations                                                           |
| `spanner.databases.list`                                                                                                                                           | View the list of databases on the Instance details page                                      |
| `resourcemanager.projects.get` `spanner.instances.list` `spanner.instanceConfigs.list`                                                                             | View the list of instances on the Instances page                                             |
| `spanner.backups.getIamPolicy`                                                                                                                                     | View the list of principals for a backup in the permissions panel on the Backup/Restore page |
| `spanner.databaseOperations.list`                                                                                                                                  | View the list of restore operations                                                          |
| `spanner.databases.getIamPolicy`                                                                                                                                   | View the list on the Permissions tab of the Database details page                            |
| `spanner.instances.getIamPolicy`                                                                                                                                   | View the list on the Permissions tab of the Instance page                                    |

<sup>1</sup> Required if you are creating a backup from the **Backup/Restore** page at the instance level instead of the database level.

## Spanner IAM policy management

You can get, set, and test IAM policies using the REST or RPC APIs on Spanner instance, database, backup, and backup schedule resources. You can also test permissions on database roles.

### Instances

| REST API                                                                                                                                      | RPC API                                                                                                                                                                             |
|-----------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [`projects.instances.getIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/getIamPolicy)             | [`GetIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstanceAdmin.GetIamPolicy)             |
| [`projects.instances.setIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/setIamPolicy)             | [`SetIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstanceAdmin.SetIamPolicy)             |
| [`projects.instances.testIamPermissions`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/testIamPermissions) | [`TestIamPermissions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstanceAdmin.TestIamPermissions) |

### Databases

| REST API                                                                                                                                                          | RPC API                                                                                                                                                                             |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [`projects.instances.databases.getIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/getIamPolicy)             | [`GetIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.GetIamPolicy)             |
| [`projects.instances.databases.setIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/setIamPolicy)             | [`SetIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.SetIamPolicy)             |
| [`projects.instances.databases.testIamPermissions`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/testIamPermissions) | [`TestIamPermissions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.TestIamPermissions) |

### Backups

| REST API                                                                                                                                                      | RPC API                                                                                                                                                                             |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [`projects.instances.backups.getIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/getIamPolicy)             | [`GetIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.GetIamPolicy)             |
| [`projects.instances.backups.setIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/setIamPolicy)             | [`SetIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.SetIamPolicy)             |
| [`projects.instances.backups.testIamPermissions`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/testIamPermissions) | [`TestIamPermissions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.TestIamPermissions) |

### Backup schedules

| REST API                                                                                                                                                                                          | RPC API                                                                                                                                                                             |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [`projects.instances.databases.backupSchedules.getIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/getIamPolicy)             | [`GetIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.GetIamPolicy)             |
| [`projects.instances.databases.backupSchedules.setIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/setIamPolicy)             | [`SetIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.SetIamPolicy)             |
| [`projects.instances.databases.backupSchedules.testIamPermissions`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/testIamPermissions) | [`TestIamPermissions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.TestIamPermissions) |

### Database roles

Database roles support testing permissions only. You can't get or set an IAM policy on a database role.

| REST API                                                                                                                                                                                      | RPC API                                                                                                                                                                             |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [`projects.instances.databases.databaseRoles.testIamPermissions`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/testIamPermissions) | [`TestIamPermissions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.TestIamPermissions) |

## What's next

- Learn more about [IAM](https://docs.cloud.google.com/iam/docs/overview) .
- Learn how to [apply IAM roles for a Spanner database, instance, or Google Cloud project](https://docs.cloud.google.com/spanner/docs/grant-permissions) .
- Learn how to [control access to Google Cloud resources, including Spanner, from the internet](https://docs.cloud.google.com/vpc-service-controls/docs/overview#internet) .
