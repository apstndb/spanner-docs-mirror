Cloud Spanner is a managed, mission-critical, globally consistent and scalable relational database service.

## Service: spanner.googleapis.com

The Service name `  spanner.googleapis.com  ` is needed to create RPC client stubs.

## `         google.longrunning.Operations       `

Methods

`  CancelOperation  `

Starts asynchronous cancellation on a long-running operation.

`  DeleteOperation  `

Deletes a long-running operation.

`  GetOperation  `

Gets the latest state of a long-running operation.

`  ListOperations  `

Lists operations that match the specified filter in the request.

`  WaitOperation  `

Waits until the specified long-running operation is done or reaches at most a specified timeout, returning the latest state.

## `         google.spanner.adapter.v1.Adapter       `

Methods

`  AdaptMessage  `

Handles a single message from the client and returns the result as a stream.

`  CreateSession  `

Creates a new session to be used for requests made by the adapter.

## `         google.spanner.admin.database.v1.DatabaseAdmin       `

Methods

`  AddSplitPoints  `

Adds split points to specified tables and indexes of a database.

`  ChangeQuorum  `

`  ChangeQuorum  ` is strictly restricted to databases that use dual-region instance configurations.

`  CopyBackup  `

Starts copying a Cloud Spanner Backup.

`  CreateBackup  `

Starts creating a new Cloud Spanner Backup.

`  CreateBackupSchedule  `

Creates a new backup schedule.

`  CreateDatabase  `

Creates a new Spanner database and starts to prepare it for serving.

`  DeleteBackup  `

Deletes a pending or completed `  Backup  ` .

`  DeleteBackupSchedule  `

Deletes a backup schedule.

`  DropDatabase  `

Drops (aka deletes) a Cloud Spanner database.

`  GetBackup  `

Gets metadata on a pending or completed `  Backup  ` .

`  GetBackupSchedule  `

Gets backup schedule for the input schedule name.

`  GetDatabase  `

Gets the state of a Cloud Spanner database.

`  GetDatabaseDdl  `

Returns the schema of a Cloud Spanner database as a list of formatted DDL statements.

`  GetIamPolicy  `

Gets the access control policy for a database or backup resource.

`  ListBackupOperations  `

Lists the backup long-running operations in the given instance.

`  ListBackupSchedules  `

Lists all the backup schedules for the database.

`  ListBackups  `

Lists completed and pending backups.

`  ListDatabaseOperations  `

Lists database longrunning-operations.

`  ListDatabaseRoles  `

Lists Cloud Spanner database roles.

`  ListDatabases  `

Lists Cloud Spanner databases.

`  RestoreDatabase  `

Create a new database by restoring from a completed backup.

`  SetIamPolicy  `

Sets the access control policy on a database or backup resource.

`  TestIamPermissions  `

Returns permissions that the caller has on the specified database or backup resource.

`  UpdateBackup  `

Updates a pending or completed `  Backup  ` .

`  UpdateBackupSchedule  `

Updates a backup schedule.

`  UpdateDatabase  `

Updates a Cloud Spanner database.

`  UpdateDatabaseDdl  `

Updates the schema of a Cloud Spanner database by creating/altering/dropping tables, columns, indexes, etc.

## `         google.spanner.admin.instance.v1.InstanceAdmin       `

Methods

`  CreateInstance  `

Creates an instance and begins preparing it to begin serving.

`  CreateInstanceConfig  `

Creates an instance configuration and begins preparing it to be used.

`  CreateInstancePartition  `

Creates an instance partition and begins preparing it to be used.

`  DeleteInstance  `

Deletes an instance.

`  DeleteInstanceConfig  `

Deletes the instance configuration.

`  DeleteInstancePartition  `

Deletes an existing instance partition.

`  GetIamPolicy  `

Gets the access control policy for an instance resource.

`  GetInstance  `

Gets information about a particular instance.

`  GetInstanceConfig  `

Gets information about a particular instance configuration.

`  GetInstancePartition  `

Gets information about a particular instance partition.

`  ListInstanceConfigOperations  `

Lists the user-managed instance configuration long-running operations in the given project.

`  ListInstanceConfigs  `

Lists the supported instance configurations for a given project.

`  ListInstancePartitionOperations  `

Lists instance partition long-running operations in the given instance.

`  ListInstancePartitions  `

Lists all instance partitions for the given instance.

`  ListInstances  `

Lists all instances in the given project.

`  MoveInstance  `

Moves an instance to the target instance configuration.

`  SetIamPolicy  `

Sets the access control policy on an instance resource.

`  TestIamPermissions  `

Returns permissions that the caller has on the specified instance resource.

`  UpdateInstance  `

Updates an instance, and begins allocating or releasing resources as requested.

`  UpdateInstanceConfig  `

Updates an instance configuration.

`  UpdateInstancePartition  `

Updates an instance partition, and begins allocating or releasing resources as requested.

## `         google.spanner.v1.Spanner       `

Methods

`  BatchCreateSessions  `

Creates multiple new sessions.

`  BatchWrite  `

Batches the supplied mutation groups in a collection of efficient transactions.

`  BeginTransaction  `

Begins a new transaction.

`  Commit  `

Commits a transaction.

`  CreateSession  `

Creates a new session.

`  DeleteSession  `

Ends a session, releasing server resources associated with it.

`  ExecuteBatchDml  `

Executes a batch of SQL DML statements.

`  ExecuteSql  `

Executes an SQL statement, returning all results in a single reply.

`  ExecuteStreamingSql  `

Like `  ExecuteSql  ` , except returns the result set as a stream.

`  GetSession  `

Gets a session.

`  ListSessions  `

Lists all sessions in a given database.

`  PartitionQuery  `

Creates a set of partition tokens that can be used to execute a query operation in parallel.

`  PartitionRead  `

Creates a set of partition tokens that can be used to execute a read operation in parallel.

`  Read  `

Reads rows from the database using key lookups and scans, as a simple key/value style alternative to `  ExecuteSql  ` .

`  Rollback  `

Rolls back a transaction, releasing any locks it holds.

`  StreamingRead  `

Like `  Read  ` , except returns the result set as a stream.
