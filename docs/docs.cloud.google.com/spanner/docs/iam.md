[Identity and Access Management](/iam/docs/overview) (IAM) lets you control user and group access to Spanner resources at the project, Spanner instance, and Spanner database levels. For example, you can specify that a user has full control of a specific database in a specific instance in your project, but cannot create, modify, or delete any instances in your project. Using access control with IAM lets you grant a permission to a user or group without having to modify each Spanner instance or database permission individually.

This document focuses on the IAM *permissions* relevant to Spanner and the IAM *roles* that grant those permissions. For a detailed description of IAM and its features, see the [Identity and Access Management](/iam/docs/overview) developer's guide. In particular, see the [Managing IAM policies](/iam/docs/granting-changing-revoking-access) section.

## Permissions

Permissions allow users to perform specific actions on Spanner resources. For example, the `  spanner.databases.read  ` permission allows a user to read from a database using Spanner's read API, while `  spanner.databases.select  ` allows a user to execute a SQL select statement on a database. You don't directly give users permissions; instead, you grant them [predefined roles](#roles) or [custom roles](#custom-roles) , which have one or more permissions bundled within them.

The following tables list the IAM permissions that are associated with Spanner.

### Instance configurations

The following permissions apply to Spanner instance configurations. For more information, see the instance configuration references for [REST](/spanner/docs/reference/rest/v1/projects.instanceConfigs) and [RPC](/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstanceConfig) APIs.

<table>
<thead>
<tr class="header">
<th>Instance configuration permission name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instanceConfigs.create      </code></td>
<td>Create a custom instance configuration.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instanceConfigs.delete      </code></td>
<td>Delete a custom instance configuration.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instanceConfigs.get      </code></td>
<td>Get an instance configuration.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instanceConfigs.list      </code></td>
<td>List the set of instance configurations.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instanceConfigs.update      </code></td>
<td>Update a custom instance configuration.</td>
</tr>
</tbody>
</table>

### Instance configuration operations

The following permissions apply to Spanner instance configuration operations. For more information, see the instance references for [REST](/spanner/docs/reference/rest/v1/projects.instanceConfigOperations) and [RPC](/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstanceConfigOperations) APIs.

<table>
<thead>
<tr class="header">
<th>Instance configuration operation permission name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instanceConfigOperations.cancel      </code></td>
<td>Cancel an instance configuration operation.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instanceConfigOperations.delete      </code></td>
<td>Delete an instance configuration operation.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instanceConfigOperations.get      </code></td>
<td>Get an instance configuration operation.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instanceConfigOperations.list      </code></td>
<td>List instance configuration operations.</td>
</tr>
</tbody>
</table>

### Instances

The following permissions apply to Spanner instances. For more information, see the instance references for [REST](/spanner/docs/reference/rest/v1/projects.instances) and [RPC](/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.Instance) APIs.

<table>
<thead>
<tr class="header">
<th>Instance permission name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instances.create      </code></td>
<td>Create an instance.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instances.delete      </code></td>
<td>Delete an instance.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instances.get      </code></td>
<td>Get the configuration of a specific instance.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instances.getIamPolicy      </code></td>
<td>Get an instance's IAM Policy.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instances.list      </code></td>
<td>List instances.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instances.setIamPolicy      </code></td>
<td>Set an instance's IAM Policy.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instances.update      </code></td>
<td>Update an instance.</td>
</tr>
</tbody>
</table>

### Instance operations

The following permissions apply to Spanner instance operations. For more information, see the instance references for [REST](/spanner/docs/reference/rest/v1/projects.instances.operations) and [RPC](/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstanceOperations) APIs.

<table>
<thead>
<tr class="header">
<th>Instance operation permission name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instanceOperations.cancel      </code></td>
<td>Cancel an instance operation.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instanceOperations.delete      </code></td>
<td>Delete an instance operation.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instanceOperations.get      </code></td>
<td>Get a specific instance operation.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instanceOperations.list      </code></td>
<td>List instance operations.</td>
</tr>
</tbody>
</table>

### Instance partitions

The following permissions apply to Spanner instance partitions. For more information, see the instance partition references for [REST](/spanner/docs/reference/rest/v1/projects.instances.instancePartitions) and [RPC](/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstancePartition) APIs.

<table>
<thead>
<tr class="header">
<th>Instance permission name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instancePartitions.create      </code></td>
<td>Create an instance partition.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instancePartitions.delete      </code></td>
<td>Delete an instance partition.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instancePartitions.get      </code></td>
<td>Get the configuration of a specific instance partition.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instancePartitions.list      </code></td>
<td>List instance partitions.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instancePartitions.update      </code></td>
<td>Update an instance partition.</td>
</tr>
</tbody>
</table>

### Instance partition operations

The following permissions apply to Spanner instance partition operations. For more information, see the instance partition references for [REST](/spanner/docs/reference/rest/v1/projects.instances.instancePartitions.operations) and [RPC](/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#listinstancepartitionoperationsrequest) APIs.

<table>
<thead>
<tr class="header">
<th>Instance partition operation permission name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instancePartitionOperations.cancel      </code></td>
<td>Cancel an instance partition operation.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instancePartitionOperations.delete      </code></td>
<td>Delete an instance partition operation.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instancePartitionOperations.get      </code></td>
<td>Get a specific instance partition operation.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instancePartitionOperations.list      </code></td>
<td>List instance partition operations.</td>
</tr>
</tbody>
</table>

### Databases

The following permissions apply to Spanner databases. For more information, see the database references for [REST](/spanner/docs/reference/rest/v1/projects.instances.databases) and [RPC](/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.Database) APIs.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Database permission name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databases.adapt      </code></td>
<td>Lets the <a href="/spanner/docs/reference/rpc/google.spanner.adapter.v1">Spanner Adapter API</a> interact directly with Spanner.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databases.beginOrRollbackReadWriteTransaction      </code></td>
<td>Begin or roll back a <a href="/spanner/docs/transactions#read-write_transactions">read-write transaction</a> on a Spanner database.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databases.beginPartitionedDmlTransaction      </code></td>
<td>Execute an instance partitioned data manipulation language (DML) statement. For more information about instance partitioned queries, see <a href="/spanner/docs/reads#read_data_in_parallel">Read data in parallel</a> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databases.beginReadOnlyTransaction      </code></td>
<td>Begin a <a href="/spanner/docs/transactions#read-only_transactions">read-only transaction</a> on a Spanner database.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databases.create      </code></td>
<td>Create a database.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databases.createBackup      </code></td>
<td>Create a backup from the database. Also requires <code dir="ltr" translate="no">       spanner.backups.create      </code> to create the backup resource.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databases.drop      </code></td>
<td>Drop a database.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databases.get      </code></td>
<td>Get a database's metadata.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databases.getDdl      </code></td>
<td>Get a database's schema.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databases.getIamPolicy      </code></td>
<td>Get a database's IAM policy.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databases.list      </code></td>
<td>List databases.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databases.read      </code></td>
<td>Read from a database using the read API.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databases.select      </code></td>
<td>Execute a SQL select statement on a database.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databases.setIamPolicy      </code></td>
<td>Set a database's IAM policy.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databases.update      </code></td>
<td>Update a database's metadata.
Currently unavailable for IAM custom roles.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databases.updateDdl      </code></td>
<td>Update a database's schema.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databases.useDataBoost      </code></td>
<td>Use the compute resources of <a href="/spanner/docs/databoost/databoost-overview">Spanner Data Boost</a> to process instance partitioned queries.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databases.useRoleBasedAccess      </code></td>
<td>Use <a href="/spanner/docs/fgac-about">fine-grained access control</a> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databases.write      </code></td>
<td>Write into a database.</td>
</tr>
</tbody>
</table>

### Database roles

The following permissions apply to Spanner database roles. For more information, see the database references for [REST](/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles) and [RPC](/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseRole) APIs.

<table>
<thead>
<tr class="header">
<th>Database role permission name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databaseRoles.list      </code></td>
<td>List database roles.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databaseRoles.use      </code></td>
<td>Use a specified database role.</td>
</tr>
</tbody>
</table>

### Database operations

The following permissions apply to Spanner database operations. For more information, see the database references for [REST](/spanner/docs/reference/rest/v1/projects.instances.databases.operations) and [RPC](/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseOperations) APIs.

<table>
<thead>
<tr class="header">
<th>Database operation permission name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databaseOperations.cancel      </code></td>
<td>Cancel a database operation.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databaseOperations.get      </code></td>
<td>Get a specific database operation.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databaseOperations.list      </code></td>
<td>List database and restore database operations.</td>
</tr>
</tbody>
</table>

### Backups

The following permissions apply to Spanner backups. For more information, see the backups references for [REST](/spanner/docs/reference/rest/v1/projects.instances.backups) and [RPC](/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.Backup) APIs.

<table>
<thead>
<tr class="header">
<th>Backup permission name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.backups.create      </code></td>
<td>Create a backup. Also requires <code dir="ltr" translate="no">       spanner.databases.createBackup      </code> on the source database.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.backups.delete      </code></td>
<td>Delete a backup.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.backups.get      </code></td>
<td>Get a backup.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.backups.getIamPolicy      </code></td>
<td>Get a backup's IAM policy.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.backups.list      </code></td>
<td>List backups.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.backups.restoreDatabase      </code></td>
<td>Restore database from a backup. Also requires <code dir="ltr" translate="no">       spanner.databases.create      </code> to create the restored database on the target instance.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.backups.setIamPolicy      </code></td>
<td>Set a backup's IAM policy.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.backups.update      </code></td>
<td>Update a backup.</td>
</tr>
</tbody>
</table>

### Backup operations

The following permissions apply to Spanner backup operations. For more information, see the database references for [REST](/spanner/docs/reference/rest/v1/projects.instances.backups.operations) and [RPC](/spanner/docs/reference/rpc/google.spanner.admin.database.v1) APIs.

<table>
<thead>
<tr class="header">
<th>Backup operation permission name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.backupOperations.cancel      </code></td>
<td>Cancel a backup operation.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.backupOperations.get      </code></td>
<td>Get a specific backup operation.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.backupOperations.list      </code></td>
<td>List backup operations.</td>
</tr>
</tbody>
</table>

### Backup schedules

The following permissions apply to Spanner backup schedules. For more information, see the database references for the [REST](/spanner/docs/reference/rest/v1/projects.instances.backups.operations) and [RPC](/spanner/docs/reference/rpc/google.spanner.admin.database.v1) APIs.

<table>
<thead>
<tr class="header">
<th>Backup schedule permission name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.backupSchedules.create      </code></td>
<td>Create a backup schedule. Also requires <code dir="ltr" translate="no">       spanner.databases.createBackup      </code> on the source database.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.backupSchedules.delete      </code></td>
<td>Delete a backup schedule.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.backupSchedules.get      </code></td>
<td>Get a backup schedule.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.backupSchedules.list      </code></td>
<td>List backup schedules.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.backupSchedules.update      </code></td>
<td>Update a backup schedule.</td>
</tr>
</tbody>
</table>

### Sessions

The following permissions apply to Spanner sessions. For more information, see the database references for [REST](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions) and [RPC](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Session) APIs.

**Note:** Sessions are an advanced concept that only apply to users of the REST API and those who are creating their own client libraries. Learn more in [Sessions](/spanner/docs/sessions) .

<table>
<thead>
<tr class="header">
<th>Session permission name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.sessions.create      </code></td>
<td>Create a session.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.sessions.delete      </code></td>
<td>Delete a session.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.sessions.get      </code></td>
<td>Get a session.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.sessions.list      </code></td>
<td>List sessions.</td>
</tr>
</tbody>
</table>

## Predefined roles

A predefined role is a bundle of one or more [permissions](#permissions) . For example, the predefined role `  roles/spanner.databaseUser  ` contains the permissions `  spanner.databases.read  ` and `  spanner.databases.write  ` . There are two types of predefined roles for Spanner:

  - Person roles: Granted to users or groups, which allows them to perform actions on the resources in your project.
  - Machine roles: Granted to service accounts, which allows machines running as those service accounts to perform actions on the resources in your project.

**Note:** To avoid providing machines with unnecessarily broad permissions, don't grant person roles to service accounts.

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
<td><h4 id="spanner.admin" class="role-title add-link" data-text="Cloud Spanner Admin" tabindex="-1">Cloud Spanner Admin</h4>
<p>( <code dir="ltr" translate="no">         roles/                  spanner.admin        </code> )</p>
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
<td><p><code dir="ltr" translate="no">           cloudkms.keyHandles.*          </code></p>
<ul>
<li><code dir="ltr" translate="no">           cloudkms.keyHandles.create          </code></li>
<li><code dir="ltr" translate="no">           cloudkms.keyHandles.get          </code></li>
<li><code dir="ltr" translate="no">           cloudkms.keyHandles.list          </code></li>
</ul>
<p><code dir="ltr" translate="no">         cloudkms.operations.get        </code></p>
<p><code dir="ltr" translate="no">         cloudkms.                  projects.                  showEffectiveAutokeyConfig        </code></p>
<p><code dir="ltr" translate="no">           monitoring.timeSeries.*          </code></p>
<ul>
<li><code dir="ltr" translate="no">           monitoring.timeSeries.create          </code></li>
<li><code dir="ltr" translate="no">           monitoring.timeSeries.list          </code></li>
</ul>
<p><code dir="ltr" translate="no">         resourcemanager.projects.get        </code></p>
<p><code dir="ltr" translate="no">         resourcemanager.projects.list        </code></p>
<p><code dir="ltr" translate="no">           spanner.*          </code></p>
<ul>
<li><code dir="ltr" translate="no">           spanner.                      backupOperations.                      cancel          </code></li>
<li><code dir="ltr" translate="no">           spanner.backupOperations.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.backupOperations.list          </code></li>
<li><code dir="ltr" translate="no">           spanner.backupSchedules.create          </code></li>
<li><code dir="ltr" translate="no">           spanner.backupSchedules.delete          </code></li>
<li><code dir="ltr" translate="no">           spanner.backupSchedules.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      backupSchedules.                      getIamPolicy          </code></li>
<li><code dir="ltr" translate="no">           spanner.backupSchedules.list          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      backupSchedules.                      setIamPolicy          </code></li>
<li><code dir="ltr" translate="no">           spanner.backupSchedules.update          </code></li>
<li><code dir="ltr" translate="no">           spanner.backups.copy          </code></li>
<li><code dir="ltr" translate="no">           spanner.backups.create          </code></li>
<li><code dir="ltr" translate="no">           spanner.backups.delete          </code></li>
<li><code dir="ltr" translate="no">           spanner.backups.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.backups.getIamPolicy          </code></li>
<li><code dir="ltr" translate="no">           spanner.backups.list          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      backups.                      restoreDatabase          </code></li>
<li><code dir="ltr" translate="no">           spanner.backups.setIamPolicy          </code></li>
<li><code dir="ltr" translate="no">           spanner.backups.update          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      databaseOperations.                      cancel          </code></li>
<li><code dir="ltr" translate="no">           spanner.databaseOperations.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      databaseOperations.                      list          </code></li>
<li><code dir="ltr" translate="no">           spanner.databaseRoles.list          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.adapt          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      databases.                      addSplitPoints          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      databases.                      beginOrRollbackReadWriteTransaction          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      databases.                      beginPartitionedDmlTransaction          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      databases.                      beginReadOnlyTransaction          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.changequorum          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.create          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.createBackup          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.drop          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.getDdl          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.getIamPolicy          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.list          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      databases.                      partitionQuery          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      databases.                      partitionRead          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.read          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.select          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.setIamPolicy          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.update          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.updateDdl          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.useDataBoost          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      databases.                      useRoleBasedAccess          </code></li>
<li><code dir="ltr" translate="no">           spanner.databases.write          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instanceConfigOperations.                      cancel          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instanceConfigOperations.                      delete          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instanceConfigOperations.                      get          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instanceConfigOperations.                      list          </code></li>
<li><code dir="ltr" translate="no">           spanner.instanceConfigs.create          </code></li>
<li><code dir="ltr" translate="no">           spanner.instanceConfigs.delete          </code></li>
<li><code dir="ltr" translate="no">           spanner.instanceConfigs.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.instanceConfigs.list          </code></li>
<li><code dir="ltr" translate="no">           spanner.instanceConfigs.update          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instanceOperations.                      cancel          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instanceOperations.                      delete          </code></li>
<li><code dir="ltr" translate="no">           spanner.instanceOperations.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instanceOperations.                      list          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instancePartitionOperations.                      cancel          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instancePartitionOperations.                      delete          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instancePartitionOperations.                      get          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instancePartitionOperations.                      list          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instancePartitions.                      create          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instancePartitions.                      delete          </code></li>
<li><code dir="ltr" translate="no">           spanner.instancePartitions.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instancePartitions.                      list          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instancePartitions.                      update          </code></li>
<li><code dir="ltr" translate="no">           spanner.instances.create          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instances.                      createTagBinding          </code></li>
<li><code dir="ltr" translate="no">           spanner.instances.delete          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instances.                      deleteTagBinding          </code></li>
<li><code dir="ltr" translate="no">           spanner.instances.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.instances.getIamPolicy          </code></li>
<li><code dir="ltr" translate="no">           spanner.instances.list          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instances.                      listEffectiveTags          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      instances.                      listTagBindings          </code></li>
<li><code dir="ltr" translate="no">           spanner.instances.setIamPolicy          </code></li>
<li><code dir="ltr" translate="no">           spanner.instances.update          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.create          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.delete          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.list          </code></li>
</ul></td>
</tr>
<tr class="even">
<td><h4 id="spanner.backupAdmin" class="role-title add-link" data-text="Cloud Spanner Backup Admin" tabindex="-1">Cloud Spanner Backup Admin</h4>
<p>( <code dir="ltr" translate="no">         roles/                  spanner.backupAdmin        </code> )</p>
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
<td><p><code dir="ltr" translate="no">         monitoring.timeSeries.list        </code></p>
<p><code dir="ltr" translate="no">         resourcemanager.projects.get        </code></p>
<p><code dir="ltr" translate="no">         resourcemanager.projects.list        </code></p>
<p><code dir="ltr" translate="no">           spanner.backupOperations.*          </code></p>
<ul>
<li><code dir="ltr" translate="no">           spanner.                      backupOperations.                      cancel          </code></li>
<li><code dir="ltr" translate="no">           spanner.backupOperations.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.backupOperations.list          </code></li>
</ul>
<p><code dir="ltr" translate="no">         spanner.backupSchedules.create        </code></p>
<p><code dir="ltr" translate="no">         spanner.backupSchedules.delete        </code></p>
<p><code dir="ltr" translate="no">         spanner.backupSchedules.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.backupSchedules.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.backupSchedules.update        </code></p>
<p><code dir="ltr" translate="no">         spanner.backups.copy        </code></p>
<p><code dir="ltr" translate="no">         spanner.backups.create        </code></p>
<p><code dir="ltr" translate="no">         spanner.backups.delete        </code></p>
<p><code dir="ltr" translate="no">         spanner.backups.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.backups.getIamPolicy        </code></p>
<p><code dir="ltr" translate="no">         spanner.backups.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.backups.setIamPolicy        </code></p>
<p><code dir="ltr" translate="no">         spanner.backups.update        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.createBackup        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.instancePartitions.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instancePartitions.                  list        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instances.                  createTagBinding        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instances.                  deleteTagBinding        </code></p>
<p><code dir="ltr" translate="no">         spanner.instances.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.instances.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instances.                  listEffectiveTags        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instances.                  listTagBindings        </code></p></td>
</tr>
<tr class="odd">
<td><h4 id="spanner.backupWriter" class="role-title add-link" data-text="Cloud Spanner Backup Writer" tabindex="-1">Cloud Spanner Backup Writer</h4>
<p>( <code dir="ltr" translate="no">         roles/                  spanner.backupWriter        </code> )</p>
<p>This role is intended to be used by scripts that automate backup creation. A principal with this role can create backups, but cannot update or delete them.</p>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td><p><code dir="ltr" translate="no">         spanner.backupOperations.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.backupOperations.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.backupSchedules.create        </code></p>
<p><code dir="ltr" translate="no">         spanner.backupSchedules.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.backupSchedules.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.backups.copy        </code></p>
<p><code dir="ltr" translate="no">         spanner.backups.create        </code></p>
<p><code dir="ltr" translate="no">         spanner.backups.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.backups.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.createBackup        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.instancePartitions.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.instances.get        </code></p></td>
</tr>
<tr class="even">
<td><h4 id="spanner.databaseAdmin" class="role-title add-link" data-text="Cloud Spanner Database Admin" tabindex="-1">Cloud Spanner Database Admin</h4>
<p>( <code dir="ltr" translate="no">         roles/                  spanner.databaseAdmin        </code> )</p>
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
<td><p><code dir="ltr" translate="no">           cloudkms.keyHandles.*          </code></p>
<ul>
<li><code dir="ltr" translate="no">           cloudkms.keyHandles.create          </code></li>
<li><code dir="ltr" translate="no">           cloudkms.keyHandles.get          </code></li>
<li><code dir="ltr" translate="no">           cloudkms.keyHandles.list          </code></li>
</ul>
<p><code dir="ltr" translate="no">         cloudkms.operations.get        </code></p>
<p><code dir="ltr" translate="no">         cloudkms.                  projects.                  showEffectiveAutokeyConfig        </code></p>
<p><code dir="ltr" translate="no">           monitoring.timeSeries.*          </code></p>
<ul>
<li><code dir="ltr" translate="no">           monitoring.timeSeries.create          </code></li>
<li><code dir="ltr" translate="no">           monitoring.timeSeries.list          </code></li>
</ul>
<p><code dir="ltr" translate="no">         resourcemanager.projects.get        </code></p>
<p><code dir="ltr" translate="no">         resourcemanager.projects.list        </code></p>
<p><code dir="ltr" translate="no">           spanner.databaseOperations.*          </code></p>
<ul>
<li><code dir="ltr" translate="no">           spanner.                      databaseOperations.                      cancel          </code></li>
<li><code dir="ltr" translate="no">           spanner.databaseOperations.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      databaseOperations.                      list          </code></li>
</ul>
<p><code dir="ltr" translate="no">         spanner.databaseRoles.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.adapt        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  addSplitPoints        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  beginOrRollbackReadWriteTransaction        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  beginPartitionedDmlTransaction        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  beginReadOnlyTransaction        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.changequorum        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.create        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.drop        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.getDdl        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.getIamPolicy        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  partitionQuery        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  partitionRead        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.read        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.select        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.setIamPolicy        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.update        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.updateDdl        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.useDataBoost        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  useRoleBasedAccess        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.write        </code></p>
<p><code dir="ltr" translate="no">         spanner.instancePartitions.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instancePartitions.                  list        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instances.                  createTagBinding        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instances.                  deleteTagBinding        </code></p>
<p><code dir="ltr" translate="no">         spanner.instances.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.instances.getIamPolicy        </code></p>
<p><code dir="ltr" translate="no">         spanner.instances.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instances.                  listEffectiveTags        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instances.                  listTagBindings        </code></p>
<p><code dir="ltr" translate="no">           spanner.sessions.*          </code></p>
<ul>
<li><code dir="ltr" translate="no">           spanner.sessions.create          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.delete          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.list          </code></li>
</ul></td>
</tr>
<tr class="odd">
<td><h4 id="spanner.databaseReader" class="role-title add-link" data-text="Cloud Spanner Database Reader" tabindex="-1">Cloud Spanner Database Reader</h4>
<p>( <code dir="ltr" translate="no">         roles/                  spanner.databaseReader        </code> )</p>
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
<td><p><code dir="ltr" translate="no">         monitoring.timeSeries.create        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  beginReadOnlyTransaction        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.getDdl        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  partitionQuery        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  partitionRead        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.read        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.select        </code></p>
<p><code dir="ltr" translate="no">         spanner.instancePartitions.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.instances.get        </code></p>
<p><code dir="ltr" translate="no">           spanner.sessions.*          </code></p>
<ul>
<li><code dir="ltr" translate="no">           spanner.sessions.create          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.delete          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.list          </code></li>
</ul></td>
</tr>
<tr class="even">
<td><h4 id="spanner.databaseReaderWithDataBoost" class="role-title add-link" data-text="Cloud Spanner Database Reader with DataBoost" tabindex="-1">Cloud Spanner Database Reader with DataBoost</h4>
<p>( <code dir="ltr" translate="no">         roles/                  spanner.databaseReaderWithDataBoost        </code> )</p>
<p>Includes all permissions in the spanner.databaseReader role enabling access to read and/or query a Cloud Spanner database using instance resources, as well as the permission to access the database with Data Boost, a fully managed serverless service that provides independent compute resources.</p>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td><p><code dir="ltr" translate="no">         monitoring.timeSeries.create        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  beginReadOnlyTransaction        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.getDdl        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  partitionQuery        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  partitionRead        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.read        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.select        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.useDataBoost        </code></p>
<p><code dir="ltr" translate="no">         spanner.instancePartitions.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.instances.get        </code></p>
<p><code dir="ltr" translate="no">           spanner.sessions.*          </code></p>
<ul>
<li><code dir="ltr" translate="no">           spanner.sessions.create          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.delete          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.list          </code></li>
</ul></td>
</tr>
<tr class="odd">
<td><h4 id="spanner.databaseRoleUser" class="role-title add-link" data-text="Cloud Spanner Database Role User" tabindex="-1">Cloud Spanner Database Role User</h4>
<p>( <code dir="ltr" translate="no">         roles/                  spanner.databaseRoleUser        </code> )</p>
<p>In conjunction with the IAM role Cloud Spanner Fine-grained Access User, grants permissions to individual Spanner database roles. Add a condition for each desired Spanner database role that includes the resource type of `spanner.googleapis.com/DatabaseRole` and the resource name ending with `/YOUR_SPANNER_DATABASE_ROLE`.</p>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td></td>
</tr>
<tr class="even">
<td><h4 id="spanner.databaseUser" class="role-title add-link" data-text="Cloud Spanner Database User" tabindex="-1">Cloud Spanner Database User</h4>
<p>( <code dir="ltr" translate="no">         roles/                  spanner.databaseUser        </code> )</p>
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
<td><p><code dir="ltr" translate="no">         monitoring.timeSeries.create        </code></p>
<p><code dir="ltr" translate="no">           spanner.databaseOperations.*          </code></p>
<ul>
<li><code dir="ltr" translate="no">           spanner.                      databaseOperations.                      cancel          </code></li>
<li><code dir="ltr" translate="no">           spanner.databaseOperations.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      databaseOperations.                      list          </code></li>
</ul>
<p><code dir="ltr" translate="no">         spanner.databases.adapt        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  beginOrRollbackReadWriteTransaction        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  beginPartitionedDmlTransaction        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  beginReadOnlyTransaction        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.changequorum        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.getDdl        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  partitionQuery        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  partitionRead        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.read        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.select        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.updateDdl        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.write        </code></p>
<p><code dir="ltr" translate="no">         spanner.instancePartitions.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.instances.get        </code></p>
<p><code dir="ltr" translate="no">           spanner.sessions.*          </code></p>
<ul>
<li><code dir="ltr" translate="no">           spanner.sessions.create          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.delete          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.sessions.list          </code></li>
</ul></td>
</tr>
<tr class="odd">
<td><h4 id="spanner.fineGrainedAccessUser" class="role-title add-link" data-text="Cloud Spanner Fine-grained Access User" tabindex="-1">Cloud Spanner Fine-grained Access User</h4>
<p>( <code dir="ltr" translate="no">         roles/                  spanner.fineGrainedAccessUser        </code> )</p>
<p>Grants permissions to use Spanner's fine-grained access control framework. To grant access to specific database roles, also add the `roles/spanner.databaseRoleUser` IAM role and its necessary conditions.</p>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td><p><code dir="ltr" translate="no">         spanner.databaseRoles.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  useRoleBasedAccess        </code></p></td>
</tr>
<tr class="even">
<td><h4 id="spanner.restoreAdmin" class="role-title add-link" data-text="Cloud Spanner Restore Admin" tabindex="-1">Cloud Spanner Restore Admin</h4>
<p>( <code dir="ltr" translate="no">         roles/                  spanner.restoreAdmin        </code> )</p>
<p>A principal with this role can restore databases from backups.</p>
<p>If you need to restore a backup to a different instance, apply this role at the project level or to both instances. This role cannot create backups.</p>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td><p><code dir="ltr" translate="no">         monitoring.timeSeries.list        </code></p>
<p><code dir="ltr" translate="no">         resourcemanager.projects.get        </code></p>
<p><code dir="ltr" translate="no">         resourcemanager.projects.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.backups.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.backups.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  backups.                  restoreDatabase        </code></p>
<p><code dir="ltr" translate="no">           spanner.databaseOperations.*          </code></p>
<ul>
<li><code dir="ltr" translate="no">           spanner.                      databaseOperations.                      cancel          </code></li>
<li><code dir="ltr" translate="no">           spanner.databaseOperations.get          </code></li>
<li><code dir="ltr" translate="no">           spanner.                      databaseOperations.                      list          </code></li>
</ul>
<p><code dir="ltr" translate="no">         spanner.databases.create        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.instancePartitions.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instancePartitions.                  list        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instances.                  createTagBinding        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instances.                  deleteTagBinding        </code></p>
<p><code dir="ltr" translate="no">         spanner.instances.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.instances.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instances.                  listEffectiveTags        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instances.                  listTagBindings        </code></p></td>
</tr>
<tr class="odd">
<td><h4 id="spanner.serviceAgent" class="role-title add-link" data-text="Cloud Spanner API Service Agent" tabindex="-1">Cloud Spanner API Service Agent</h4>
<p>( <code dir="ltr" translate="no">         roles/                  spanner.serviceAgent        </code> )</p>
<p>Cloud Spanner API Service Agent</p>
<strong>Warning:</strong> Do not grant service agent roles to any principals except <a href="/iam/docs/service-agents">service agents</a> .</td>
<td><p><code dir="ltr" translate="no">         aiplatform.endpoints.get        </code></p>
<p><code dir="ltr" translate="no">         aiplatform.endpoints.list        </code></p>
<p><code dir="ltr" translate="no">         aiplatform.endpoints.predict        </code></p>
<p><code dir="ltr" translate="no">         aiplatform.models.get        </code></p>
<p><code dir="ltr" translate="no">         aiplatform.models.list        </code></p>
<p><code dir="ltr" translate="no">         compute.disks.create        </code></p>
<p><code dir="ltr" translate="no">         compute.disks.createTagBinding        </code></p>
<p><code dir="ltr" translate="no">         compute.disks.use        </code></p>
<p><code dir="ltr" translate="no">         compute.instances.create        </code></p>
<p><code dir="ltr" translate="no">         compute.                  instances.                  createTagBinding        </code></p>
<p><code dir="ltr" translate="no">         compute.instances.delete        </code></p>
<p><code dir="ltr" translate="no">         compute.instances.get        </code></p>
<p><code dir="ltr" translate="no">         compute.instances.setLabels        </code></p>
<p><code dir="ltr" translate="no">         compute.instances.setMetadata        </code></p>
<p><code dir="ltr" translate="no">         compute.                  instances.                  setServiceAccount        </code></p>
<p><code dir="ltr" translate="no">         compute.networks.create        </code></p>
<p><code dir="ltr" translate="no">         compute.networks.use        </code></p>
<p><code dir="ltr" translate="no">         compute.networks.useExternalIp        </code></p>
<p><code dir="ltr" translate="no">         compute.subnetworks.create        </code></p>
<p><code dir="ltr" translate="no">         compute.subnetworks.use        </code></p>
<p><code dir="ltr" translate="no">         compute.                  subnetworks.                  useExternalIp        </code></p>
<p><code dir="ltr" translate="no">         logging.logEntries.create        </code></p>
<p><code dir="ltr" translate="no">         run.jobs.run        </code></p>
<p><code dir="ltr" translate="no">         run.routes.invoke        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  beginReadOnlyTransaction        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  databases.                  partitionQuery        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.select        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.useDataBoost        </code></p>
<p><code dir="ltr" translate="no">         spanner.sessions.create        </code></p>
<p><code dir="ltr" translate="no">         storage.buckets.create        </code></p>
<p><code dir="ltr" translate="no">         storage.buckets.get        </code></p>
<p><code dir="ltr" translate="no">         storage.buckets.list        </code></p>
<p><code dir="ltr" translate="no">         storage.objects.create        </code></p>
<p><code dir="ltr" translate="no">         storage.objects.delete        </code></p>
<p><code dir="ltr" translate="no">         storage.objects.get        </code></p>
<p><code dir="ltr" translate="no">         storage.objects.list        </code></p></td>
</tr>
<tr class="even">
<td><h4 id="spanner.viewer" class="role-title add-link" data-text="Cloud Spanner Viewer" tabindex="-1">Cloud Spanner Viewer</h4>
<p>( <code dir="ltr" translate="no">         roles/                  spanner.viewer        </code> )</p>
<p>A principal with this role can:</p>
<ul>
<li>View all Spanner instances (but cannot modify instances).</li>
<li>View all Spanner databases (but cannot modify or read from databases).</li>
</ul>
<p>For example, you can combine this role with the <code dir="ltr" translate="no">          roles/spanner.databaseUser         </code> role to grant a user with access to a specific database, but only view access to other instances and databases.</p>
<p>This role is recommended at the Google Cloud project level for users interacting with Cloud Spanner resources in the Google Cloud console.</p>
<p>Lowest-level resources where you can grant this role:</p>
<ul>
<li>Instance</li>
<li>Database</li>
</ul></td>
<td><p><code dir="ltr" translate="no">         monitoring.timeSeries.list        </code></p>
<p><code dir="ltr" translate="no">         resourcemanager.projects.get        </code></p>
<p><code dir="ltr" translate="no">         resourcemanager.projects.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.databases.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.instanceConfigs.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.instanceConfigs.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.instancePartitions.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instancePartitions.                  list        </code></p>
<p><code dir="ltr" translate="no">         spanner.instances.get        </code></p>
<p><code dir="ltr" translate="no">         spanner.instances.list        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instances.                  listEffectiveTags        </code></p>
<p><code dir="ltr" translate="no">         spanner.                  instances.                  listTagBindings        </code></p></td>
</tr>
</tbody>
</table>

**Note:** When the assigned role is `  spanner.databaseReader  ` , requests for a read-only transaction might occasionally fail with a permissions error. To resolve this problem, see [Manage the write-sessions fraction](/spanner/docs/sessions#write-sessions-fraction) .

### Basic roles

Basic roles are project-level roles that predate IAM. See [Basic roles](/iam/docs/roles-overview#basic) for additional details.

Although Spanner supports the following basic roles, you should use one of the predefined roles shown earlier whenever possible. Basic roles include broad permissions that apply to all of your Google Cloud resources; in contrast, Spanner's predefined roles include fine-grained permissions that apply only to Spanner.

<table>
<thead>
<tr class="header">
<th>Basic role</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       roles/editor      </code></td>
<td>Can do all that a <code dir="ltr" translate="no">       roles/viewer      </code> can do. Can also create instances and databases and write data into a database.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       roles/owner      </code></td>
<td>Can do all that a <code dir="ltr" translate="no">       roles/editor      </code> can do. Can also modify access to databases and instances.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       roles/viewer      </code></td>
<td>Can list and get the metadata of schemas and instances. Can also read and query using SQL on a database.</td>
</tr>
</tbody>
</table>

## Custom roles

If the [predefined roles](#roles) for Spanner don't address your business requirements, you can define your own custom roles with permissions that you specify.

Before you create a custom role, you must identify the tasks that you need to perform. You can then identify the permissions that are required for each task and add these permissions to the custom role.

### Custom roles for service account tasks

For most tasks, it's obvious which permissions you need to add to your custom role. For example, if you want your service account to be able to create a database, add the permission `  spanner.databases.create  ` to your custom role.

However, when you're reading or writing data in a Spanner table, you need to add several different permissions to your custom role. The following table shows which permissions are required for reading and writing data.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Service account task</th>
<th>Required permissions</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Create a backup</td>
<td><code dir="ltr" translate="no">       spanner.backups.create      </code><br />
<code dir="ltr" translate="no">       spanner.databases.createBackup      </code></td>
</tr>
<tr class="even">
<td>Read data</td>
<td><code dir="ltr" translate="no">       spanner.databases.select      </code><br />
<code dir="ltr" translate="no">       spanner.sessions.create      </code><br />
<code dir="ltr" translate="no">       spanner.sessions.delete      </code></td>
</tr>
<tr class="odd">
<td>Restore a database</td>
<td><code dir="ltr" translate="no">       spanner.backups.restoreDatabase      </code><br />
<code dir="ltr" translate="no">       spanner.databases.create      </code></td>
</tr>
<tr class="even">
<td>Insert, update, or delete data</td>
<td><code dir="ltr" translate="no">       spanner.databases.beginOrRollbackReadWriteTransaction      </code><br />
<code dir="ltr" translate="no">       spanner.sessions.create      </code><br />
<code dir="ltr" translate="no">       spanner.sessions.delete      </code><br />
<code dir="ltr" translate="no">       spanner.databases.write      </code></td>
</tr>
</tbody>
</table>

### Custom roles for Google Cloud console tasks

To identify the list of permissions you need for a given task in the Google Cloud console, you determine the workflow for that task and compile the permissions for that workflow. For example, to view the data in a table, you would follow these steps in the Google Cloud console:

<table>
<thead>
<tr class="header">
<th>Step</th>
<th>Permissions</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1. Access the project</td>
<td><code dir="ltr" translate="no">       resourcemanager.projects.get      </code></td>
</tr>
<tr class="even">
<td>2. View the list of instances</td>
<td><code dir="ltr" translate="no">       spanner.instances.list      </code></td>
</tr>
<tr class="odd">
<td>3. Select an instance</td>
<td><code dir="ltr" translate="no">       spanner.instances.get      </code></td>
</tr>
<tr class="even">
<td>4. View the list of databases</td>
<td><code dir="ltr" translate="no">       spanner.databases.list      </code></td>
</tr>
<tr class="odd">
<td>5. Select a database and a table</td>
<td><code dir="ltr" translate="no">       spanner.databases.getDdl      </code></td>
</tr>
<tr class="even">
<td>6. View data in a table</td>
<td><code dir="ltr" translate="no">       spanner.databases.select      </code> , <code dir="ltr" translate="no">       spanner.sessions.create      </code> , <code dir="ltr" translate="no">       spanner.sessions.delete      </code></td>
</tr>
</tbody>
</table>

In this example, you need these permissions:

  - `  resourcemanager.projects.get  `
  - `  spanner.databases.getDdl  `
  - `  spanner.databases.list  `
  - `  spanner.databases.select  `
  - `  spanner.instances.get  `
  - `  spanner.instances.list  `
  - `  spanner.sessions.create  `
  - `  spanner.sessions.delete  `

The following table lists the permissions required for actions in the Google Cloud console.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Permissions</th>
<th>Action</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databases.setIamPolicy      </code></td>
<td>Add principals on the Permissions tab of the Database details page</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instances.setIamPolicy      </code></td>
<td>Add principals on the Permissions tab of the Instance page</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.backups.create      </code><br />
<code dir="ltr" translate="no">       spanner.databases.createBackup      </code><br />
<code dir="ltr" translate="no">       spanner.databases.list      </code> <sup>1</sup><br />
<code dir="ltr" translate="no">       spanner.backupOperations.list      </code> <sup>1</sup></td>
<td>Create a backup</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.backupSchedules.create      </code><br />
<code dir="ltr" translate="no">       spanner.databases.createBackup      </code></td>
<td>Create a backup schedule</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databases.create      </code></td>
<td>Create a database</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instancePartitions.list      </code><br />
<code dir="ltr" translate="no">       spanner.instancePartitionOperations.get      </code><br />
<code dir="ltr" translate="no">       spanner.instancePartitions.create      </code></td>
<td>Create an instance partition</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databaseOperations.get      </code><br />
<code dir="ltr" translate="no">       spanner.databaseOperations.list      </code><br />
<code dir="ltr" translate="no">       spanner.databases.updateDdl      </code></td>
<td>Create a table<br />
Update a table schema</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instanceConfigs.list      </code><br />
<code dir="ltr" translate="no">       spanner.instanceOperations.get      </code><br />
<code dir="ltr" translate="no">       spanner.instances.create      </code></td>
<td>Create an instance</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.backups.delete      </code></td>
<td>Delete a backup</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.backupSchedules.delete      </code></td>
<td>Delete a backup schedule</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databases.drop      </code></td>
<td>Delete a database</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instancePartitions.delete      </code></td>
<td>Delete an instance partition</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instances.delete      </code></td>
<td>Delete an instance</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instancePartitionOperations.get      </code><br />
<code dir="ltr" translate="no">       spanner.instancePartitions.update      </code></td>
<td>Modify an instance partition</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instanceOperations.get      </code><br />
<code dir="ltr" translate="no">       spanner.instances.update      </code></td>
<td>Modify an instance</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databases.beginOrRollbackReadWriteTransaction      </code><br />
<code dir="ltr" translate="no">       spanner.databases.select      </code><br />
<code dir="ltr" translate="no">       spanner.databases.write      </code><br />
<code dir="ltr" translate="no">       spanner.sessions.create      </code><br />
<code dir="ltr" translate="no">       spanner.sessions.delete      </code></td>
<td>Modify data in a table</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instanceConfigs.list      </code><br />
<code dir="ltr" translate="no">       spanner.instances.get      </code><br />
<code dir="ltr" translate="no">       spanner.backups.get      </code><br />
<code dir="ltr" translate="no">       spanner.backups.restoreDatabase      </code><br />
<code dir="ltr" translate="no">       spanner.instances.list      </code><br />
<code dir="ltr" translate="no">       spanner.databases.create      </code></td>
<td>Restore a database from a backup</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databases.get      </code><br />
<code dir="ltr" translate="no">       spanner.databases.getDdl      </code></td>
<td>Select a database from the database list and view the schema on the Database details page</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.instances.get      </code></td>
<td>Select an instance from the instance list to view the Instance Details page</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.backups.update      </code></td>
<td>Update a backup</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.backupSchedules.update      </code></td>
<td>Update a backup schedule</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databases.select      </code><br />
<code dir="ltr" translate="no">       spanner.sessions.create      </code><br />
<code dir="ltr" translate="no">       spanner.sessions.delete      </code></td>
<td>View data in the Data tab of the Database details page<br />
Create and run a query</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.backups.list      </code><br />
<code dir="ltr" translate="no">       spanner.backups.get      </code></td>
<td>View the Backup/Restore page</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       monitoring.metricDescriptors.get      </code><br />
<code dir="ltr" translate="no">       monitoring.metricDescriptors.list      </code><br />
<code dir="ltr" translate="no">       monitoring.timeSeries.list      </code><br />
<code dir="ltr" translate="no">       spanner.instances.get      </code></td>
<td>View the graphs in the Monitor tab on the Instance details page or the Database details page</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.backupOperations.list      </code></td>
<td>View the list of backup operations</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databases.list      </code></td>
<td>View the list of databases on the Instance details page</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       resourcemanager.projects.get      </code><br />
<code dir="ltr" translate="no">       spanner.instances.list      </code></td>
<td>View the list of instances on the Instances page</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.databaseOperations.list      </code></td>
<td>View the list of restore operations</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.databases.getIamPolicy      </code></td>
<td>View the list on the Permissions tab of the Database details page</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.instances.getIamPolicy      </code></td>
<td>View the list on the Permissions tab of the Instance page</td>
</tr>
</tbody>
</table>

<sup>1</sup> Required if you are creating a backup from the **Backup/Restore** page at the instance level instead of the database level.

## Spanner IAM policy management

You can get, set, and test IAM policies using the REST or RPC APIs on Spanner instance, database, and backup resources.

### Instances

<table>
<thead>
<tr class="header">
<th>REST API</th>
<th>RPC API</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/reference/rest/v1/projects.instances/getIamPolicy"><code dir="ltr" translate="no">        projects.instances.getIamPolicy       </code></a></td>
<td><a href="/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstanceAdmin.GetIamPolicy"><code dir="ltr" translate="no">        GetIamPolicy       </code></a></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/rest/v1/projects.instances/setIamPolicy"><code dir="ltr" translate="no">        projects.instances.setIamPolicy       </code></a></td>
<td><a href="/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstanceAdmin.SetIamPolicy"><code dir="ltr" translate="no">        SetIamPolicy       </code></a></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/rest/v1/projects.instances/testIamPermissions"><code dir="ltr" translate="no">        projects.instances.testIamPermissions       </code></a></td>
<td><a href="/spanner/docs/reference/rpc/google.spanner.admin.instance.v1#google.spanner.admin.instance.v1.InstanceAdmin.GetIamPolicy"><code dir="ltr" translate="no">        TestIamPermissions       </code></a></td>
</tr>
</tbody>
</table>

### Databases

<table>
<thead>
<tr class="header">
<th>REST API</th>
<th>RPC API</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/reference/rest/v1/projects.instances.databases/getIamPolicy"><code dir="ltr" translate="no">        projects.instances.databases.getIamPolicy       </code></a></td>
<td><a href="/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.GetIamPolicy"><code dir="ltr" translate="no">        GetIamPolicy       </code></a></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/rest/v1/projects.instances.databases/setIamPolicy"><code dir="ltr" translate="no">        projects.instances.databases.setIamPolicy       </code></a></td>
<td><a href="/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.SetIamPolicy"><code dir="ltr" translate="no">        SetIamPolicy       </code></a></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/rest/v1/projects.instances.databases/testIamPermissions"><code dir="ltr" translate="no">        projects.instances.databases.testIamPermissions       </code></a></td>
<td><a href="/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.TestIamPermissions"><code dir="ltr" translate="no">        TestIamPermissions       </code></a></td>
</tr>
</tbody>
</table>

### Backups

<table>
<thead>
<tr class="header">
<th>REST API</th>
<th>RPC API</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/reference/rest/v1/projects.instances.backups/getIamPolicy"><code dir="ltr" translate="no">        projects.instances.backups.getIamPolicy       </code></a></td>
<td><a href="/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.GetIamPolicy"><code dir="ltr" translate="no">        GetIamPolicy       </code></a></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/rest/v1/projects.instances.backups/setIamPolicy"><code dir="ltr" translate="no">        projects.instances.backups.setIamPolicy       </code></a></td>
<td><a href="/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.SetIamPolicy"><code dir="ltr" translate="no">        SetIamPolicy       </code></a></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/rest/v1/projects.instances.backups/testIamPermissions"><code dir="ltr" translate="no">        projects.instances.backups.testIamPermissions       </code></a></td>
<td><a href="/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.TestIamPermissions"><code dir="ltr" translate="no">        TestIamPermissions       </code></a></td>
</tr>
</tbody>
</table>

## What's next

  - Learn more about [Identity and Access Management](/iam/docs/overview) .
  - Learn how to [apply IAM roles for a Spanner database, instance, or Google Cloud project](/spanner/docs/grant-permissions) .
  - Learn how to [control access to Google Cloud resources, including Spanner, from the internet](/vpc-service-controls/docs/overview#internet) .
